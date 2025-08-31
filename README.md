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
├─ README.md                        # Start here: quickstart, repo map (this file)
├─ LICENSE
├─ .editorconfig
├─ .gitignore
├─ Makefile                         # Common dev tasks: make dev / test / lint / build
├─ pyproject.toml                   # Backend tooling (uv, ruff, mypy, pytest)
├─ package.json                     # WebUI tooling (SvelteKit, Vite, TS)

# ──────────────────────────────────────────────
# Delivery & ops (build images, systemd, packaging)
# ──────────────────────────────────────────────
├─ docker/
│  ├─ backend.Dockerfile            # FastAPI backend image
│  └─ webui.Dockerfile              # SvelteKit static build → nginx (or node adapter)
├─ deploy/
│  ├─ systemd/
│  │  ├─ nautipi.service            # Main service (HTTP API + workers)
│  │  └─ nautipi-reconciler.timer   # Optional periodic reconcile trigger
│  ├─ packaging/
│  │  ├─ debian/                    # .deb metadata (optional distro packaging)
│  │  └─ tarball/                   # Release tarball scripts
│  └─ firewall/
│     └─ nftables-template.nft      # Template for LAN/Hotspot policy
├─ setup/
│  ├─ install.sh                    # One-liner entrypoint (docs reference this)
│  ├─ uninstall.sh
│  └─ first-boot/                   # Boots the "fail-safe hotspot" & captive portal
│     ├─ enable-hotspot.sh
│     ├─ captive-portal.service
│     └─ avahi-captive.service
├─ scripts/
│  ├─ dev.sh                        # Local dev orchestration
│  ├─ lint.sh                       # ruff/mypy/eslint/etc.
│  ├─ release.sh                    # Version bump + artifact build
│  └─ collect-support-bundle.sh     # Gathers logs/configs for debugging

# ──────────────────────────────────────────────
# Docs & specs (drive UI, validation, and contributor workflow)
# ──────────────────────────────────────────────
├─ docs/
│  ├─ index.md                      # Doc landing page
│  ├─ architecture.md               # Operator model, reconciler, jobs
│  ├─ api.md                        # Generated from OpenAPI (backend/app.py)
│  ├─ plugin-cookbook.md            # Write your own service/plugin descriptors
│  ├─ networking.md                 # Safe networks, hotspot, firewall
│  └─ security.md                   # Sudo model, allow-list, PAM
├─ schemas/                         # JSON Schemas used by validator + WebUI forms
│  ├─ service-descriptor.schema.json
│  ├─ config-editor.schema.json
│  └─ state.schema.json
├─ descriptors/                     # Built-in service descriptors + dev plugins
│  ├─ native/
│  │  ├─ avnav.yaml
│  │  ├─ signalk.yaml
│  │  ├─ mosquitto.yaml
│  │  └─ nodered.yaml
│  └─ plugins/                      # For local development of third-party plugins

