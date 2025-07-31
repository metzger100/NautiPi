# 🌊 **NautiPi – Das zentrale Software-Management für deinen Raspberry Pi Bordcomputer**

---

## Was ist NautiPi und warum ist es wichtig?

**NautiPi** ist die universelle, benutzerfreundliche Schaltzentrale für den Raspberry Pi als Bordcomputer auf Segel- und Motorbooten. Mit NautiPi kann jeder – egal ob Technikprofi oder Freizeitsegler – den Raspberry Pi auf einfache Weise als Herzstück der Bordelektronik nutzen, verwalten und jederzeit flexibel erweitern.

Viele Bootscrews möchten heute die Vorteile der Digitalisierung nutzen: Navigation, Sensordaten, Wetter, Musik und mehr – alles möglichst einfach, modular und unabhängig von proprietären Komplettlösungen. Hier setzt NautiPi an:

* **Für Anwender (Segler):**
  NautiPi macht den Einstieg in den Raspberry Pi Bordcomputer so einfach wie möglich. Ohne Linux-Kenntnisse und ohne komplexe Installationsroutinen kann jeder ein modernes, sicheres und wartbares Bord-IT-System aufsetzen. Ein Einrichtungsassistent führt Schritt für Schritt von der WLAN- und Hotspot-Einrichtung bis hin zur Installation und Konfiguration beliebter Bord-Services wie AvNav oder SignalK. Das alles läuft direkt im Browser – ganz ohne Terminal-Befehle.

* **Für Entwickler:**
  NautiPi ist ein offenes, modernes und modulares Framework zur Verwaltung und Integration von Marine-Softwareprojekten. Über standardisierte YAML-Deskriptoren können neue Services ohne Core-Fork oder tiefen Code-Eingriff eingebunden werden. Plugins werden wie die nativen Service-Yamls behandelt können aber über die WebUI importiert werden. Eine anschauliche, stets aktuelle Entwicklerdokumentation machen die Mitarbeit einfach und nachhaltig.

**NautiPi** ist damit ein wichtiger Schritt, die Digitalisierung und Automatisierung an Bord einfach, sicher und unabhängig zu gestalten – ein Gewinn für Segler, Entwickler und die Community gleichermaßen!

---

## 💾 Grundsätzliche Projektübersicht:

### 🚀 Workflow für User:

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

## 🧩 Grundsätzliche Strukturübersicht:

NautiPi ist modular aufgebaut und besteht aus zwei Kernkomponenten:

* **Backend:**

  * Verwaltung von Systemdiensten, Installation, Konfiguration, Updates, Sicherheit und Logging
  * Kommunikation über REST-API/WebSocket (Single Source of Truth)
  * Verwaltung über standardisierte und versionierte YAML-Beschreibungsdateien pro Service/Plugin

* **WebUI (Frontend):**

  * Benutzerfreundlich, intuitiv, einfach wartbar und leicht erweiterbar
  * Responsive für Desktop, Tablet und Smartphone

---

## 📁 Dateistruktur:

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
│   │   └── plugin_manager.py
│   ├── services/
│   │   ├── avnav.yml
│   │   ├── signalk.yml
│   │   └── ...
│   ├── plugins/
│   │   └── (Third-party service descriptors)
│   ├── api/
│   │   └── rest_api.py
│   ├── main.py
│   └── logging.py
│
├── frontend/
│   └── webui/
│       ├── src/
│       │   ├── lib/
│       │   │   └── design-system/
│       │   └── routes/
│       └── package.json
│
├── docs/
│   └── (Dokumentation & Entwickleranleitung, Plugin Cookbook)
│
└── setup/
    ├── install.sh
    └── setup.py
