# M3TAL Environment Variables Reference

All M3TAL environment variables are read from `/etc/m3tal/.env` by both the CLI and all Docker Compose stacks via `--env-file`. This file is managed by the `m3tal config wizard` and can be updated using `m3tal config set KEY value`.

## Quick Reference

| Variable Name             | Default Value      | Description                                                                                                                                                                                                                               |
|---------------------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Core**                  |                    |                                                                                                                                                                                                                                           |
| `HTTP_PORT`               | `8080`             | The port the M3TAL API daemon listens on.                                                                                                                                                                                                 |
| `LOG_LEVEL`               | `info`             | Sets the logging verbosity for M3TAL services. Can be `debug`, `info`, `warn`, `error`.                                                                                                                                                |
| `TZ`                      | `America/Denver`   | The timezone to use for M3TAL services.                                                                                                                                                                                                   |
| `PUID`                    | `1000`             | The user ID to run Docker containers as.                                                                                                                                                                                                  |
| `PGID`                    | `1000`             | The group ID to run Docker containers as.                                                                                                                                                                                                 |
| `DEBUG_MODE`              | `false`            | Enables debug mode for M3TAL services.                                                                                                                                                                                                    |
| **Auth**                  |                    |                                                                                                                                                                                                                                           |
| `DASHBOARD_SECRET`        | `change_me_immediately` | A secret key for securing the dashboard session. **Auto-generated on first `m3tal init`. Do NOT set manually unless rotating.**                                                                                                        |
| `API_TOKEN`               | `change_me_api_token` | An API token for authenticating with the M3TAL API. **Auto-generated on first `m3tal init`. Do NOT set manually unless rotating.**                                                                                                      |
| `ADMIN_PASSWORD`          | `admin_pass`       | The default password for the admin user of the dashboard.                                                                                                                                                                                 |
| **Network**               |                    |                                                                                                                                                                                                                                           |
| `LOCAL_IP`                | `127.0.0.1`        | The local IP address used for internal service communication.                                                                                                                                                                             |
| `NETWORK_NAME`            | `m3tal`            | The name of the Docker network M3TAL services will connect to.                                                                                                                                                                              |
| **Storage**               |                    |                                                                                                                                                                                                                                           |
| `BASE_STORAGE_PATH`       | `./data`           | **Controls where media data is stored.** Defaults to `/mnt` in production deployments.                                                                                                                                                    |
| `MEDIA_PATH`              | `./data/media`     | The subdirectory within `BASE_STORAGE_PATH` for storing media files.                                                                                                                                                                      |
| `CONFIG_PATH`             | `./data/config`    | The subdirectory within `BASE_STORAGE_PATH` for storing configuration files.                                                                                                                                                            |
| `DOWNLOADS_PATH`          | `./data/downloads` | The subdirectory within `BASE_STORAGE_PATH` for storing downloaded files.                                                                                                                                                                 |
| `STATE_DIR`               | `./state`          | The directory where the M3TAL API daemon stores its state database (`state.db`). This is distinct from `BASE_STORAGE_PATH`.                                                                                                             |
| **Traefik**               |                    |                                                                                                                                                                                                                                           |
| `DOMAIN`                  | `localhost`        | The domain name used for Traefik routing rules. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes.                                                                                                                             |
| `DASHBOARD_PORT`          | `8082`             | The internal port the M3TAL dashboard listens on.                                                                                                                                                                                         |
| `DASHBOARD_EXPOSE_MODE`   | `local`            | Controls how the dashboard is exposed: `local` (direct port binding) or `traefik` (via Traefik reverse proxy).                                                                                                                            |
| `TRAEFIK_WEB_PORT`        | `80`               | The host port Traefik listens on for HTTP traffic.                                                                                                                                                                                        |
| `TRAEFIK_WEBHTTPS_PORT`   | `443`              | The host port Traefik listens on for HTTPS traffic.                                                                                                                                                                                       |
| `TRAEFIK_DASHBOARD_PORT`  | `8080`             | The internal port Traefik's own dashboard listens on. Accessible via `127.0.0.1:TRAEFIK_DASHBOARD_PORT`.                                                                                                                                   |
| **VPN**                   |                    |                                                                                                                                                                                                                                           |
| `VPN_USER`                | `user`             | Username for the VPN connection.                                                                                                                                                                                                          |
| `VPN_PASSWORD`            | `password`         | Password for the VPN connection.                                                                                                                                                                                                          |
| **System**                |                    |                                                                                                                                                                                                                                           |
| `METRICS_ENABLED`         | `true`             | Enables the Prometheus metrics endpoint for M3TAL services.                                                                                                                                                                               |

---

## Detailed Variable Descriptions

### Core

*   **`HTTP_PORT`**
    *   **Description:** The port on which the M3TAL API daemon (Go binary) listens for incoming HTTP requests.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component(s) Using:** `m3tal-api.service`

