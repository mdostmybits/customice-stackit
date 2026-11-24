# Fragen an Beteiligte

## Alex – technische Planung und Umsetzung

- [ ] Sind die erwähnten Anforderungen nochmals zu überprüfen?
- [ ] Sind dir irgendwelche Hürden aufgefallen?

## Jan – Application Owner und Betrieb

- [x] Gibt es schon Termine? nein aus der Sicht von Customice keine Eile
- [x] aktuelle MySql Version: 8.0.27
- [x] Wurde DB Migration MySql nach Mariadb schon getestet? nein
  - Wie soll der Datenbankmigrationstest aussehen: Test Dump in temporär
    aufgesetzte Mariadb einspielen, bei Problemen zurück an Customice
- [x] Migration Object Storage: rclone kann von Quellumgebung auf sfpt
      exportieren und von da in Zielumgebung.
- [x] Wie werden die Deployments gespeichert? ArgoCD, das auch im Cluster läuft
- [x] Wie lange darf die Anwendung beim Cutover nicht oder nur eingeschränkt
      verfügbar sein? ca 1 h
- [x] Wie sehen Datenbank-Final-Sync, Schreibstopp, Rückfall auf Hetzner und
      Wiederaufnahme des Betriebs konkret aus? Es gibt eine Downtime
- [x] Wie wird die neue Umgebung getestet? Mit Testdaten
- [x] Wer übernimmt nach der Stabilisierung den laufenden Betrieb? Customice.
      Einweisung muss dann schon vorher passieren.
- [x] Welche Betriebsanforderungen gelten für Backups, Wiederherstellung,
      Alarmierung und Hypercare?
- [ ] Gibt es MySQL-spezifische Funktionen, Stored Procedures, Trigger, Events,
      Zeichensätze oder SQL-Modi, die bei MariaDB Ärger machen könnten?
- [ ] Welche Komponenten, Jobs und externen Abhängigkeiten der Anwendung müssen
      neben den genannten Backends berücksichtigt werden?
- [ ] Welche fachlichen und technischen Testfälle müssen vor dem Go-Live
      bestanden sein? Wer führt sie aus und wer erteilt die Freigabe?

**Anmerkungen von Jan:**

- Kann Umgebung langsam hochskaliert werden?
  - Prinzip an test Umgebung testen
  - Dann schrittweise hochskalieren
- 3 Namespaces für test demo prod
- Für loki S3 Bucket statt file system.
- Blockstorage Lösung für pods für statische Bilderresourcen wurde vergessen
- Longhorn als Persistierlösung soll nicht mehr verwendet werden

## Dominik – Projektkoordination und Kundenseite

- [x] Sind die erwähnten Anforderungen nochmals zu überprüfen? (Alex) ja
- [x] In SOW_STACKIT Funding_V2.pdf wird eine Rollenverteilung genannt. Ist das
      aktuell, oder ist das sowieso nur für das Funding? muss angepasst werden
- [ ] Welche Zugänge, Berechtigungen und Freigaben müssen bereitgestellt werden
      – etwa für STACKIT, Hetzner, DNS/Cloudflare, Quellcode und CI/CD – und bis
      wann? Muss erst noch geklärt werden

## Moritz – kaufmännische und Funding-Abhängigkeiten

- [x] Sind durch das Funding Termine vorgegeben, die eingehalten werden müssen? nichts kritisches
- [x] Wer muss welche Leistungen wann dokumentieren oder bestätigen, damit
      daraus für mich kein unerwarteter Zusatzaufwand kurz vor dem Go-Live
      entsteht? nichts kritisches
- [x] Customice will mit kleinerer Umgebung anfangen, als im Fundking
      Dokument beschrieben. 1 statt 3 workers. In Ordnung? ja
