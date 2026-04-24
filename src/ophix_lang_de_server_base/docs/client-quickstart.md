---
title: Client-Schnellstart
slug: client-quickstart
order: 5
section: Erste Schritte
---

Jeder Ophix Tier-1-Client folgt demselben Initialisierungsprozess. Die nachstehenden Beispiele verwenden `cred-client` — ersetzen Sie diesen je nach Domäne durch `cert-client`, `conf-client` usw.

---

## Installation

```bash
pip install ophix-cred-client
```

---

## Einrichtung in einem Schritt

```bash
cred-client quickstart https://credserver.internal mein-client
```

Dieser Befehl:

1. Speichert die Server-URL in `.cred.env`
2. Lädt das CA-Zertifikat des Servers herunter und speichert es lokal
3. Registriert diesen Client beim Server und speichert das API-Token

Das war's — der Client ist jetzt einsatzbereit.

---

## Schritt für Schritt

Wenn Sie jeden Schritt separat kontrollieren möchten:

```bash
cred-client set server https://credserver.internal
cred-client download ca-cert
cred-client register mein-client
```

---

## Überprüfung

```bash
cred-client doctor
```

Zeigt den Konfigurationsstatus an: Server-URL, CA-Zertifikat, Token und Verbindung.

---

## Konfigurationsdatei

Die Konfiguration wird in `.cred.env` (im aktuellen Verzeichnis oder in `~`) gespeichert. Beispiel:

```ini
CREDSERVER_URL=https://credserver.internal
CREDSERVER_CA_CERT=/home/user/.cred-ca.pem
CREDSERVER_API_TOKEN=<64-stelliges Hex-Token>
```

---

## Token-Rotation

Tokens sollten regelmäßig erneuert werden. Die Rotation generiert ein neues Token, validiert es beim Server und aktualisiert dann die Konfigurationsdatei:

```bash
cred-client rotate-token
```

Für die geplante Rotation (Cron):

```cron
0 2 * * 1   user  /path/to/venv/bin/cred-client rotate-token
```

Die Rotation schlägt laut fehl, wenn der Server nicht erreichbar ist — führen Sie sie niemals im stillen Modus aus.

---

## Zugriff auf Artefakte gewähren

Clients haben standardmäßig keinen Zugriff auf Artefakte. Ein Administrator muss sie explizit über die Django-Administration verknüpfen:

1. Die Detailseite des **Clients** oder des **Artefakts** in der Administration öffnen
2. Die Inline-Ansicht zum Erstellen der Verknüpfung verwenden
3. **Enabled** auf dem Link aktivieren

Sobald die Verknüpfung aktiviert ist, kann der Client das Artefakt abrufen.
