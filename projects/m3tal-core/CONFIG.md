# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are primarily managed through the `/etc/m3tal/.env` file, which is read by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option.

## Quick Reference

| Variable Name | Description | Default Value |
|---|---|---|
| **Core** | | |
| `HTTP_PORT` | The port the M3TAL API daemon listens on. | `8080` |
| `STATE_DIR` | Directory for storing M3TAL's state database and configuration files. | `./state` |
| `LOG_LEVEL` | Sets the logging verbosity for M3TAL services. | `info` |
| `DEBUG_MODE` | Enables or disables debug mode for M3TAL. | `false` |
| `METRICS_ENABLED` | Enables or disables metrics collection for M3TAL. | `true` |
| **Auth** | | |
| `DASHBOARD_SECRET` | Secret key for the dashboard. **Auto-generated on first `m3tal init`.** | `change_me_immediately` |
| `API_TOKEN` | API token for authenticating with the M3TAL API. **Auto-generated on first `m3tal init`.** | `change_me_api_token` |
| `ADMIN_PASSWORD` | Password for the M3TAL dashboard administrator. | `admin_pass` |
| **Network** | | |
| `API_PORT` | The port exposed by the M3TAL API daemon. | `5050` |
| `DASHBOARD_PORT` | The port the M3TAL dashboard container listens on. | `8082` |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed (e.g., `local` or `traefik`). | `local` |
| `NETWORK_NAME` | The name of the Docker network M3TAL services will join. | `m3tal` |
| `LOCAL_IP` | The IP address of the host machine. | `127.0.0.1` |
| `DOMAIN` | The domain name used for Traefik routing (e.g., `dash.DOMAIN`, `api.DOMAIN`). | `localhost` |
| **Storage** | | |
| `BASE_STORAGE_PATH` | The root directory for M3TAL's data storage. Defaults to `/mnt` in production. | `./data` |
| `MEDIA_PATH` | Path for storing media files within `BASE_STORAGE_PATH`. | `./data/media` |
| `CONFIG_PATH` | Path for storing configuration files within `BASE_STORAGE_PATH`. | `./data/config` |
| `DOWNLOADS_PATH` | Path for storing downloaded files within `BASE_STORAGE_PATH`. | `./data/downloads` |
| `PUID` | The user ID (UID) for running Docker containers. | `1000` |
| `PGID` | The group ID (GID) for running Docker containers. | `1000` |
| **Traefik** | | |
| `TRAEFIK_WEB_PORT` | The port Traefik listens on for HTTP traffic. | `80` |
| `TRAEFIK_WEBHTTPS_PORT` | The port Traefik listens on for HTTPS traffic. | `443` |
| `TRAEFIK_DASHBOARD_PORT` | The internal port Traefik uses for its own dashboard. | `8080` |
| **VPN** | | |
| `VPN_USER` | Username for VPN connection. | `user` |
| `VPN_PASSWORD` | Password for VPN connection. | `password` |
| **System** | | |
| `TZ` | The timezone to use for M3TAL services. | `America/Denver` |

---

## Detailed Environment Variable Reference

All environment variables are read from `/etc/m3tal/.env` by both the M3TAL CLI and all Docker Compose stacks via `--env-file`.

### Core

These variables configure fundamental aspects of the M3TAL system.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `HTTP_PORT` | The port the M3TAL API daemon listens on. This is the internal port the API uses. | `8080` | `8080` | `m3tal-api.service` |
| `STATE_DIR` | Directory for storing M3TAL's state database and configuration files. This is relative to the `CONFIG_PATH`. | `./state` | `/var/lib/m3tal/state` | `m3tal-api.service`, `m3tal-dashboard` |
| `LOG_LEVEL` | Sets the logging verbosity for M3TAL services. Supported levels include `debug`, `info`, `warn`, `error`. | `info` | `debug` | `m3tal-api.service`, `m3tal-dashboard` |
| `DEBUG_MODE` | Enables or disables debug mode for M3TAL. When enabled, more verbose logging and debugging features may be active. | `false` | `true` | `m3tal-api.service`, `m3tal-dashboard` |
| `METRICS_ENABLED` | Enables or disables metrics collection for M3TAL. This can be used for monitoring and performance analysis. | `true` | `false` | `m3tal-api.service` |

### Auth

These variables are related to authentication and security within M3TAL.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `DASHBOARD_SECRET` | Secret key for the dashboard. **This variable is auto-generated on first `m3tal init` and should NOT be set manually unless rotating.** | `change_me_immediately` | `s3cr3t_k3y_f0r_d4shb04rd` | `m3tal-dashboard` |
| `API_TOKEN` | API token for authenticating with the M3TAL API. **This variable is auto-generated on first `m3tal init` and should NOT be set manually unless rotating.** | `change_me_api_token` | `4p1_t0k3n_f0r_s3cur1ty` | `m3tal-api.service` |
| `ADMIN_PASSWORD` | Password for the M3TAL dashboard administrator. | `admin_pass` | `my_secure_admin_password` | `m3tal-dashboard` |

