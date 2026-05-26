# M3TAL Configuration Reference

This guide documents the variables in `/etc/m3tal/.env` which control how M3TAL acts.

---

## Configuration Location
All configuration parameters are loaded from:
`/etc/m3tal/.env`

Both the CLI, API daemon, and all Docker Compose services share these variables. You can edit this file manually, run `m3tal config wizard`, or configure individual keys using:
```bash
m3tal config set KEY value
```

---

## Core Configurations

| Variable Name | Default | Purpose |
| :--- | :--- | :--- |
| `STATE_DIR` | `./state` | Path to store the SQLite `state.db` database and configs. |
| `LOG_LEVEL` | `info` | Logging verbosity: `debug`, `info`, `warn`, `error`. |
| `TZ` | `America/Denver` | System timezone for logs and metrics. |
| `DEBUG_MODE` | `false` | Enables full debug logging. |

---

## Security & Credentials

| Variable Name | Default | Purpose |
| :--- | :--- | :--- |
| `DASHBOARD_SECRET` | `change_me` | Secret token used to secure web sessions. Auto-generated on first install. |
| `API_TOKEN` | `change_me` | Local token used to authorize the dashboard to query the API service. |
| `ADMIN_PASSWORD` | `admin_pass` | Dashboard user password. |

---

## Network & Host Routing

| Variable Name | Default | Purpose |
| :--- | :--- | :--- |
| `HTTP_PORT` | `8080` | Port the API daemon runs on. |
| `DOMAIN` | `localhost` | Primary domain. Enables routing rules for `dash.DOMAIN` and `api.DOMAIN` when in `traefik` mode. |
| `NETWORK_NAME` | `m3tal` | Docker network for container communication. |
| `TRAEFIK_WEB_PORT`| `80` | External HTTP port bound by Traefik. |
| `TRAEFIK_WEBHTTPS_PORT` | `443` | External HTTPS port bound by Traefik. |

---

## Storage Contracts

| Variable Name | Default | Purpose |
| :--- | :--- | :--- |
| `BASE_STORAGE_PATH` | `./data` | Root path for storage volumes. In production, defaults to `/mnt`. |
| `MEDIA_PATH` | `./data/media` | Target folder for media files. |
| `CONFIG_PATH` | `./data/config` | Target folder for configuration files. |
| `DOWNLOADS_PATH` | `./data/downloads`| Target folder for downloaded assets. |

---

## Expose Modes (`DASHBOARD_EXPOSE_MODE`)
Controls how you view the Dashboard web interface:

### 1. `local` mode (Default)
* Binds the Dashboard directly to your host's network interfaces.
* Accessible at `http://YOUR_SERVER_IP:8082` (uses `DASHBOARD_PORT`).
* Does not need Traefik or a custom domain.

### 2. `traefik` mode
* Attaches labels to route requests through Traefik proxy.
* Accessible at `http://dash.YOUR_DOMAIN`.
* Requires Traefik (`m3tal up`) and DNS setup.
