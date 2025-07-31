# 🌊 **NautiPi – The Central Software Management for Your Web-Based Raspberry Pi Onboard Computer**

---

## What is NautiPi and why is it important?

**NautiPi** is the universal, user-friendly control center for using the Raspberry Pi as a web-based onboard computer on sailboats and motorboats. With NautiPi, anyone—whether tech pro or leisure sailor—can easily use the Raspberry Pi as the heart of their onboard electronics, manage it, and flexibly extend it at any time. NautiPi focuses strongly on web-based applications without a Linux desktop. All interfaces are accessible via browser, with no need for a permanently attached display or input devices like mouse and keyboard on the Pi.

Many boat crews today want to take advantage of digitalization: navigation, sensor data, weather, music, and more—all as simple, modular, and independent from proprietary all-in-one solutions as possible. This is where NautiPi comes in:

* **For Users (Sailors):**
  NautiPi makes getting started with the Raspberry Pi onboard computer as easy as possible. With no need for Linux knowledge or complex installation routines, anyone can set up a modern, secure, and maintainable onboard IT system. A setup wizard guides users step-by-step—from Wi-Fi and hotspot setup to installing and configuring popular onboard services like AvNav or SignalK. Everything runs directly in the browser—no terminal commands required.

* **For Developers:**
  NautiPi is an open, modern, and modular framework for managing and integrating marine software projects. Through standardized [YAML descriptors](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml), new services can be integrated without core forks or deep code changes. Plugins are handled just like native service YAMLs, but can also be imported via the WebUI. Clear, always up-to-date developer documentation makes contributing simple and sustainable.

**NautiPi** is an important step toward making onboard digitalization and automation easy, secure, and independent—a win for sailors, developers, and the community alike!

---

## 💾 Basic Project Overview

### 🚀 User Workflow

* Flash Raspberry Pi OS Lite
* Run installation script
* Hotspot starts automatically → connect to it
* Wizard via WebUI guides user through:

  * Wi-Fi configuration
  * Enable SSH/FTP
  * Set user & passwords
* Hotspot deactivates, IP & access data displayed
* Install, manage & configure services via WebUI (including plugins)
* Central management, updates, and log analysis directly in WebUI

## 🧩 Basic Structure Overview

NautiPi is modular and consists of two core components:

* **Backend:**

  * Manages system services, installation, configuration, updates, security, and logging
  * Communicates via REST API/WebSocket (Single Source of Truth)
  * Managed via standardized and versioned [YAML descriptor files](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) per service/plugin

* **WebUI (Frontend):**

  * User-friendly, intuitive, easy to maintain and extend
  * Responsive for desktop, tablet, and smartphone

---

## 📁 File Structure

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
│   └── (Documentation & developer guide, plugin cookbook)
│
└── setup/
    ├── install.sh
    └── setup.py
```

---

## 🛠️ Backend Technology (Python)

The backend is implemented in **Python**, as Python is standard on Raspberry Pi OS, easy to maintain, and requires minimal dependencies.

* **Framework:** [FastAPI](https://fastapi.tiangolo.com/)

  * Fast, lightweight, modern (async)
  * REST API & WebSocket natively supported:

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

  * Uvicorn with worker/async configuration as default (`--workers 1 --loop uvloop --http httptools`), can be extended for higher load
  * Optional: Gunicorn for worker model if scaling is needed

* **Service Management via YAML:**

  * Each service (AvNav, SignalK, plugins) gets its own [YAML descriptor file](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) with semver header

* **Plugin System:**

  * Plugins are imported via [YAML](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml)
  * Plugins are identical to service [YAMLs](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml), just not shipped with the core package

* **Security & Deployment:**

  * **Auth:** local Linux user (or PAM)
  * **OTA/Self-Updates:** `git pull` and restart
  * **Logging:** systemd tail logs, download button in WebUI

* **Install as a Service (systemd):**

  * Automatic setup during installation
  * Self-update mechanism

---

## 🌐 WebUI Frontend (SvelteKit)

**SvelteKit** is used for the WebUI:

* **SvelteKit**

  * Modern, performant, intuitive, and easy to extend
* **Styling:** TailwindCSS (fast, responsive, modern)
* **Design System:** Shared component library (Tailwind + Radix/Headless UI), styles and tokens managed centrally (look & feel stays consistent)
* **API Communication:** Axios or Fetch

**Features in the WebUI:**

* Step-by-step onboarding/wizard (Wi-Fi, hotspot, user, SSH, updates…)
* [Service](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) installation and management interface (with plugin support)
* Configuration editor per service ([YAML descriptor](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) defines which options are editable)
* (Low priority) Central log view (systemd logs via tail, download for support)
* Self-update button
* (Low priority) Plugins: third parties can provide their service YAMLs, which can be imported and displayed via WebUI

---

## 🗄️ Plugins & Extensibility (Low Priority)

* Services are integrated modularly as [YAML descriptors](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) (native or third-party/plugins)
* Plugins can be easily provided by third-party developers, imported, and enabled directly in the WebUI
* [Plugin YAML](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml) validation during import conceivable in the future

---

## 🔒 Security & Updates

* **Login/Authentication** via local user (optionally PAM login)
* **OTA/Self-Updates** via `git pull` and restart
* **Log and error analysis:** systemd logs via tail and download for support

---

## 📜 Installation and Initial Setup

### Installation workflow (one-liner after flashing Raspberry OS Lite):

```bash
curl -sSL https://github.com/youruser/NautiPi/raw/main/setup/install.sh | bash
```

* Script installs all dependencies
* Hotspot activated automatically (via `hostapd` + `dnsmasq`)
* Systemd service set up and starts automatically

---

## 📖 Documentation & Developer Friendliness

* **Markdown documentation** in the `docs` folder of the project
* **OpenAPI schema** for REST API
* Example YAML provided for third parties (plugin development)
* Plugin cookbook (hello-world plugin) as doc template
* GitHub Wiki or GitHub Pages
* **Automatic documentation** from API & YAML (Markdown, Mermaid, Redocly)

---

## 🌐 Positioning in the Open Source Boat Software Scene

NautiPi positions itself as a lightweight, web-based management framework for Raspberry Pi onboard computers—with a clear focus on modern web technologies like SignalK and AvNav. In contrast to established systems like OpenPlotter or Bareboat (BBN) OS, NautiPi avoids a classic desktop GUI or chartplotter frontends, offering instead a modular WebUI for easy installation, configuration, and management of services.

While OpenPlotter and BBN OS provide full operating system images, including graphical user interfaces, chartplotters (e.g., OpenCPN), media playback, and extensive hardware integration, NautiPi targets users and developers seeking a slim, headless, or hotspot-based browser solution.

NautiPi is the perfect complement to the ecosystem as a **modular boat service manager**, serving as an alternative for users who want to focus on network-based services and prefer to skip heavy GUIs or dedicated displays.

---

### 🔍 Overview: NautiPi Compared to the Open Source Ecosystem

| **System / Project**  | **Type / Focus**                          | **GUI Type**              | **SignalK & AvNav**          |   |
| --------------------- | ----------------------------------------- | ------------------------- | ---------------------------- | - |
| **NautiPi**           | Web-based framework                       | WebUI / Wizard (Headless) | Integrated via YAML services |   |
| **OpenPlotter**       | Complete Raspberry Pi OS image for marine | Desktop GUI (X Server)    | Supported via plugins        |   |
| **Bareboat (BBN) OS** | Linux distro focused on onboard computers | Desktop GUI + touchscreen | SignalK & AvNav preinstalled |   |

---
