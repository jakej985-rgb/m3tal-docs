# M3TAL System Core

M3TAL is a modular platform built for containerized service hosting, secure routing, and automated infrastructure management.

At the center of M3TAL is `m3tal-core` (the main system engine), which orchestrates multiple Docker Compose stacks, dynamically configures Traefik reverse proxies, handles state database updates via SQLite, and exposes local APIs for the M3TAL Dashboard.

---

## Technical Highlights
* **CLI Engine**: A Go-based command line interface that wraps Docker Compose operations.
* **REST API Daemon**: A systemd Go service listening locally on port `8080` to bridge GUI actions and Docker management.
* **Dashboard Console**: A lightweight web interface to monitor container states, configure network parameters, and secure database backups.
* **Automatic Reverse Proxy**: Out-of-the-box Traefik integration with automated label generation.

---

## Filesystem Structure
The M3TAL system coordinates configuration, database stores, and Compose definitions across these paths:

| Path | Purpose |
| :--- | :--- |
| `/etc/m3tal/.env` | Global environment configuration file. |
| `/var/lib/m3tal/state.db` | SQLite database for maintaining service route history and plugin metadata. |
| `/opt/m3tal/stack/` | Canonical directory containing Docker Compose templates and Traefik files. |
| `/docker` | Symbolic link mapping back to `/opt/m3tal/stack/`. User stacks go here. |
| `/docker/users.json` | Encrypted credential database for the dashboard. |

---

## Getting Started
To proceed with setting up the M3TAL engine:
* Go to the [Installation Guide](#/m3tal-core/INSTALL) to add the package repository and download `m3tal`.
* See the [Configuration Reference](#/m3tal-core/CONFIG) to explore all available parameters.
