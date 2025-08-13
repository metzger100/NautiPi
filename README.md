---

# 🌊 NautiPi – The Central Software Management for Your Web-Based Raspberry Pi Onboard Computer

---

## What is NautiPi and why does it matter?

**NautiPi** is the universal, user-friendly control center for running a Raspberry Pi as a **web-based onboard computer** on sailboats and motorboats. With NautiPi, anyone, whether power user or weekend sailor, can use the Pi as the heart of onboard electronics, manage it, and extend it at any time. NautiPi is focused on **headless, web-based** apps (no Linux desktop). Everything is accessible in the browser; no permanently attached screen, mouse, or keyboard required.

Many crews want modular digitalization without being locked into proprietary all-in-one solutions: navigation, sensor data, weather, music, and more. This is where NautiPi shines:

* **For Users (Sailors):**
  NautiPi makes getting started with a Pi onboard computer as simple as possible. No Linux knowledge, no complex terminal routines. A setup wizard guides you step by step from Wi-Fi/hotspot to installing and configuring popular onboard services like AvNav or SignalK. Everything runs directly in the browser.

* **For Developers:**
  NautiPi is an open, modern, modular framework for integrating and managing marine software. Through standardized, versioned [**YAML descriptors**](https://github.com/metzger100/NautiPi/blob/main/service-template.yaml), new services can be added without forking the core. Plugins behave just like native service YAMLs and can be imported via the WebUI. Clear, up-to-date docs make contributions sustainable.

**Bottom line:** NautiPi aims to make onboard digitalization easy, safe, and independent; benefiting sailors, developers, and the wider community.

---

## 💾 Basic Project Overview

### 🚀 User Workflow

* Flash Raspberry Pi OS Lite
* Run one-line installation script
* **Fail-safe Hotspot** starts automatically → connect to it
* First-run wizard (WebUI) guides you through:

  * Wi-Fi configuration (join marina LAN or your own router)
  * Enable SSH/FTP
  * Set user & passwords

* Hotspot policy applied (see networking section)
* Install, manage & configure services via WebUI (incl. plugins)
* Central updates, logs, and health directly in the WebUI

---

## 🧠 Desired State & Reconciliation (Operator Pattern)

NautiPi persists a **Desired State** (installed services, versions, inputs) in a local store (e.g. SQLite). A background **Reconciler** continuously compares desired vs. actual and **repairs drift** (e.g., restarts services, reapplies config, re-links integrations) in an idempotent manner. This turns one-time install scripts into a **self-healing** system, ideal on boats far from shore support.

---

## 📦 Jobs, Progress & Live Logs

Long-running actions (install/update/configure/reconfigure/uninstall) run as **Jobs** with states: `queued → running → succeeded/failed/canceled`.
The WebUI shows:

* **Progress** and **step timeline** (per command)
* **Live log tail** via WebSocket (systemd tail, command output)
* **Job-scoped sudo** prompt (see security)
  Jobs are cancellable when safe.

---

## 🔐 Security & Sudo Model (no root backend)

* The backend does **not** run as root.
* **Sudo prompts** appear in the WebUI **once per job** when a command requires elevation (`sudo: true`). The password is:

  * validated via PAM,
  * held **in memory only** for the job’s lifetime (short TTL),
  * never persisted.

* Sudo is restricted to an **allow-list** of commands used by NautiPi (minimal surface).
* UI passwords/inputs marked as `secret` are masked in logs.
* PAM login for the WebUI (local user).
* Rate limits and lockouts for authentication attempts.

> ⚠️ **Deliberate HTTP-only:** Many boats are offline for weeks. Public CAs may be unreachable and self-signed TLS confuses users. **NautiPi serves over HTTP on local networks by design.** Do **not** expose NautiPi directly to the public Internet. If you must, use your own router/VPN/proxy at your own risk.

---

## 📶 Networking Model (Fail-Safe Hotspot, “Safe Networks” & Firewall)

**Goals:** Keep the onboard devices always connected via the Pi hotspot; supply Internet via marina Wi-Fi or an external travel router; strongly isolate unsafe uplinks.

* **Fail-Safe Hotspot (always-on):**

  * The NautiPi hotspot remains enabled so onboard devices stay connected.
  * If the onboard “safe” network disappears, the hotspot **stays** available.
  * An option lets you **turn off hotspot when a marked safe network is connected** (only for those safe networks).

* **Safe Networks:**
  Users can mark a network (LAN or Wi-Fi; e.g., an external travel router creating a boat LAN) as **safe**.

  * For **safe** networks including the own one: the firewall **does not** block boat protocols in that direction (mDNS, SignalK, MQTT, NMEA, etc.).
  * For **unsafe** uplinks (e.g., raw marina Wi-Fi): the firewall enforces a **strict egress policy** (NAT to Internet only) and blocks inbound access from uplink to the boat LAN.

* **Implementation choices:**

  * **NetworkManager** (with **wpa\_supplicant**) + **systemd-resolved**.
  * **Avahi** provides mDNS (`nautipi.local`) and a **captive portal** for first-run convenience.
  * Firewall via **nftables**, generated from policy + optional per-service hints (see YAML template).

---

## 🧱 Storage & SD-Card Longevity

* journald rate limiting and size caps
* log2ram / zram swap
* conservative mount options where appropriate
* per-service log download (support bundle)

---

## 🧩 Project Structure

```plaintext
nautipi/
├── backend/
│   ├── core/
│   │   ├── installer.py
│   │   ├── updater.py
│   │   ├── configurator.py
│   │   ├── reconciler.py           # desired state loop
│   │   ├── jobs.py                 # job queue + WS log streaming
│   │   ├── hotspot.py
│   │   ├── network.py              # NetworkManager + policies
│   │   ├── firewall.py             # nftables rules (safe/unsafe)
│   │   ├── system.py
│   │   ├── auth.py
│   │   ├── security.py
│   │   └── plugin_manager.py
│   ├── services/                   # built-in descriptors and user-imported descriptors
│   │   ├── avnav.yml
│   │   ├── signalk.yml
│   │   └── ...
│   ├── api/
│   │   └── rest_api.py
│   ├── state.db                    # SQLite (desired state, jobs, events)
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

* **Language:** Python (standard on Pi, low friction)
* **Framework:** **FastAPI** (async, lightweight, OpenAPI built-in)
* **Serving:** Uvicorn (`--workers 1 --loop uvloop --http httptools`)
* **State:** SQLite for desired state, jobs, events
* **Service Management via YAML:** Each service (AvNav, SignalK, plugins) has a versioned YAML descriptor
* **Plugin System:** Import plugin YAMLs via WebUI; validate against a strict schema
* **Auth:** Local Linux user (PAM optional)
* **Self-Update:** Versioned releases (tar/deb) + signature + restart
* **Logging:** systemd tail logs + WebSocket streaming

**Representative API (HTTP-only):**

```plaintext
GET  /api/services
GET  /api/services/{id}
POST /api/services/{id}/install
POST /api/services/{id}/update
POST /api/services/{id}/configure
POST /api/services/{id}/reconfigure
POST /api/services/{id}/uninstall

GET  /api/system/status
GET  /api/logs/tail?unit=...
GET  /api/jobs
GET  /api/jobs/{jobId}
POST /api/jobs/{jobId}/cancel

GET  /api/state             # Desired state
POST /api/state/apply       # Apply desired state

POST /api/sudo/challenge    # Start job-scoped sudo (PAM)
POST /api/sudo/verify

GET  /api/network/status
POST /api/network/connect
POST /api/network/mark-safe
POST /api/network/hotspot-policy

POST /api/selfupdate
GET  /metrics
```

---

## 🌐 WebUI Frontend (SvelteKit)

* **SvelteKit**: modern, fast, intuitive
* **Styling:** TailwindCSS; shared design system (Radix/Headless UI patterns)
* **API:** TypeScript + OpenAPI-generated client; **Zod** for runtime validation
* **Features:**

  * Step-by-step wizard (Wi-Fi, hotspot policy, user, SSH, updates…)
  * Service management UI from **JSON Schema** (auto-forms generated from descriptors)
  * Job timeline with **live logs**
  * i18n (DE/EN), offline-friendly assets, dark mode
  * Config snapshot export/import (tar.gz)

---

## 🗄️ Plugins & Extensibility

* Services are integrated via versioned YAML descriptors (native or third-party).
* Import plugin YAMLs directly in the WebUI; strict validation (schema + basic safety).

---

## 🔒 Security & Updates

* **HTTP-only** on local networks (no TLS by default; see rationale above).
* **Sudo prompts** per job (in-memory, short TTL).
* **Self-updates** via signed release artifacts; rollback snapshots for configs.
* **Support bundle**: collect logs/config snippets for troubleshooting.

---

## 📜 Installation & First-Run

**One-liner after flashing Raspberry Pi OS Lite:**

```bash
curl -sSL https://github.com/youruser/NautiPi/raw/main/setup/install.sh | bash
```

* Script installs dependencies
* Hotspot enabled automatically (hostapd + dnsmasq)
* Systemd service is created and started
* Captive portal via Avahi assists first-time access

---

## 📖 Documentation & Developer Friendliness

* Markdown docs in `docs/`
* OpenAPI schema (used to generate the TS client)
* Example YAML for service/plugin authors
* Plugin cookbook (hello-world)
* Automatic docs generation from API & YAML (Markdown, Mermaid, Redoc-style pages)

---

## 🌐 Positioning in the Open-Source Boat Software Scene

NautiPi is a lightweight **web-based management framework** for Raspberry Pi onboard computers, focusing on modern network-centric tools like SignalK and AvNav. Compared to full OS images like OpenPlotter or Bareboat (BBN) OS, NautiPi avoids desktop GUIs and instead offers a **modular WebUI** for installation, configuration, and lifecycle management.

| **System / Project**  | **Type / Focus**                          | **GUI Type**              | **SignalK & AvNav**          |
| --------------------- | ----------------------------------------- | ------------------------- | ---------------------------- |
| **NautiPi**           | Web-based framework                       | WebUI / Wizard (Headless) | Integrated via YAML services |
| **OpenPlotter**       | Complete Raspberry Pi OS image for marine | Desktop GUI (X Server)    | Supported via plugins        |
| **Bareboat (BBN) OS** | Linux distro for onboard computers        | Desktop GUI + touchscreen | SignalK & AvNav preinstalled |

---

## 🔧 Potential Supported Services

**Navigation / Boat-specific**

* **pypilot (Autopilot) Web-UI** — steer & calibrate via browser; pairs well with SignalK/OpenPlotter.
* **AIS-catcher** — AIS reception with RTL-SDR; small webserver; outputs NMEA (UDP/HTTP/TCP) → perfect source for SignalK/AvNav.
* **OpenWebRX** — multi-user web-SDR (helpful for AIS/VHF monitoring).

**IoT / Automation & Data**

* **Node-RED** — low-code flow editor (browser); great for sensor/MQTT/SignalK flows.
* **Eclipse Mosquitto (MQTT Broker)** — lightweight broker for onboard automation.
* **InfluxDB** — time-series DB with web UI; perfect for sensor histories.
* **Grafana** — dashboards/alerts on top of InfluxDB/Prometheus/…
* **Home Assistant (Supervised)** — central automation hub with web UI.

**System & Networking**

* **Nginx Proxy Manager** — simple reverse-proxy UI (note: NautiPi itself stays HTTP-only).

---

## 🧰 Config Snapshots & Support Bundles

* **Snapshot export/import**: descriptors + user inputs + relevant `/etc` files + compose files → single `tar.gz`.
* **Support bundle**: journald excerpts, `nautipi diag`, network status, versions → downloadable archive.

---