### Network

These variables control network-related configurations, including ports and domain routing.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `API_PORT` | The port exposed by the M3TAL API daemon. This is the external-facing port for API access. | `5050` | `5050` | `m3tal-api.service` |
| `DASHBOARD_PORT` | The port the M3TAL dashboard container listens on internally. | `8082` | `8082` | `m3tal-dashboard` |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed. Options are: <br>- `local`: Exposes the dashboard directly via a port binding. <br>- `traefik`: Exposes the dashboard via Traefik routing rules (e.g., `dash.DOMAIN`). | `local` | `traefik` | `m3tal-dashboard` |
| `NETWORK_NAME` | The name of the Docker network M3TAL services will join. | `m3tal` | `m3tal-net` | Docker Compose |
| `LOCAL_IP` | The IP address of the host machine. Used for host.docker.internal resolution. | `127.0.0.1` | `192.168.1.100` | `m3tal-dashboard` |
| `DOMAIN` | The domain name used for Traefik routing rules. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. | `localhost` | `m3tal.local` | `traefik`, `m3tal-dashboard` |

### Storage

These variables define the paths and permissions for M3TAL's data storage.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `BASE_STORAGE_PATH` | The root directory for M3TAL's data storage. **Note that this defaults to `/mnt` in production deployments, not `./data` as in the template.** | `./data` | `/mnt` | `m3tal-dashboard`, `m3tal-api.service` |
| `MEDIA_PATH` | Path for storing media files within `BASE_STORAGE_PATH`. | `./data/media` | `/mnt/media` | `m3tal-dashboard` |
| `CONFIG_PATH` | Path for storing configuration files within `BASE_STORAGE_PATH`. | `./data/config` | `/mnt/config` | `m3tal-dashboard` |
| `DOWNLOADS_PATH` | Path for storing downloaded files within `BASE_STORAGE_PATH`. | `./data/downloads` | `/mnt/downloads` | `m3tal-dashboard` |
| `PUID` | The user ID (UID) for running Docker containers. This ensures proper file permissions. | `1000` | `1001` | Docker Compose |
| `PGID` | The group ID (GID) for running Docker containers. This ensures proper file permissions. | `1000` | `1001` | Docker Compose |

### Traefik

These variables configure the Traefik reverse proxy.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `TRAEFIK_WEB_PORT` | The port Traefik listens on for HTTP traffic. | `80` | `80` | `traefik` |
| `TRAEFIK_WEBHTTPS_PORT` | The port Traefik listens on for HTTPS traffic. | `443` | `443` | `traefik` |
| `TRAEFIK_DASHBOARD_PORT` | The internal port Traefik uses for its own dashboard. Accessible via `http://127.0.0.1:8080` by default. | `8080` | `8080` | `traefik` |

### VPN

These variables are used for configuring VPN connections.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `VPN_USER` | Username for VPN connection. | `user` | `vpn_username` | `cloudflared` (potentially) |
| `VPN_PASSWORD` | Password for VPN connection. | `password` | `vpn_password_123` | `cloudflared` (potentially) |

### System

These variables control system-level settings.

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `TZ` | The timezone to use for M3TAL services. This ensures correct time logging and scheduling. | `America/Denver` | `UTC` | `m3tal-dashboard`, `m3tal-api.service` |

---

## File System Contract

| Path | Purpose |
|---|---|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

---

## Dashboard Access Modes

The M3TAL dashboard can be accessed in two modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable:

### Mode 1: `local` (Default)

- **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
- **Mechanism:** Uses `m3tal-compose.local.yml` to add a direct port binding `${DASHBOARD_PORT:-8082}:8082` to the dashboard container.
- **Access:** You can access the dashboard via `http://HOST_IP:8082` or `http://localhost:8082`.
- **Use Case:** Ideal for LAN-only setups, first-time users, or local testing, as it does not require Traefik to be running.

### Mode 2: `traefik`

- **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Mechanism:** Uses `m3tal-compose.traefik.yml` to add Traefik labels to the dashboard container. Traefik then routes traffic from `dash.${DOMAIN}` to the dashboard on port 8082.
- **Access:** Access the dashboard via `http://dash.DOMAIN` (requires Traefik to be running via `m3tal up`).
- **Use Case:** Best for domain-based setups and when running multiple services behind a reverse proxy like Traefik.

---

## Installation

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```