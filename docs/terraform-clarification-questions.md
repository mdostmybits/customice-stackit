# Klärungsfragen für die Terraform-Infrastruktur bei STACKIT

Dieses Dokument sammelt die Fragen, die vor beziehungsweise während der
Erstellung der Terraform-Konfiguration für die neue STACKIT-Umgebung geklärt
werden müssen. Das Deployment der Anwendungen soll weiterhin über Argo CD
erfolgen. Terraform soll dagegen die darunterliegende, bisher nicht als Code
verwaltete Infrastruktur bereitstellen.

Grundlage ist der in [`current-architecture.md`](current-architecture.md)
beschriebene Ist-Zustand. Für den PoC müssen nicht alle Fragen abschließend für
den späteren Produktivbetrieb beantwortet sein. Die Entscheidungen dürfen dem
späteren Zielbild jedoch nicht entgegenstehen.

## 1. Umfang des PoC

- [x] Soll zunächst die Umgebung `test` oder `demo` migriert werden? test
- [ ] Soll der PoC in einem eigenen Kubernetes-Cluster laufen? egal
- [x] Ist später ein gemeinsamer Cluster für TEST, DEMO und PROD oder ein
      eigener PROD-Cluster vorgesehen? ja
- Welche bestehenden externen Dienste werden im PoC zunächst weiterverwendet?
  - [x] MySQL bei Hetzner, auf MariaDB testen
  - [x] Cloudflare R2, nein
  - [x] PDF-Server, ja ist erreichbar
  - [x] Mailserver, ja
  - [x] Strato HiDrive, ja für Testbackup
- [x] Welche Dienste sollen bereits im PoC durch STACKIT-Dienste ersetzt werden?
  - Datenbank
  - S3 Storage

Diese Entscheidung bestimmt den Terraform-Umfang stärker als die Anzahl der im
PoC betriebenen Anwendungen.

## 2. STACKIT-Zugang und organisatorische Struktur

- [ ] Welche STACKIT-Organisation und welches Projekt werden verwendet?
- [ ] Welche Region und Availability Zone sind vorgesehen?
- [ ] Sind ausreichende Quotas für SKE, Server, Load Balancer, Volumes und
      öffentliche IP-Adressen vorhanden?
- Wie authentifiziert sich Terraform?
  - [ ] persönlicher Account für den PoC
  - [ ] Service Account für CI/CD, github
- [ ] Wer darf Infrastruktur erstellen, ändern und löschen?
- [ ] Gibt es Vorgaben für Namen, Tags und Kostenstellen?

## 3. Terraform-Betriebsmodell

- [ ] In welchem Repository liegt die Terraform-Konfiguration?
- [ ] Wo wird der Terraform-State gespeichert?
- [ ] Wie werden State Locking und Verschlüsselung umgesetzt?
- Von wo werden `terraform plan` und `terraform apply` ausgeführt?
  - [x] lokale Administration, ja das
  - [x] GitLab- oder GitHub-CI, nein
  - [x] anderer Runner, nein
- [ ] Wer prüft und genehmigt ein `apply`?
- [ ] Wie werden Terraform- und Provider-Versionen festgelegt?
- Wie werden TEST, DEMO und PROD getrennt?
  - [ ] getrennte States
  - [ ] getrennte Verzeichnisse
  - [ ] getrennte STACKIT-Projekte
- Welche Ressourcen benötigen einen Löschschutz, insbesondere Datenbanken und
  persistente Speicher? auf jeden Fall
  - [x] Datenbanken, ja
  - [x] S3, ja
- [ ] Welche Terraform-Ausgaben benötigt der nachfolgende Argo-CD-Bootstrap?

## 4. SKE-Cluster

- [x] Welche Kubernetes-Version soll eingesetzt werden? neueste
- [x] Soll die Kubernetes-API öffentlich oder nur privat erreichbar sein? nur
      privat
- [ ] Welche Node-Pools werden benötigt?
- Welche VM-Größen und wie viele Nodes sind für den PoC vorgesehen?
  - [x] Migrationsanalyse
  - [ ] Report von JO
- [x] Wird Cluster Autoscaling benötigt? nein
- [x] Müssen Nodes über mehrere Availability Zones verteilt werden? nein
- Welche Anforderungen gibt es an:
  - [ ] Labels und Taints
  - [ ] Wartungsfenster
  - [ ] automatische Kubernetes-Upgrades
  - [ ] CPU-Architektur, MS fragen
- [x] Welche CPU- und RAM-Requests benötigen die Anwendungen tatsächlich? JO
      schickt Report

Insbesondere das erforderliche Sizing ist aus den vorhandenen Deployment-Dateien
noch nicht ausreichend ableitbar.

## 5. Netzwerk

- [x] Welche CIDRs werden für Nodes, Pods und Services verwendet? keine Vorgaben
      bis jetzt
- [x] Benötigen die Workloads eine feste ausgehende IP-Adresse? Cloudflare
      braucht fixe IPs
- Gibt es IP-Allowlisting bei
  - [x] PMS-Systemen (nein)
  - [x] PDF- (nein)
  - [ ] Mail- (Marc) ?
