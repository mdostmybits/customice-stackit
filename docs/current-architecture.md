# Architektur der aktuellen Customice-Umgebung

## Systemkontext und Laufzeitarchitektur

Das Diagramm zeigt den aus den GitOps-Dateien und den Bestandsunterlagen
ableitbaren Ist-Zustand. Die drei Namespaces `customice-test`, `customice-demo`
und `customice-prod` folgen grundsätzlich demselben Aufbau. Replikazahlen und
Größen im Diagramm beziehen sich auf Produktion.

```mermaid
flowchart LR
    users["Hotelkunden<br/>Hotelmitarbeiter<br/>Displays"]
    pms["PMS / Drittsysteme<br/>apaleo<br/>MEWS<br/>Siemens SIHOT/SCD"]
    git["GitOps-Repository<br/>Base<br/>Overlays"]

    subgraph edge["Internet / Edge"]
        cf["Cloudflare<br/>DNS / Reverse Proxy"]
    end

    subgraph hetzner["Hetzner"]
        subgraph k8s["Self-hosted Kubernetes-Cluster"]
            argo["Argo CD"]
            istio["Istio Gateways<br/>VirtualServices<br/>TLS-Zertifikate via cert-manager"]

            subgraph namespaces["Getrennte Namespaces: TEST | DEMO | PROD"]
                web["Web-UIs<br/>Frontend/Admin (2)<br/>Configurator (2)<br/>Reports (1)<br/>Screens (2)<br/>Pickup (1)<br/>Portal (1)<br/>Frontend v2 (1)<br/>Configurator v2 (1)"]
                api1["Legacy Backend<br/>1 Job-Instanz<br/>2 API-Replikate"]
                api2["Spring Backend v2<br/>2 Replikate"]
                reports["Reports Backend<br/>1 Replikat"]
                webhook["Webhook Receiver<br/>3 Replikate"]
                rabbit["RabbitMQ-Cluster<br/>3 Replikate<br/>AMQP 5672"]
                pvc[("Longhorn RWX PVC<br/>customice-pvc<br/>PROD 200 GiB")]
                backup["Kubernetes<br/>Backup-CronJobs<br/>Dateien und S3"]
                ops["Manuell verwaltete<br/>Plattformdienste<br/>Monitoring<br/>Logging<br/>Sealed Secrets<br/>(Secret-Nutzung nicht durchgängig)"]
            end
        end

        db[("MySQL auf separater DB-VM<br/>Datenbanken für TEST<br/>DEMO und PROD")]
        pdf["Externer<br/>Windows-PDF-Server<br/>DOCX → PDF"]
        mail["Externer Mailserver"]
        dbbackup["Backup-CronJob<br/>auf DB-VM"]
    end

    subgraph cloudflare["Cloudflare"]
        r2[("R2 Buckets<br/>TEST<br/>DEMO<br/>PROD<br/>SSE-C verschlüsselt")]
    end

    subgraph strato["Strato HiDrive"]
        sftp[("SFTP<br/>verschlüsselte Datei-Backups<br/>S3-Backups<br/>DB-Backups")]
        s3backup[("S3 Bucket<br/>Longhorn-Backups")]
    end

    git -->|automatische Synchronisation| argo
    argo --> namespaces

    users -->|HTTPS| cf
    pms -->|REST / Webhooks| cf
    cf --> istio
    istio --> web
    istio --> api1
    istio --> api2
    istio --> reports
    istio --> webhook

    web -->|REST| api1
    web -->|REST| api2
    web -->|REST| reports
    api1 <-->|AMQP| rabbit
    api2 <-->|AMQP| rabbit
    webhook -->|AMQP| rabbit

    api1 -->|MySQL| db
    api2 -->|MySQL| db
    reports -->|MySQL| db
    api1 --> pdf
    api1 --> mail
    api1 <-->|REST| pms
    api2 <-->|REST| pms

    api1 <-->|Dateien| pvc
    api2 <-->|S3 API| r2
    pvc --> backup
    r2 -->|Entschlüsseln und<br/>neu verschlüsseln| backup
    backup --> sftp
    pvc -.->|Longhorn-Backup| s3backup
    db --> dbbackup
    dbbackup --> sftp

    classDef inferred stroke-dasharray: 5 5;
    class ops inferred;
```