```

---

## 🛠️ Backend-Technologie (Python):

Das Backend wird in **Python** umgesetzt, da Python Standard auf Raspberry Pi OS ist, einfach zu warten ist und mit minimalen Abhängigkeiten auskommt.

* **Framework:** [FastAPI](https://fastapi.tiangolo.com/)

  * Schnell, leichtgewichtig, modern (async)
  * REST-API & WebSocket nativ unterstützt:

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

  * **Deployment:**

    * Uvicorn mit Worker/Async-Konfiguration geprüft (`--workers 1 --loop uvloop --http httptools` als Default, erweiterbar bei Mehrlast)
    * Optional: Gunicorn für Workermodell, falls Skalierung notwendig

* **Serviceverwaltung via YAML**:

  * Jeder Service (AvNav, SignalK, Plugins) erhält eine eigene YAML-Beschreibungsdatei mit semver-Header

* **Plugin-System:**

  * Plugins via yaml import.
  * Plugins sind identisch wie die service yamls, nur kommen diese nicht mit dem core package mit

* **Security & Deployment:**

  * **Auth**: lokaler Linux-User (oder PAM)
  * **OTA/Self-Updates:** `git pull` und restart
  * **Logging:** systemd tail logs, Download-Button im WebUI

* **Installation als Service (systemd):**

  * Automatische Einrichtung bei Installation
  * Self-Update-Mechanismus

---

## 🌐 WebUI-Frontend (SvelteKit):

Für die WebUI kommt **SvelteKit** zum Einsatz:

* **SvelteKit**
  * Modern, performant, intuitiv und einfach zu erweitern
* **Styling**: TailwindCSS (schnell, responsive, modern)
* **Design-System:** Gemeinsame Komponentenbibliothek (Tailwind + Radix/Headless UI), Styles und Tokens zentral gepflegt (Look & Feel bleibt einheitlich)
* **API-Kommunikation:** Axios oder Fetch

**Features im WebUI:**

* Step-by-Step Onboarding/Wizard (WLAN, Hotspot, User, SSH, Updates…)
* Service-Installations- und Verwaltungsoberfläche (mit Plugin-Unterstützung)
* Konfigurationseditor pro Service (YAML-Deskriptor steuert, welche Optionen editierbar sind)
* (Keine Prio) Zentrale Log-Ansicht (Systemd-Logs via Tail auslesen, Download für Support)
* Self-Update-Button
* (Keine Prio) Plugins: Drittanbieter können ihre Service-YAMLs bereitstellen, die über das WebUI importiert und angezeigt werden

---

## 🗄️ Plugins & Erweiterbarkeit (Keine Prio)

* Services sind als YAML-Deskriptoren modular integriert (nativ oder Drittanbieter/Plugins)
* Plugins können von Drittentwicklern einfach bereitgestellt, importiert und direkt im WebUI aktiviert werden.
* Plugin-YAML validierung beim import in Zukunft vorstellbar

---

## 🔒 Security & Updates

* **Login/Authentifizierung** lokaler User (eventuell PAM-Login)
* **OTA/Self-Updates** `git pull` und restart
* **Log- und Fehleranalyse:** systemd logs via tail und Download für support

---

## 📜 Installation und Initialsetup:

### Installationsworkflow (Einzeiler nach Flashing von Raspberry OS Lite):

```bash
curl -sSL https://github.com/youruser/NautiPi/raw/main/setup/install.sh | bash
```

* Skript installiert alle Abhängigkeiten
* Hotspot automatisch aktiviert (via `hostapd` + `dnsmasq`)
* Systemd-Service eingerichtet und startet automatisch

---

## 📖 Dokumentation & Entwicklerfreundlichkeit:

* **Markdown-Dokumentation** im `docs`-Ordner des Projekts
* **OpenAPI-Schema** für REST-API
* Beispiel-YAML für Drittanbieter bereitstellen (Plugin-Entwicklung)
* Plugin-Cookbook (Hello-World-Plugin) als Doku-Vorlage
* GitHub-Wiki oder GitHub-Pages
* **Automatische Dokumentation** aus API & YAML (Markdown, Mermaid, Redocly)

---

## 🌐 Einbettung in die Open‑Source‑Bootsoftware‑Szene

NautiPi positioniert sich bewusst als leichtgewichtiges, webbasiertes Management‑Framework für Raspberry Pi Bordcomputer — mit klarem Fokus auf moderne Webtechnologien wie SignalK und AvNav. Im Vergleich zu etablierten Systemen wie OpenPlotter oder Bareboat (BBN) OS verzichtet NautiPi auf eine klassische Desktop‑GUI oder Chartplotter‑Frontends und bietet stattdessen eine modulare WebUI zur einfachen Installation, Konfiguration und Verwaltung von Services.

Während OpenPlotter und BBN OS vollwertige Betriebssystem‑Images liefern, inklusive grafikbasierter Nutzeroberfläche, Chartplotter (z. B. OpenCPN), Medienwiedergabe und umfangreicher Hardware‑Integration, zielt NautiPi auf Anwender und Entwickler, die eine schlanke, headless oder hotspot‑basierte Lösung im Browser nutzen möchten.

NautiPi ergänzt die Szene ideal als **modularer Boot-Service‑Manager**, als Alternative für User, die sich auf Netzwerk‑basierte Dienste konzentrieren und auf aufwändige GUIs/feste Displays verzichten möchten.

---

### 🔍 Übersicht: NautiPi im Vergleich zum Open Source Umfeld

| **System / Projekt**       | **Art / Fokus**                             | **GUI‑Typ**                    | **SignalK & AvNav**                                                                      |
| -------------------------- | ------------------------------------------- | ------------------------------ | -----------------------------------------------------------------------------------------|
| **NautiPi**                | Web‑basiertes Framework                     | WebUI / Wizard (Headless)      | Integriert via YAML‑Services                                                             |
| **OpenPlotter**            | Komplettes Raspberry Pi OS‑Image für Marine | Desktop‑GUI (X‑Server)         | Unterstützt via Plugins                                                                  ||
| **Bareboat (BBN) OS**      | Linux‑Distro mit Fokus auf Bootscomputer    | Desktop‑GUI + Touchscreen      | SignalK & AvNav vorinstalliert                                                           |

---
