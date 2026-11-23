# customice stackit migration

## Beschreibung

Es soll eine App Customice von Hetzner mit selfhosted Kubernetes zu dem SaaS
Stackit umgezogen werden.

Es fanden schon Vorarbeiten wie eine Migrationsanalyse statt, die hier in dem
Verzeichnis dokumentiert wurden.

MD ist IT-Architekt und Devops Engineer und hat die Aufgabe bekommen den Umzug
zu planen und über einen PoC bis zum Go-Live umzusetzen. Seine Rolle im Gesamten
ist weiter unten erklärt.

Das Ziel des Projektes ist zunächst in einem PoC zu beurteilen ob diese Umgebung
für die Anwendung verwendet werden kann. Wenn sich die Umgebung als passend
erweist, ist eine Migration der produktiven Anwendung geplant.

Für Stackit soll eine Terraformkonfiguration erstellt werden.

### Dokumente

| Dateiname                                                         | Beschreibung                          |
| ----------------------------------------------------------------- | ------------------------------------- |
| `docs/current-architecture.md`                                    | Abgeleitete Ist-Architektur           |
| `docs/customize-stackit-migration.md`                             | Projektbeschreibung                   |
| `resources/Customice-Migrationsanalyse.pdf`                       | Migrationsanalyse                     |
| `resources/SOW_STACKIT-Funding_V2.pdf`                            | Funding Dokument                      |
| `resources/current-system/customice-files-and-backups.drawio.png` | Backuplocations                       |
| `resources/current-system/customice-overview.drawio.pdf`          | Architekturdiagramm                   |
| `resources/current-system/customize-running-pods.png`             | Consolenscreenshot der laufenden Pods |
| `src/customice_k8s`                                               | GitOps config der aktuellen Umgebung  |

### Rollen

Anders als im Dokument `resources/SOW_STACKIT-Funding_V2.pdf` sind die Rollen
folgende.

- Alexander Zubeil, AZ, (BITS) — Erstellte die technische Analyse in der
  Vergangenheit. Ist jetzt nicht mehr im Projekt.
- Jan Ottensmeyer, JO, (Customice) — Application Owner/Betrieb;
  Bestandsinformationen, Tests und technische Freigaben.
- Dominik Ridinger, DR, (Customice) — Projektowner; Kundenseite koordinieren,
  Zugänge und Anforderungen bereitstellen.
- Moritz Gloth, MG, (BITS) — kaufmännischer Ansprechpartner für Vertrag, Funding
  und Eskalationen.
- Martin Moser, MM, (Customice) — Entscheider und finale Freigabe
- Michael Dost, MD, (BITS) - jetztiger Umsetzer, der die Arbeit von Alexander
  Zubeil fortsetzt.

### Beteiligte Geschäftsparteien

- Customice GmbH – Endkunde; die Hotel- und Eventplattform soll von der
  bisherigen Infrastruktur nach STACKIT umziehen.
- BITS GmbH – Umsetzungspartner; soll den PoC und die vereinbarten technischen
  Leistungen erbringen. BITS und Customize arbeiten schon länger eng zusammen.
  Es gibt kurze Komminikationswege.
- STACKIT – Cloud-Anbieter und vorgesehener Funding-Geber für Partnerleistungen.
- Hetzner – bisheriger Infrastrukturprovider, der in der Migrationsanalyse als
  Altprovider erwähnt wird; im SOW nicht als Funding-Partei aufgeführt.
- BITS und Customice arbeiten eng zusammen. Meiner Einschätzung nache ist der
  treibende Faktor, dass ein Umzug in eine managed Umgebung an der Zeit ist und
  das Funding mitzunehmen würde günstig dazukommen.

### Das Funding und PoC

Der Migrationsplan beschreibt eine Zeitlinie bis zum Go-Live.

Funding und PoC sind nur zufällig parallel und sollten den Zeitplan nicht
relevant beeinflussen.