Die produktiven öffentlichen Routen sind:

| Route                                                | Kubernetes-Service       |
| ---------------------------------------------------- | ------------------------ |
| `app.customice.de`                                   | Frontend/Admin           |
| `api.customice.de`                                   | Legacy Backend           |
| `app-v2.customice.de`                                | Frontend v2              |
| `api-v2.customice.de`                                | Spring Backend v2        |
| `cfg.customice.de`                                   | Configurator             |
| `cfg-v2.customice.de`, `glc-reservation.siemens.com` | Configurator v2          |
| `reports.customice.de`, `reports-api.customice.de`   | Reports Frontend/Backend |
| `screens.customice.de`                               | Screens                  |
| `pickup.customice.de`                                | Pickup                   |
| `portal.customice.de`                                | Portal                   |
| `webhook.customice.de`                               | Webhook Receiver         |

## Einordnung der Aussagen

### Durch Dateien direkt belegt

- Argo CD wendet `base` und `overlays` automatisch an; Inhalte unter `misc`
  werden laut Repository-README manuell mit `kubectl` verwaltet.
- Die Produktionskonfiguration rendert 13 Deployments, 12 Services und zwei
  Longhorn-PVCs. `customice-pvc-files-v2` (20 GiB) ist definiert, wird in den
  vorhandenen Workload-Manifests aber nicht gemountet.
- Legacy Backend, Spring Backend und Webhook Receiver verwenden den
  RabbitMQ-Cluster `rabbitmq-customice-prod-new`.
- Die DB-Zugangsdaten von Legacy- und Spring-Backend werden aus Kubernetes
  Secrets gelesen; die zugehörigen verschlüsselten `SealedSecret`-Manifeste
  liegen unter `misc/sealed-secrets`.
- Abweichend davon enthält die vom Reports-Backend verwendete ConfigMap die
  vollständige DB-Verbindungs-URL einschließlich Passwort als GitOps-Literal.
- Das Legacy Backend bindet `customice-pvc` ein. Das Spring Backend verwendet
  einen S3-Endpunkt; die Bestandsunterlage identifiziert diesen als Cloudflare
  R2 mit SSE-C.
- Die Pod-Aufnahme aus `customice-prod` zeigt die Anwendungs-Workloads sowie
  drei laufende RabbitMQ-Pods. Der Status `2/2` der Anwendungs-Pods ist mit
  einem zusätzlichen Sidecar vereinbar.

### Aus dem Bestandsdiagramm übernommen

- Cloudflare liegt vor den öffentlich erreichbaren Anwendungen.
- MySQL, PDF- und Mailserver laufen außerhalb von Kubernetes bei Hetzner.
- apaleo und MEWS kommunizieren per REST beziehungsweise Webhooks mit der
  Anwendung.
- Datei- und Datenbank-Backups gehen verschlüsselt zu Strato HiDrive; S3-Daten
  werden aus Cloudflare R2 gelesen und für das Ziel neu verschlüsselt.

### Nicht abschließend verifizierbar

- Die GitOps-Dateien beschreiben den Sollzustand. Ohne Clusterzugriff ist nicht
  feststellbar, ob alle manuell verwalteten Ressourcen unter `misc` aktuell
  installiert sind. Deshalb ist der Plattformdienste-Knoten gestrichelt.
- Das Bestandsdiagramm stammt laut PDF-Metadaten vom 13. August 2024 und kennt
  einige inzwischen in GitOps enthaltene v2-Komponenten noch nicht.
- Die konkreten DB-, S3- und externen Service-Endpunkte liegen teilweise in
  Secrets. Das Diagramm zeigt daher die belegte logische Verbindung, nicht jeden
  Hostnamen oder Netzwerkpfad.

## Quellen

- `src/customice_k8s/base/`
- `src/customice_k8s/overlays/{test,demo,prod}/`
- `src/customice_k8s/misc/`
- `resources/current-system/customice-overview.drawio.pdf`
- `resources/current-system/customice-files-and-backups.drawio.png`
- `resources/current-system/customize-running-pods.png`