- Wie erreicht der Cluster die zunächst bei Hetzner verbleibenden Dienste?
  - [x] über öffentliche Endpunkte, Mail?, PDF
  - [ ] über ein VPN
  - [ ] über eine Site-to-Site-Verbindung
- [x] Ist privates Networking zur Datenbank erforderlich? ja Tunnel
- [x] Welche eingehenden Ports werden benötigt? keine
- [x] Soll der Ingress ausschließlich über Cloudflare erreichbar sein? ja
- [ ] Müssen interne Admin-Endpunkte oder Argo CD separat erreichbar sein?

Eine feste Egress-IP könnte insbesondere wegen externer PMS-Systeme ein
entscheidender Punkt sein.

## 6. Ingress, Cloudflare und TLS

- [x] Bleibt Cloudflare DNS- und Reverse-Proxy-Anbieter?, ja
- [ ] Bleibt Istio als Ingress-Lösung bestehen?
- [ ] Soll das Istio Ingress Gateway einen STACKIT Load Balancer erzeugen?
- [ ] Wird dafür eine feste öffentliche IP-Adresse benötigt?
- Wo endet TLS?
  - [ ] bei Cloudflare
  - [ ] am Istio Gateway
  - [ ] an beiden Stellen
- Wie stellt `cert-manager` Zertifikate aus?
  - [ ] HTTP-01
  - [ ] DNS-01 über Cloudflare
- [ ] Wo werden Cloudflare API-Tokens verwaltet?
- [ ] Welche Hostnamen werden für den PoC verwendet?
- [ ] Wer erstellt die DNS-Einträge und zu welchem Zeitpunkt?
- [ ] Werden die vorhandenen `NodePort`-Services weiterhin benötigt oder können
      sie hinter Istio durch `ClusterIP`-Services ersetzt werden?

## 7. Persistenter Speicher

Das Legacy Backend verwendet derzeit das Storage-Profil `longhorn-rwx`. Daher
ist insbesondere zu klären:

- [ ] Benötigt die Anwendung tatsächlich `ReadWriteMany`?
- [ ] Greifen mehrere Pods gleichzeitig schreibend auf dasselbe Dateisystem zu?
- [ ] Kann der Legacy-Dateispeicher durch S3-kompatiblen Object Storage ersetzt
      werden?
- [ ] Falls RWX weiterhin benötigt wird:
  - [ ] Welcher STACKIT-Dienst kann ein gemeinsames Dateisystem bereitstellen?
  - [ ] Oder soll Longhorn weiterhin im SKE-Cluster betrieben werden?
- [ ] Welche Kapazität wird für PoC und Produktion benötigt?
- [ ] Welche Anforderungen bestehen an IOPS, Latenz, Snapshots, Backup,
      Verschlüsselung und Wiederherstellungszeit?
- Wie werden die bestehenden Daten in den neuen Speicher übertragen?
  - [ ] rclone, ja
  - [ ] sftp server

Ein normales Kubernetes-Blockvolume ersetzt ein bestehendes RWX-Volume nicht
automatisch.

## 8. Datenbank

- [x] Bleibt MySQL für den PoC bei Hetzner oder wird eine STACKIT-Datenbank
      aufgebaut? mariadb DB SaaS Lösung
- [x] Welche genaue MySQL- beziehungsweise MariaDB-Version läuft aktuell? MySQL
      8.0.27
- [ ] Welche Datenbankfunktionen werden verwendet, beispielsweise Stored
      Procedures, Trigger, Events, besondere Collations oder SQL-Modi?
- [ ] Welche Datenbanken, Benutzer und Berechtigungen werden benötigt?
- [ ] Ist TLS für Datenbankverbindungen vorgeschrieben?
- [x] Welche Anforderungen bestehen an Größe, Performance, Hochverfügbarkeit und
      Backups?
- [x] Welche RPO- und RTO-Ziele gelten?
- [x] Wie erfolgt die Datenmigration?
- Darf Terraform
  - [ ] Datenbanken und Datenbankbenutzer verwalten
  - [ ] nur die DB-Instanz?
- Wo werden Datenbankpasswörter abgelegt?
  - [ ] Sealed Secrets

Das derzeit in einer ConfigMap enthaltene Passwort des Reports-Backends sollte
im Zuge der Migration in eine Secret-Lösung überführt werden.

## 9. Object Storage und Dateidaten

- [ ] Bleibt Cloudflare R2 bestehen?
- [ ] Soll stattdessen STACKIT Object Storage verwendet werden?
- [ ] Unterstützt die Anwendung die notwendige S3-Kompatibilität vollständig?
- [ ] Wird SSE-C weiterhin benötigt und vom Ziel unterstützt?
- [ ] Wer erstellt Buckets, Credentials, Lifecycle-Regeln und Zugriffspolicies?
- [ ] Welche dieser Ressourcen werden durch Terraform verwaltet?
- [ ] Müssen vorhandene R2-Daten migriert werden?
- [ ] Kann die Anwendung während der Migration gleichzeitig auf alten und neuen
      Storage zugreifen?

## 10. Abgrenzung zwischen Terraform und Argo CD