# ──────────────────────────────────────────────
# Backend (FastAPI + operator core)
# ──────────────────────────────────────────────
├─ backend/
│  ├─ nautipi/
│  │  ├─ __init__.py
│  │  ├─ app.py                     # FastAPI app factory (OpenAPI source of truth)
│  │  ├─ api/                       # HTTP surface: 1 file per domain
│  │  │  ├─ services.py             # /api/services … install/update/configure
│  │  │  ├─ jobs.py                 # /api/jobs … progress, cancel, logs
│  │  │  ├─ logs.py                 # /api/logs … live tail via WS
│  │  │  ├─ sudo.py                 # /api/sudo … job-scoped elevation
│  │  │  ├─ state.py                # /api/state … desired state apply
│  │  │  ├─ network.py              # /api/network … connect, mark-safe, hotspot policy
│  │  │  ├─ selfupdate.py           # /api/selfupdate … signed updates
│  │  │  └─ metrics.py              # /metrics … Prometheus
│  │  ├─ core/                      # Business logic used by routers
│  │  │  ├─ db.py                   # SQLite + migrations
│  │  │  ├─ models.py               # Pydantic models (shared types)
│  │  │  ├─ jobs.py                 # Job queue & lifecycle (queued → running → done)
│  │  │  ├─ runner.py               # Command runner (sudo-aware, streaming logs)
│  │  │  ├─ reconcilers/            # Operator loop to reach desired state
│  │  │  │  ├─ desired_state.py     # Converge services, apply drift repairs
│  │  │  │  └─ health.py            # Status checks (systemd/http/tcp/custom)
│  │  │  ├─ sudo.py                 # PAM verify + allow-list checks
│  │  │  ├─ security.py             # Auth, rate limiting, lockouts
│  │  │  ├─ descriptors.py          # YAML loader + JSON-schema validation
│  │  │  ├─ network.py              # NetworkManager policy (safe vs uplink), hotspot
│  │  │  └─ firewall.py             # nftables generator (LAN exposure hints)
│  │  ├─ adapters/                  # Thin wrappers over system services
│  │  │  ├─ systemd.py              # start/stop/status for units
│  │  │  ├─ docker.py               # docker run/pull/restart helpers
│  │  │  ├─ compose.py              # docker-compose integration (if used)
│  │  │  └─ logs.py                 # journalctl tail, file tails
│  │  ├─ resources/
│  │  │  ├─ allowlist-sudo.json     # Allowed root commands (tight surface)
│  │  │  └─ compose-templates/      # Optional templated compose snippets
│  │  └─ cli.py                     # `nautipi` admin CLI (diag, snapshot, bundle)
│  └─ tests/                        # pytest (unit + integration)
│     ├─ test_descriptors.py
│     ├─ test_runner.py
│     ├─ test_reconciler.py
│     └─ test_api.py

# ──────────────────────────────────────────────
# Web UI (SvelteKit, offline-friendly)
# ──────────────────────────────────────────────
├─ webui/
│  ├─ src/
│  │  ├─ app.d.ts
│  │  ├─ routes/
│  │  │  ├─ +layout.svelte          # Shell, toasts, theme, i18n loading
│  │  │  ├─ +layout.ts
│  │  │  ├─ +page.svelte            # Dashboard
│  │  │  ├─ wizard/                 # First-run flow (Wi-Fi, hotspot, users, SSH, updates)
│  │  │  ├─ services/               # Install/manage from YAML descriptors
│  │  │  ├─ jobs/                   # Live progress + logs
│  │  │  ├─ network/                # Safe networks, hotspot policy
│  │  │  ├─ logs/                   # Systemd tails
│  │  │  ├─ state/                  # Desired state import/export
│  │  │  └─ settings/               # Auth, updates, snapshots
│  │  ├─ lib/
│  │  │  ├─ api/                    # TS client (generated from OpenAPI)
│  │  │  ├─ components/             # Cards, AutoForm (JSON-schema → form), SudoPrompt, etc.
│  │  │  ├─ stores/                 # jobs store, toasts
│  │  │  └─ i18n/                   # de/en bundles
│  │  ├─ params/
│  │  └─ hooks.client.ts
│  ├─ static/
│  │  ├─ icon.png
│  │  └─ offline/                   # Assets for marina-offline use
│  ├─ svelte.config.js
│  ├─ vite.config.ts
│  └─ tailwind.config.js

# ──────────────────────────────────────────────
# Runtime data on the device (NOT in container images)
# ──────────────────────────────────────────────
└─ var/                              # Mounted at /opt/nautipi/var on the Pi
   ├─ state.sqlite                   # Desired state + jobs + events
   ├─ descriptors/                   # Merged built-ins + imported plugins
   ├─ snapshots/                     # Config snapshot export/import
   ├─ support-bundles/               # Archives from support-bundle script
   └─ logs/                          # Service + job logs
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
