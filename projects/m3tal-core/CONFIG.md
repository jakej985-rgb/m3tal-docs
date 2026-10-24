# M3TAL Environment Variables Reference

All M3TAL configuration is managed through environment variables, which are read from `/etc/m3tal/.env` by both the CLI and all Docker Compose stacks via `--env-file`. This file is best managed using the `m3tal config wizard` command.

## Quick Reference Table

| Variable Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| **Core** | | | | |
| `API_PORT` | The port the M3TAL API daemon listens on. | `5050` | `5050` | API daemon, Dashboard |
| `DASHBOARD_PORT` | The internal port the M3TAL Dashboard container runs on. | `8082` | `8082` | Dashboard container |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed: `local` (direct port) or `traefik` (via Traefik). | `local` | `traefik` | Dashboard container, Traefik |
| `HTTP_PORT` | The port that the Go API daemon uses internally. | `8080` | `8080` | API daemon |
| `STATE_DIR` | The directory where M3TAL stores its state database and configuration files. | `./state` | `/var/lib/m3tal/state` | API daemon |
| `LOG_LEVEL` | The minimum severity level for logs to be displayed. | `info` | `debug` | All components |
| `PUID` | The user ID to run containers as. | `1000` | `1000` | Dashboard container |
| `PGID` | The group ID to run containers as. | `1000` | `1000` | Dashboard container |
| `TZ` | The timezone to use for logging and container timestamps. | `America/Denver` | `UTC` | Dashboard container |
| `DEBUG_MODE` | Enables or disables debug logging and features. | `false` | `true` | All components |
| `METRICS_ENABLED` | Enables or disables the collection and reporting of system metrics. | `true` | `false` | API daemon |
| **Authentication** | | | | |
| `DASHBOARD_SECRET` | A secret key used for signing session cookies in the dashboard. Auto-generated on first `m3tal init`. | `change_me_immediately` | `a_very_secret_and_long_key_generated_by_m3tal` | Dashboard container |
| `API_TOKEN` | An API token for authenticating with the M3TAL API. Auto-generated on first `m3tal init`. | `change_me_api_token` | `a_secure_api_token_string` | API daemon |
| `ADMIN_PASSWORD` | The password for the default `admin` user. | `admin_pass` | `a_strong_password_for_admin` | Dashboard container |
| **Network** | | | | |
| `LOCAL_IP` | The local IP address that M3TAL services should bind to. | `127.0.0.1` | `192.168.1.100` | API daemon |
| `NETWORK_NAME` | The name of the Docker network M3TAL services will join. | `m3tal` | `m3tal_network` | All components |
| **Storage** | | | | |
| `BASE_STORAGE_PATH` | The root directory for all M3TAL data storage. Defaults to `/mnt` in production. | `./data` | `/mnt` | All components |
| `MEDIA_PATH` | The subdirectory within `BASE_STORAGE_PATH` for media files. | `./data/media` | `/mnt/media` | All components |
| `CONFIG_PATH` | The subdirectory within `BASE_STORAGE_PATH` for configuration files. | `./data/config` | `/mnt/config` | All components |
| `DOWNLOADS_PATH` | The subdirectory within `BASE_STORAGE_PATH` for downloaded files. | `./data/downloads` | `/mnt/downloads` | All components |
| **Traefik** | | | | |
| `DOMAIN` | The primary domain name for accessing M3TAL services via Traefik. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. | `localhost` | `m3tal.example.com` | Traefik, API daemon |
| `TRAEFIK_WEB_PORT` | The host port Traefik listens on for HTTP traffic. | `80` | `80` | Traefik container |
| `TRAEFIK_WEBHTTPS_PORT` | The host port Traefik listens on for HTTPS traffic. | `443` | `443` | Traefik container |
| `TRAEFIK_DASHBOARD_PORT` | The internal port Traefik's own dashboard is exposed on. | `8080` | `8080` | Traefik container |
| **VPN** | | | | |
| `VPN_USER` | The username for VPN authentication. | `user` | `myvpnuser` | VPN client |
| `VPN_PASSWORD` | The password for VPN authentication. | `password` | `mysecretvpnpassword` | VPN client |

---

## Environment Variables Reference

All M3TAL configuration is managed through environment variables, which are read from `/etc/m3tal/.env` by both the CLI and all Docker Compose stacks via `--env-file`. This file is best managed using the `m3tal config wizard` command.

### Core

These variables control fundamental aspects of the M3TAL system's operation.

