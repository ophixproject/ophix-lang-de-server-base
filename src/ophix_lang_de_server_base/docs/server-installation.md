---
title: Server-Installation
slug: server-installation
order: 10
section: Erste Schritte
---

Dieser Leitfaden beschreibt die Installation eines Ophix-Servers von Grund auf. Jeder Server ist eine eigenständige Django-Instanz — ein Domain-Pip-Paket, das über `ophix-server-base` installiert wird.

---

## Voraussetzungen

- Python 3.10+
- MariaDB (Standard-Engine) oder eine andere unterstützte Engine
- Ein Webserver (nginx empfohlen)
- Root- oder sudo-Zugriff für die Dienst-Installation

---

## Geführte Installation

Die empfohlene Reihenfolge verwendet zwei Verwaltungsbefehle:

### Schritt 1 — Installation konfigurieren

```bash
ophix-manage configure_install credserver
```

Der interaktive Assistent erfasst:

- Installationsverzeichnis
- Hostname und TLS-Zertifikate (mit CN/SAN-Validierung)
- Datenbank-Zugangsdaten (Live-Verbindungstest)
- Superuser-Zugangsdaten
- Theme-Aktivierung und Admin-Titel

Das Ergebnis wird in `.<slug>.conf` und `.env` geschrieben (beide chmod 600).

### Schritt 2 — Installation ausführen

```bash
ophix-manage run_install credserver
```

Dieser Befehl:

- Erstellt die Verzeichnisstruktur in `INSTALL_DIR`
- Kopiert TLS-Dateien in `ssl/certs/` und `ssl/private/`
- Generiert nginx- und systemd-Konfigurationsdateien
- Führt `migrate`, `collectstatic --noinput` aus
- Erstellt oder aktualisiert den Superuser
- Aktiviert das Theme und setzt den Admin-Titel
- Importiert die Inline-Dokumentation automatisch

### Schritt 3 — Systemdienst installieren

```bash
sudo bash credserver_sudo_install.sh
```

Setzt Berechtigungen, installiert die nginx-Konfiguration, installiert und startet den systemd-Dienst.

---

## Aktualisierung

Für ein Routine-Upgrade:

```bash
pip install --upgrade ophix-server-base ophix-creds
ophix-manage migrate
ophix-manage collectstatic --noinput
sudo systemctl restart credserver
```

Wenn neue `.env`-Variablen hinzugefügt wurden, führen Sie zuerst `generate_deploy_config --append` aus.

---

## Entwicklungsumgebung

Für eine lokale Entwicklungsumgebung:

```bash
python -m venv venv
source venv/bin/activate
pip install ophix-server-base ophix-creds ophix-docs
ophix-manage configure_install credserver
ophix-manage run_install credserver
ophix-manage runserver
```

---

## Einstellungsreferenz

### Server-Identität

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `SERVER_NAME` | _(erforderlich)_ | Anzeigename dieser Serverinstanz |
| `SERVER_VERSION` | _(erforderlich)_ | Version der eingesetzten Server-Software |
| `INSTALL_DIR` | _(erforderlich)_ | Stammverzeichnis der Installation |
| `ALLOWED_HOSTS` | _(erforderlich)_ | Zulässige Hostnamen (kommagetrennt) |
| `DEBUG` | `False` | Django-Debug-Modus. In der Produktion niemals aktivieren |

### Sicherheit

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `SECRET_KEY` | _(erforderlich)_ | Django-Secret-Key. Bei der Installation automatisch generiert |
| `AUTH_LEAK_INFO` | `False` | Fehlerdetails in API-Antworten aufnehmen. Nur in der Entwicklung auf `True` setzen |
| `MINIMUM_TOKEN_ROTATE_TIME` | `3600` | Mindestintervall in Sekunden zwischen Token-Rotationen |

### Datenbank

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `DB_ENGINE` | `mariadb` | Datenbank-Engine. Optionen: `mariadb`, `postgres`, `mssql`, `oracle`, `cockroachdb` |
| `DB_NAME` | _(erforderlich)_ | Datenbankname |
| `DB_USER` | _(erforderlich)_ | Datenbankbenutzer |
| `DB_PASSWORD` | _(erforderlich)_ | Datenbankpasswort |
| `DB_HOST` | `localhost` | Datenbankhost |
| `DB_PORT` | _(Engine-Standard)_ | Datenbankport |

### Lokalisierung

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `LANGUAGE_CODE` | `en-us` | Django-Sprachcode (z. B. `fr`, `de`, `ru`) |
| `TIME_ZONE` | `UTC` | Server-Zeitzone |

### Administrationsoberfläche

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `ADMIN_SITE_HEADER` | `Ophix` | Titelzeile der Administration |
| `SHOW_CLIENT_ARTIFACT_MODEL` | `False` | Client-Artefakt-Verknüpfungsmodell in der Administration anzeigen |
| `SHOW_ACCESS_LOGS` | `False` | Zugriffsprotokoll-Abschnitt in der Administration anzeigen |
| `SHOW_AUTH_MODELS` | `False` | Django-Auth-Modelle in der Administration anzeigen |
| `SHOW_THEME_MODEL` | `False` | Theme-Modell in der Administration anzeigen |

### Zugriffsprotokoll

| Variable | Standard | Beschreibung |
| --- | --- | --- |
| `AUDIT_BATCH_SIZE` | `50` | Anzahl Ereignisse pro Schreib-Batch |
| `AUDIT_FLUSH_INTERVAL` | `5` | Maximale Flush-Verzögerung in Sekunden |

### Produktionshinweise

- Immer hinter einem Reverse Proxy (nginx) betreiben
- Der Proxy muss `X-Forwarded-For` zuverlässig setzen — die Ophix-Authentifizierung hängt von der Quell-IP ab
- `DEBUG=False` in der Produktion beibehalten
- Die Datei `.env` separat von der Datenbank sichern — sie enthält Verschlüsselungsschlüssel
