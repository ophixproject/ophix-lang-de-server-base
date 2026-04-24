---
title: Zugriffsprotokoll
slug: access-auditing
order: 10
section: Erste Schritte
---

Jeder Ophix-Server führt ein Protokoll der Client-Artefakt-Zugriffsereignisse. Das Zugriffsprotokoll ist in `ophix-server-base` integriert und erfordert keine zusätzliche Konfiguration — es ist auf allen Domänenservern aktiv (Anmeldeinformationen, Konfigurationen, Zertifikate und alle zukünftigen Domänen).

---

## Was protokolliert wird

Jedes Mal, wenn ein Client einen Artefakt erfolgreich liest oder schreibt, wird ein `AccessLog`-Datensatz erstellt mit:

| Feld | Beschreibung |
| --- | --- |
| `client` | Der Client, der die Anfrage gestellt hat |
| `host` | Der Host, auf dem der Client ausgeführt wird |
| `operation` | `GET` (Lesen) oder `POST` (Schreiben/Erstellen) |
| `artifact_type` | Name der Modellklasse — `Credential`, `CertBundle` usw. |
| `artifact_name` | Artefaktname zum Zeitpunkt des Zugriffs (Snapshot — überlebt Umbenennungen und Löschungen) |
| `artifact_id` | Primärschlüssel des Artefakts zum Zeitpunkt des Zugriffs (überlebt die Artefaktlöschung) |
| `timestamp` | Zeitpunkt der Anfrage (bei Ereigniserstellung gesetzt, nicht beim Datenbankschreiben) |

Nur erfolgreiche Operationen werden protokolliert. Durch Authentifizierungs- oder Berechtigungsprüfungen abgelehnte Anfragen erscheinen nicht im Zugriffsprotokoll (sie werden über die Standard-Django-Protokollierung im Anwendungsprotokoll erfasst).

---

## Zugriffsprotokoll anzeigen

Das Zugriffsprotokoll ist in der Django-Administration unter **Ophix Core → Zugriffsprotokolle** verfügbar.

Die Listenansicht unterstützt das Filtern nach:

- Client
- Host
- Operation (GET / POST)
- Artefakttyp
- Datum (Datumsnavigation)

Das Protokoll ist schreibgeschützt — Datensätze können nicht über die Administration hinzugefügt, bearbeitet oder gelöscht werden.

---

## Leistung

Der Audit-Writer ist konzeptionell nicht blockierend. Domänenansichten legen Ereignisse in eine In-Memory-Warteschlange und kehren sofort zurück — der Anfrage-Thread wartet nie auf einen Datenbankschreibvorgang. Ein Daemon-Thread im Hintergrund leert die Warteschlange und schreibt Chargen über einen einzigen `bulk_create()`-Aufruf.

Wenn die Warteschlange unter extremer Last voll ist, werden überschüssige Ereignisse verworfen und eine Warnung in das Anwendungsprotokoll geschrieben. Die Protokollierung fügt dem primären Anfragedienst nie Gegendruck hinzu.

---

## Einstellungen

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `AUDIT_BATCH_SIZE` | `50` | Anzahl der Ereignisse, die vor dem Schreiben eines Batches in die Datenbank akkumuliert werden. |
| `AUDIT_FLUSH_INTERVAL` | `5` | Maximale Wartezeit in Sekunden vor dem Leeren eines unvollständigen Batches. Verhindert, dass Ereignisse bei geringem Datenverkehr unbegrenzt zurückgehalten werden. |

Diese können in `.env` gesetzt werden. Für die meisten Deployments sind die Standardwerte angemessen.

---

## Alte Datensätze bereinigen

Zugriffsprotokoll-Datensätze akkumulieren sich unbegrenzt, sofern sie nicht bereinigt werden. Verwenden Sie den Verwaltungsbefehl `prune_access_log`:

```bash
# Datensätze älter als 90 Tage löschen (Standard)
ophix-manage prune_access_log

# Datensätze älter als 30 Tage löschen
ophix-manage prune_access_log --days 30

# Vorschau, wie viele Datensätze entfernt würden
ophix-manage prune_access_log --dry-run
ophix-manage prune_access_log --days 30 --dry-run
```

Führen Sie dies nach einem Zeitplan aus — ein täglicher oder wöchentlicher Cron-Job ist typisch:

```cron
0 3 * * 0   ophixuser  /path/to/venv/bin/ophix-manage prune_access_log --days 90
```

Führen Sie auf einem Produktionsserver immer zuerst `--dry-run` aus, bevor Sie den Aufbewahrungszeitraum anpassen.