*   **`API_PORT`**
    *   **Description:** The port the M3TAL API daemon listens on for incoming requests.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** API daemon, Dashboard container (for `GO_API_URL`)

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port the M3TAL Dashboard container runs on.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** Dashboard container

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the dashboard is exposed to the network.
        *   `local`: Exposes the dashboard via a direct port binding (`DASHBOARD_PORT`). Access via `http://HOST_IP:DASHBOARD_PORT`. Best for LAN-only or initial setups.
        *   `traefik`: Routes traffic to the dashboard via Traefik. Access via `http://dash.DOMAIN`. Requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** Dashboard container (via compose overrides), Traefik (via labels)

*   **`HTTP_PORT`**
    *   **Description:** The port that the Go API daemon uses internally for its HTTP server.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** API daemon

*   **`STATE_DIR`**
    *   **Description:** The directory where M3TAL stores its state database (`state.db`) and other persistent configuration files.
    *   **Default Value:** `./state` (relative to the compose file context)
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** API daemon

*   **`LOG_LEVEL`**
    *   **Description:** Sets the minimum severity level for logs to be displayed or written. Valid levels typically include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** All components

*   **`PUID`**
    *   **Description:** The user ID (UID) that the M3TAL containers should run as. This is important for file permissions when interacting with host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Dashboard container

*   **`PGID`**
    *   **Description:** The group ID (GID) that the M3TAL containers should run as. This is important for file permissions when interacting with host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Dashboard container

*   **`TZ`**
    *   **Description:** The timezone to use for logging and container timestamps. This helps ensure accurate timekeeping across the system.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `UTC`
    *   **Used By:** Dashboard container

*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug logging and potentially other development-focused features within M3TAL components.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** All components

*   **`METRICS_ENABLED`**
    *   **Description:** Enables or disables the collection and reporting of system and service metrics.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** API daemon

### Authentication

These variables are crucial for securing your M3TAL installation.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used for signing session cookies in the M3TAL Dashboard. **This is auto-generated on the first `m3tal init`**. Users should generally NOT set this manually unless rotating it.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_very_secret_and_long_key_generated_by_m3tal`
    *   **Used By:** Dashboard container

*   **`API_TOKEN`**
    *   **Description:** An API token used for authenticating with the M3TAL API. **This is auto-generated on the first `m3tal init`**. Users should generally NOT set this manually unless rotating it.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a_secure_api_token_string`
    *   **Used By:** API daemon

*   **`ADMIN_PASSWORD`**
    *   **Description:** The password for the default `admin` user account, used for initial dashboard login.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `a_strong_password_for_admin`
    *   **Used By:** Dashboard container

### Network

These variables configure network-related settings for M3TAL.

*   **`LOCAL_IP`**
    *   **Description:** The local IP address that M3TAL services should bind to. This is often `127.0.0.1` for internal communication or a specific host IP if required.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** API daemon

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will join. This network is used for inter-container communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_network`
    *   **Used By:** All components

### Storage

These variables define the locations for M3TAL's data and configuration files on the host system.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory where all M3TAL data is stored. This includes configuration, media, downloads, and the state database. **In production deployments, this defaults to `/mnt`, not `./data` as in template files.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt`
    *   **Used By:** All components (for volume mounts)

*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/media`
    *   **Used By:** All components (for volume mounts)

*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL stores its configuration files.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Used By:** All components (for volume mounts)

*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/downloads`
    *   **Used By:** All components (for volume mounts)

### Traefik

These variables are specifically for configuring the Traefik reverse proxy.

*   **`DOMAIN`**
    *   **Description:** The primary domain name for accessing M3TAL services via Traefik. Setting this variable enables routing rules for services like `dash.DOMAIN` and `api.DOMAIN`. If set, Traefik will be configured to route traffic to the dashboard and API based on this domain.
    *   **Default Value:** `localhost`
    *   **Example Value:** `m3tal.example.com`
    *   **Used By:** Traefik (for router rules), API daemon (for `GO_API_URL` when Traefik is used)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik listens on for incoming HTTP (port 80) traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik listens on for incoming HTTPS (port 443) traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port that Traefik's own web dashboard is exposed on. This is typically accessed on `127.0.0.1:TRAEFIK_DASHBOARD_PORT`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik container

### VPN

These variables are used if you configure M3TAL to use a VPN connection.

*   **`VPN_USER`**
    *   **Description:** The username required to authenticate with the VPN service.
    *   **Default Value:** `user`
    *   **Example Value:** `myvpnuser`
    *   **Used By:** VPN client container

*   **`VPN_PASSWORD`**
    *   **Description:** The password required to authenticate with the VPN service.
    *   **Default Value:** `password`
    *   **Example Value:** `mysecretvpnpassword`
    *   **Used By:** VPN client container