Terraform sollte typischerweise folgende Infrastruktur verwalten:

- [ ] STACKIT-Projekt- und IAM-Ressourcen
- [ ] Netzwerk, Subnetze und Security-Regeln
- [ ] SKE-Cluster und Node-Pools
- [ ] öffentliche IP-Adressen und gegebenenfalls Infrastruktur-Load-Balancer
- [ ] Managed Database
- [ ] Object Storage
- [ ] Infrastruktur-Backups
- [ ] gegebenenfalls DNS

Argo CD sollte typischerweise folgende Kubernetes-Ressourcen verwalten:

- [ ] Namespaces
- [ ] Istio und dessen Konfiguration
- [ ] cert-manager
- [ ] RabbitMQ Operator und RabbitMQ-Cluster
- [ ] Monitoring und Logging innerhalb des Clusters
- [ ] Secret-Controller
- [ ] Anwendungen und CronJobs

Für diese Trennung sind noch folgende Fragen zu beantworten:

- [ ] Wie wird Argo CD erstmalig installiert?
- [ ] Gibt es ein Bootstrap-Repository beziehungsweise eine Root Application?
- [ ] Verwaltet Argo CD nach dem Bootstrap auch sich selbst?
- [ ] Wie erhält Argo CD Zugriff auf private Git- und Image-Repositories?
- [ ] In welcher Reihenfolge werden CRDs, Operatoren und Anwendungen
      installiert?
- [ ] Sollen die bislang manuell installierten Inhalte unter `misc` vollständig
      in GitOps überführt werden?

Terraform und Argo CD dürfen nicht dieselben Kubernetes-Ressourcen verwalten.

## 11. Secrets

Vorhandene Sealed Secrets sind an den Schlüssel des aktuellen
Sealed-Secrets-Controllers gebunden. Deshalb ist zu klären:

- [ ] Soll der bestehende private Sealed-Secrets-Schlüssel sicher migriert
      werden?
- [ ] Oder werden sämtliche Secrets für den neuen Cluster neu verschlüsselt?
- [ ] Sind alle Klartextwerte für eine Neuerstellung verfügbar?
- [ ] Soll langfristig ein externer Secret Manager verwendet werden?
- [ ] Wie erhält Argo CD seine Git- und Registry-Credentials?
- [ ] Wie wird verhindert, dass Secrets im Terraform-State landen?
- [ ] Wer ist für Rotation und Widerruf der Credentials verantwortlich?

## 12. Backup und Disaster Recovery

- [ ] Was muss bereits im PoC gesichert werden?
  - [x] Datenbank
  - [x] S3 Storage
- [ ] Bleibt Strato HiDrive als Backupziel erhalten?
- Welche Daten müssen zukünftig gesichert werden?
  - [ ] Datenbank
  - [ ] RWX-Dateien
  - [ ] Object Storage
  - [ ] Kubernetes-Ressourcen
- Welche RPO- (Recovery Point Objective) und RTO- (Recovery Time Objective)
  Werte gelten?\
  **RPO**: maximal tolerierter Datenverlust, gemessen in Zeit. Er bestimmt
  Backup- bzw. Replikationsintervall\
  **RTO**: maximal tolerierte Wiederherstellungsdauer bis der Service wieder
  läuft. Er bestimmt Automatisierungsgrad und Standby-Architektur
  - [ ] RPO
  - [ ] RTO
- [ ] Wie lange werden Backups aufbewahrt?
- [ ] Wer überwacht fehlgeschlagene Backups?
- [ ] Wann und wie wird ein Restore-Test durchgeführt?
- [ ] Werden STACKIT-Snapshots zusätzlich oder anstelle der bisherigen Backups
      verwendet?

## 13. Monitoring, Logging und Betrieb

- [ ] Welche bestehende Monitoring-Lösung soll übernommen werden?
- [ ] Wo werden Metriken und Logs persistent gespeichert?
- [ ] Welche Alarmierungsziele gibt es?
- [ ] Müssen STACKIT-Infrastrukturmetriken integriert werden?
- [ ] Wer erhält Cluster- und Argo-CD-Alarme?
- [ ] Welche Audit- und Aufbewahrungsanforderungen gelten?
- [ ] Wer ist für Kubernetes-, Plattform- und Add-on-Upgrades verantwortlich?

## Vorrangig zu klärende Punkte

Vor Beginn einer belastbaren Terraform-Implementierung sollten mindestens die
folgenden Punkte geklärt sein:

1. Ziel für das bestehende RWX-Volume
2. Netzwerkzugriff und feste Egress-IP zu externen Systemen
3. Verbleib oder Migration der MySQL-Datenbank
4. Migration oder Neuerstellung der Secrets
5. Bootstrap und eindeutige Zuständigkeit von Terraform und Argo CD
6. PoC-Umgebung, STACKIT-Projekt und Region

Sind diese Punkte entschieden, kann eine erste Terraform-Struktur für den PoC
erstellt werden. Anforderungen an Hochverfügbarkeit, endgültiges Sizing und den
Produktivbetrieb können anschließend schrittweise ergänzt werden.
