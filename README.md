# 🌊 **NautiPi – Das zentrale Software-Management für deinen Raspberry Pi Bordcomputer**

## Was ist NautiPi und warum ist es wichtig?

**NautiPi** ist die universelle, benutzerfreundliche Schaltzentrale für den Raspberry Pi als Bordcomputer auf Segel- und Motorbooten. Mit NautiPi kann jeder – egal ob Technikprofi oder Freizeitsegler – den Raspberry Pi auf einfache Weise als Herzstück der Bordelektronik nutzen, verwalten und jederzeit flexibel erweitern.

Viele Bootscrews möchten heute die Vorteile der Digitalisierung nutzen: Navigation, Sensordaten, Wetter, Musik und mehr – alles möglichst einfach, modular und unabhängig von proprietären Komplettlösungen. Hier setzt NautiPi an:

* **Für Anwender (Segler):**
  NautiPi macht den Einstieg in den Raspberry Pi Bordcomputer so einfach wie möglich. Ohne Linux-Kenntnisse und ohne komplexe Installationsroutinen kann jeder ein modernes, sicheres und wartbares Bord-IT-System aufsetzen. Ein Einrichtungsassistent führt Schritt für Schritt von der WLAN- und Hotspot-Einrichtung bis hin zur Installation und Konfiguration beliebter Bord-Services wie AvNav oder SignalK. Das alles läuft direkt im Browser – ganz ohne Terminal-Befehle. Sicherheitsfeatures wie HTTPS, Nutzerverwaltung und Firewall sorgen für Schutz und sorgenfreien Betrieb an Bord und auf See. Offline-Betrieb, progressive Web App (PWA) Funktionalität und Backup-/Restore-Optionen erhöhen die Ausfallsicherheit und Bedienbarkeit im Bordalltag.

* **Für Entwickler:**
  NautiPi ist ein offenes, modernes und modulares Framework zur Verwaltung und Integration von Marine-Softwareprojekten. Über standardisierte, validierte YAML-Deskriptoren können neue Services ohne Core-Fork oder tiefen Code-Eingriff eingebunden werden. Plugins und eigene Services können nach einer klaren Plugin-Spezifikation samt Lifecycle-Hooks, Version-Constraints und validierter Schnittstelle entwickelt werden – Core-Stabilität bleibt stets erhalten. Automatische Tests, Code- und YAML-Linting, sowie eine anschauliche, stets aktuelle Entwicklerdokumentation (Markdown, OpenAPI, Mermaid-Diagramme) machen die Mitarbeit einfach und nachhaltig.

**NautiPi** ist damit ein wichtiger Schritt, die Digitalisierung und Automatisierung an Bord einfach, sicher und unabhängig zu gestalten – ein Gewinn für Segler, Entwickler und die Community gleichermaßen!

---

## 🧩 1. Grundsätzliche Strukturübersicht:

NautiPi ist modular aufgebaut und besteht aus zwei Kernkomponenten:

* **Backend:**

  * Verwaltung von Systemdiensten, Installation, Konfiguration, Updates, Sicherheit, Logging, und automatischer Migration bei Breaking Changes
  * Kommunikation über REST-API/WebSocket (Single Source of Truth)
  * Verwaltung über standardisierte, versionierte und validierte YAML-Beschreibungsdateien pro Service/Plugin mit Migrations-Layer
  * Secrets wie Passwörter/JWT-Keys werden nicht in YAML gespeichert, sondern in einem verschlüsselten Vault (z.B. age, sops)

* **WebUI (Frontend):**

  * Benutzerfreundlich, intuitiv, einfach wartbar und leicht erweiterbar
  * Responsive für Desktop, Tablet und Smartphone
  * Progressive Web App (PWA)-fähig für Offline-Nutzung

---

## 📁 2. Dateistruktur (Empfohlene Verzeichnisstruktur):

```plaintext
nautipi/
├── backend/
│   ├── core/
│   │   ├── installer.py
│   │   ├── updater.py
│   │   ├── configurator.py
│   │   ├── hotspot.py
│   │   ├── system.py
│   │   ├── auth.py
│   │   ├── security.py
│   │   ├── plugin_manager.py
│   │   └── vault.py
│   ├── services/
│   │   ├── avnav.yml
│   │   ├── signalk.yml
│   │   └── ...
│   ├── plugins/
│   │   └── (Third-party service descriptors)
│   ├── api/
│   │   └── rest_api.py
│   ├── main.py
│   ├── logging.py
│   ├── metrics.py
│   ├── migrations/
│   │   └── (YAML-Schema-Migrations)
│   ├── pyproject.toml         # Poetry für Reproducibility
│   └── sbom/                  # CycloneDX SBOM aus CI
│
├── frontend/
│   ├── src/
│   │   ├── lib/
│   │   │   └── design-system/
│   │   └── routes/
│   └── package.json
│
├── docs/
│   └── (Dokumentation & Entwickleranleitung, Plugin Cookbook)
│
├── .devcontainer/
│   └── devcontainer.json      # Für VS Code/Podman
│
└── setup/
    ├── install.sh
    └── setup.py
```