*   **`LOG_LEVEL`**
    *   **Description:** Controls the verbosity of logs generated by M3TAL services. Supported levels include `debug`, `info`, `warn`, and `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Component(s) Using:** `m3tal-api.service`, `m3tal-dashboard`

*   **`TZ`**
    *   **Description:** Specifies the timezone used by M3TAL services for consistent timestamping and date-related operations.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `UTC`
    *   **Component(s) Using:** `m3tal-dashboard`, `m3tal-api.service` (via Docker image environment)

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers will run as. This is important for file permissions within the containers and on the host.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component(s) Using:** `m3tal-dashboard`, `ollama`, `traefik` (via Docker Compose `user` directive)

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers will run as. This complements `PUID` for managing file permissions.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component(s) Using:** `m3tal-dashboard`, `ollama`, `traefik` (via Docker Compose `user` directive)

*   **`DEBUG_MODE`**
    *   **Description:** A boolean flag to enable or disable debug logging and features across M3TAL services.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Component(s) Using:** `m3tal-api.service`, `m3tal-dashboard`

### Auth

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used for securing session cookies and other sensitive operations within the M3TAL dashboard. **This variable is auto-generated on the first `m3tal init` command. Users should NOT set it manually unless they intend to rotate it.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `some_long_random_secret_string`
    *   **Component(s) Using:** `m3tal-dashboard`

*   **`API_TOKEN`**
    *   **Description:** A token used for authenticating requests to the M3TAL API. **This variable is auto-generated on the first `m3tal init` command. Users should NOT set it manually unless they intend to rotate it.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a_secure_api_token_generated_by_m3tal`
    *   **Component(s) Using:** `m3tal-dashboard` (when communicating with API)

*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial password for the administrative user of the M3TAL dashboard. It's highly recommended to change this after initial setup.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `my_new_strong_password`
    *   **Component(s) Using:** `m3tal-dashboard`

### Network

*   **`LOCAL_IP`**
    *   **Description:** The IP address that M3TAL services will use for internal communication, particularly when referring to `host.docker.internal` or other service discovery mechanisms.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Component(s) Using:** `m3tal-api.service` (via `GO_API_URL` in dashboard compose)

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will join. This ensures they can communicate with each other.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_net`
    *   **Component(s) Using:** `m3tal-dashboard`, `traefik`, `ollama` (via Docker Compose network configuration)

### Storage

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host machine where M3TAL will store its persistent data, including configuration, media, and downloads. **In production deployments, this defaults to `/mnt`. In template/development setups, it might default to `./data`.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt` (production), `./data` (development)
    *   **Component(s) Using:** `m3tal-dashboard` (volumes), `m3tal-api.service` (via Docker Compose volumes)

*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files (e.g., images, videos) are stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `./data/media`
    *   **Component(s) Using:** `m3tal-dashboard` (volumes)

*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL configuration files are stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `./data/config`
    *   **Component(s) Using:** `m3tal-dashboard` (volumes)

*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are placed.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `./data/downloads`
    *   **Component(s) Using:** `m3tal-dashboard` (volumes)

*   **`STATE_DIR`**
    *   **Description:** The directory where the M3TAL API daemon stores its SQLite state database (`state.db`). This is separate from `BASE_STORAGE_PATH`.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Component(s) Using:** `m3tal-api.service` (via Docker Compose `volumes` and Go binary environment)

### Traefik

*   **`DOMAIN`**
    *   **Description:** The primary domain name configured for your M3TAL instance. Setting this variable enables Traefik to route traffic to services using subdomains like `dash.DOMAIN` and `api.DOMAIN`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `my.m3tal.com`
    *   **Component(s) Using:** `traefik` (dynamic configuration), `m3tal-dashboard` (Traefik labels)

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL dashboard container listens.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Component(s) Using:** `m3tal-dashboard` (compose file), `traefik` (compose file)

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Determines how the M3TAL dashboard is made accessible.
        *   `local`: The dashboard is exposed directly via a port binding (`DASHBOARD_PORT`). Accessible via `http://<HOST_IP>:<DASHBOARD_PORT>`. Ideal for LAN-only setups.
        *   `traefik`: The dashboard is exposed through the Traefik reverse proxy. Accessible via `http://dash.<DOMAIN>`. Requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Component(s) Using:** `m3tal-dashboard` (compose override selection)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik will bind to for incoming HTTP (port 80) traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Component(s) Using:** `traefik` (compose file)

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik will bind to for incoming HTTPS (port 443) traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Component(s) Using:** `traefik` (compose file)

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik's own web UI dashboard listens on. This is typically accessed directly on the host at `127.0.0.1:TRAEFIK_DASHBOARD_PORT`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8081`
    *   **Component(s) Using:** `traefik` (compose file)

### VPN

*   **`VPN_USER`**
    *   **Description:** The username to use when establishing a VPN connection.
    *   **Default Value:** `user`
    *   **Example Value:** `myvpnuser`
    *   **Component(s) Using:** Assumed to be used by a VPN client component (not explicitly defined in provided JSON but common for network services).

*   **`VPN_PASSWORD`**
    *   **Description:** The password for the VPN connection.
    *   **Default Value:** `password`
    *   **Example Value:** `mysecurevpnpassword`
    *   **Component(s) Using:** Assumed to be used by a VPN client component.

### System

*   **`METRICS_ENABLED`**
    *   **Description:** A boolean flag to enable or disable the Prometheus metrics endpoint for M3TAL services, allowing for monitoring and scraping.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Component(s) Using:** `m3tal-api.service`