---

## 🛠️ 3. Backend-Technologie (Python):

Das Backend wird in **Python** umgesetzt, da Python Standard auf Raspberry Pi OS ist, einfach zu warten ist und mit minimalen Abhängigkeiten auskommt.

* **Framework:** [FastAPI](https://fastapi.tiangolo.com/)

  * Schnell, leichtgewichtig, modern (async), OpenAPI-Doku out-of-the-box
  * REST-API & WebSocket nativ unterstützt
  * Automatische API-Dokumentation und OpenAPI-Schema (redocly/swagger)
  * **Deployment:**

    * Uvicorn mit Worker/Async-Konfiguration geprüft (`--workers 1 --loop uvloop --http httptools` als Default, erweiterbar bei Mehrlast)
    * Optional: Gunicorn für Workermodell, falls Skalierung notwendig

* **Serviceverwaltung via YAML**:

  * Jeder Service (AvNav, SignalK, Plugins) erhält eine eigene YAML-Beschreibungsdatei mit semver-Header
  * Validierung der YAML-Dateien mittels JSON-Schema (`pydantic`, `pykwalify`)
  * **Migrations-Layer:** Automatische Migration der YAMLs bei Breaking Changes (z.B. via pydantic-Revision, semver im Header)

* **Plugin-System:**

  * Plugins via Python Entry Points
  * Plugin-Spec inkl. README, pydantic-Schema, Lifecycle-Hooks, Version-Constraints
  * Erweiterungen bleiben rückwärtskompatibel und update-sicher

* **CLI**

  * Hook-basiertes CLI (`nautipi [install|update|tail]`), REST-Calls werden für Skripting/Automatisierung gewrapped

* **Python-Bibliotheken:**

  * fastapi
  * uvicorn\[standard]
  * pyyaml
  * python-dotenv
  * pydantic
  * structlog
  * poetry (für Dependency/Build Management)
  * requests
  * subprocess
  * pluggy (für Plugins)
  * sops/age (Secrets Vault)
  * prometheus\_client (Metrics Endpoint)

* **Security & Deployment:**

  * **Auth**: lokaler User (PAM)
  * **Secrets-Vault:** Passwords/Keys werden in einem verschlüsselten File gespeichert (Age/SOPS), nie im YAML
  * **OTA/Self-Updates:** Rolling Release via GitHub Releases, rsync-Delta-Updates (Fallback auf Full Download, Checksummen + Minisign-Signatur)
  * **Logging:** structlog, Ringbuffer (Loki-Style) für persistente Logs, Download-Button im WebUI
  * **Metrics:** `/metrics`-Endpoint für Prometheus/Alerting an Bord

* **Installation als Service (systemd):**

  * Automatische Einrichtung bei Installation
  * Self-Update-Mechanismus über GitHub Releases

* **Dev-Experience:**

  * Dev-Container (`.devcontainer.json`) für schnellen Einstieg
  * Pre-commit Hooks (ruff, black, mypy, pyproject-tasks)
  * Automatisierte Tests, Linting und SBOM in CI/CD

---

## 🌐 4. WebUI-Frontend (SvelteKit):

Für die WebUI kommt **SvelteKit** zum Einsatz:

* **SvelteKit**

  * Modern, performant, intuitiv und einfach zu erweitern
  * Deployment als statisches Bundle über Caddy/nginx Reverse Proxy (gleichzeitig TLS/HTTPS)
  * Progressive Web App (PWA): Offline-Nutzung an Bord ohne Internet
* **Styling**: TailwindCSS (schnell, responsive, modern)
* **Design-System:** Gemeinsame Komponentenbibliothek (Tailwind + Radix/Headless UI), Styles und Tokens zentral gepflegt (Look & Feel bleibt einheitlich)
* **API-Kommunikation:** Axios oder Fetch

**Features im WebUI:**

* Step-by-Step Onboarding/Wizard (WLAN, Hotspot, User, SSH, Updates…)
* Service-Installations- und Verwaltungsoberfläche (mit Plugin-Unterstützung und Migrationstools)
* Konfigurationseditor pro Service (YAML-Deskriptor steuert, welche Optionen editierbar sind, Validation und Migrations-Layer)
* Zentrale Log-Ansicht (Ringbuffer, Live Logtail via WebSocket, Download für Support)
* Self-Update-Button, Anzeige Systemstatus
* Metrics/Health-Status im Dashboard
* Plugin-Galerie: Drittanbieter können ihre Service-YAMLs bereitstellen, die über das WebUI importiert werden

---

## 🗄️ 5. Plugins & Erweiterbarkeit

* Services sind als YAML-Deskriptoren modular integriert (nativ oder Drittanbieter/Plugins)
* Plugins werden via Entry Points registriert und über YAML- und Python-API validiert (Plugin-Spec)
* Plugins können von Drittentwicklern einfach bereitgestellt, importiert und direkt im WebUI aktiviert werden, inkl. automatischer Validierung und Version-Check
* Automatische Migration bei Breaking Changes via Semver und Migrations-Layer

---

## 🔒 6. Security & Updates

* **Login/Authentifizierung** PAM-Login
* **OTA/Self-Updates** via GitHub Releases und Self-Update-Button im WebUI (rolling release, rsync-delta, Checksummen-Validierung, Minisign)
* **Log- und Fehleranalyse:** structlog, Ringbuffer für persistente Logs, zentrale Ansicht im WebUI, Logtail live via WebSocket
* **Secrets-Vault:** SOPS/Age für verschlüsselte Speicherung sensibler Daten, niemals im Klartext im YAML
* **Metrics:** Prometheus-Endpoint `/metrics`

---

## 📜 7. Installation und Initialsetup:

### Installationsworkflow (Einzeiler nach Flashing von Raspberry OS Lite):

```bash
curl -sSL https://github.com/youruser/NautiPi/raw/main/setup/install.sh | bash
```

* Skript installiert alle Abhängigkeiten (Python, Poetry, FastAPI, nginx/Caddy, systemd-unit)
* Hotspot automatisch aktiviert (via `hostapd` + `dnsmasq`)
* Systemd-Service eingerichtet und startet automatisch
* TLS/HTTPS direkt im Setup aktiviert, Zugangsdaten im Wizard vergeben
* Firewall und Fail2Ban mit sicheren Presets aktiviert

---

## 🚀 8. Start-Workflow für User:

* Raspberry Pi OS Lite flashen
* Installationsskript ausführen
* Hotspot automatisch gestartet → Verbindung herstellen
* Wizard via WebUI führt User durch:
  * WLAN konfigurieren
  * SSH/FTP aktivieren
  * Nutzer & Passwörter setzen
* Hotspot deaktiviert, IP & Zugangsdaten werden angezeigt
* Services via WebUI installieren, verwalten & konfigurieren (inkl. Plugins)
* Zentrale Verwaltung, Updates und Logauswertung direkt im WebUI

---

## 📖 9. Dokumentation & Entwicklerfreundlichkeit:

* **Markdown-Dokumentation** im `docs`-Ordner des Projekts
* **OpenAPI-Schema** (Redocly/Swagger) für REST-API
* **Mermaid-Diagramme** für Architektur/Workflow
* Beispiel-YAML für Drittanbieter bereitstellen (Plugin-Entwicklung)
* Plugin-Cookbook (Hello-World-Plugin) als Doku-Vorlage
* GitHub-Wiki oder GitHub-Pages
* **Automatische Dokumentation** aus API & YAML (Markdown, Mermaid, Redocly)
* Linting/Validierung per CI-Workflow (GitHub Actions), Pre-commit Hooks (ruff/black/mypy)
* **Schema-Validierung:** Fehler in Plugins/YAMLs werden früh erkannt und im WebUI mitgeteilt
* Dev-Container (VS Code/Podman) für schnellen Einstieg

---

## ✅ 10. Zusammenfassung der ausgewählten Technologien:
 
| Bereich         | Technologie                            | Gründe                                               |
| --------------- | -------------------------------------- | ---------------------------------------------------- |
| Backend         | Python + FastAPI + Poetry              | Schlank, performant, REST/API-unterstützt            |
| Web Frontend    | SvelteKit + TailwindCSS + PWA          | Einfach, modern, wartbar, wenig Overhead             |
| Konfiguration   | YAML + Schema-Validation + Migration   | Klar, minimalistisch, fehlertolerant, update-sicher  |
| Installation    | Shellskript                            | Einfach, minimalistisch, wartungsfreundlich          |
| Sicherheit      | Auth, Vault                            | Sichere und nachvollziehbare Basis                   |
| Logging         | structlog, Ringbuffer, Web-Logtail     | Schnelle Fehlersuche, Support                        |
| Erweiterbarkeit | Plugins via Entry Points & YAML        | Schnell und ohne Core-Fork erweiterbar               |
| CI/CD           | GitHub Actions, Pre-commit, SBOM       | Zuverlässig, nachvollziehbar, contributor-freundlich |

---

## ⚙️ 11. Beispiel für REST-Endpunkte (Backend):

```plaintext
GET /api/services
GET /api/services/{name}
POST /api/services/{name}/install
POST /api/services/{name}/update
POST /api/services/{name}/configure
GET /api/system/status
GET /api/logs/tail
POST /api/selfupdate
GET /metrics
```

