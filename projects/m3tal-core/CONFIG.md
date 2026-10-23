# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are primarily managed through the `/etc/m3tal/.env` file, which is read by both the M3TAL CLI and all Docker Compose stacks.

## Quick Reference

| Variable Name              | Description                                                                                                                               | Default Value      | Example Value                 | Used By                                                                          |
| :------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- | :----------------- | :---------------------------- | :------------------------------------------------------------------------------- |
| **Core**                   |                                                                                                                                           |                    |                               |                                                                                  |
| `API_PORT`                 | Port for the M3TAL API daemon.                                                                                                            | `5050`             | `5050`                        | `m3tal-api.service`, `m3tal-dashboard`                                           |
| `DASHBOARD_PORT`           | Port for the M3TAL Dashboard.                                                                                                             | `8082`             | `8082`                        | `m3tal-dashboard`                                                                |
| `DASHBOARD_EXPOSE_MODE`    | Controls how the dashboard is exposed: `local` (direct port) or `traefik` (via Traefik).                                                  | `local`            | `traefik`                     | `m3tal-dashboard`                                                                |
| `HTTP_PORT`                | General HTTP port, potentially for internal services or specific configurations.                                                          | `8080`             | `8080`                        | `m3tal-api.service` (internal usage)                                             |
| `STATE_DIR`                | Directory where M3TAL stores its state database and configuration.                                                                        | `./state`          | `/var/lib/m3tal/state`        | `m3tal-api.service`, `m3tal-dashboard`                                           |
| `LOG_LEVEL`                | The verbosity of logs. Options typically include `debug`, `info`, `warn`, `error`.                                                      | `info`             | `debug`                       | `m3tal-api.service`, `m3tal-dashboard`                                           |
| `DEBUG_MODE`               | Enables debug mode for enhanced logging and diagnostics.                                                                                  | `false`            | `true`                        | `m3tal-api.service`, `m3tal-dashboard`                                           |
| `METRICS_ENABLED`          | Enables the collection and exposition of system metrics.                                                                                  | `true`             | `false`                       | `m3tal-api.service`                                                              |
| **Auth**                   |                                                                                                                                           |                    |                               |                                                                                  |
| `DASHBOARD_SECRET`         | Secret key for securing the M3TAL Dashboard. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.**                | `change_me_immediately` | `your_generated_secret_here` | `m3tal-dashboard`                                                                |
| `API_TOKEN`                | API authentication token. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.**                                   | `change_me_api_token` | `your_generated_api_token`    | `m3tal-api.service` (for internal auth)                                          |
| `ADMIN_PASSWORD`           | Password for the default administrator account on the dashboard.                                                                          | `admin_pass`       | `your_strong_password`        | `m3tal-dashboard`                                                                |
| **Network**                |                                                                                                                                           |                    |                               |                                                                                  |
| `NETWORK_NAME`             | The name of the Docker network used by M3TAL services.                                                                                    | `m3tal`            | `m3tal-net`                   | All M3TAL Docker services                                                        |
| `LOCAL_IP`                 | The local IP address of the host machine.                                                                                                 | `127.0.0.1`        | `192.168.1.100`               | `m3tal-api.service` (for `host.docker.internal`)                                 |
| `DOMAIN`                   | The primary domain name for M3TAL services. Setting this enables Traefik routing for `dash.DOMAIN` and `api.DOMAIN`.                      | `localhost`        | `m3tal.example.com`           | Traefik, `m3tal-dashboard`                                                       |
| **Storage**                |                                                                                                                                           |                    |                               |                                                                                  |
| `BASE_STORAGE_PATH`        | Root path for M3TAL's persistent storage. Defaults to `/mnt` in production deployments.                                                   | `./data`           | `/mnt`                        | `m3tal-dashboard`, `m3tal-api.service` (for volumes)                             |
| `MEDIA_PATH`               | Path within `BASE_STORAGE_PATH` for storing media files.                                                                                  | `./data/media`     | `/mnt/media`                  | `m3tal-dashboard` (for volumes)                                                  |
| `CONFIG_PATH`              | Path within `BASE_STORAGE_PATH` for storing configuration files.                                                                          | `./data/config`    | `/mnt/config`                 | `m3tal-dashboard` (for volumes)                                                  |
| `DOWNLOADS_PATH`           | Path within `BASE_STORAGE_PATH` for storing downloaded files.                                                                             | `./data/downloads` | `/mnt/downloads`              | `m3tal-dashboard` (for volumes)                                                  |
| **User/Group IDs**         |                                                                                                                                           |                    |                               |                                                                                  |
| `PUID`                     | The user ID to run Docker containers as.                                                                                                  | `1000`             | `1000`                        | `m3tal-dashboard`, `ollama`                                                      |
| `PGID`                     | The group ID to run Docker containers as.                                                                                                 | `1000`             | `1000`                        | `m3tal-dashboard`, `ollama`                                                      |
| **System**                 |                                                                                                                                           |                    |                               |                                                                                  |
| `TZ`                       | Timezone for system services.                                                                                                             | `America/Denver`   | `Europe/London`               | `m3tal-dashboard`, `ollama`                                                      |
| **Traefik**                |                                                                                                                                           |                    |                               |                                                                                  |
| `TRAEFIK_WEB_PORT`         | The host port Traefik listens on for HTTP traffic.                                                                                        | `80`               | `80`                          | `traefik` container                                                              |
| `TRAEFIK_WEBHTTPS_PORT`    | The host port Traefik listens on for HTTPS traffic.                                                                                       | `443`              | `443`                         | `traefik` container                                                              |
| `TRAEFIK_DASHBOARD_PORT`   | The internal port Traefik uses to access its own dashboard.                                                                               | `8080`             | `8080`                        | `traefik` container (internal mapping)                                           |
| **VPN**                    |                                                                                                                                           |                    |                               |                                                                                  |
| `VPN_USER`                 | Username for the VPN connection.                                                                                                          | `user`             | `my_vpn_user`                 | (Not explicitly used in provided JSON, but assumed for VPN integration)          |
| `VPN_PASSWORD`             | Password for the VPN connection.                                                                                                          | `password`         | `my_vpn_password`             | (Not explicitly used in provided JSON, but assumed for VPN integration)          |

---

## Detailed Environment Variable Reference

All environment variables are read from `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` and can be updated using `m3tal config set KEY value`.

### Core

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon (Go binary) listens for incoming requests. This is used for internal communication between services.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`DASHBOARD_PORT`**
    *   **Description:** The port on which the M3TAL Dashboard service runs internally.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** `m3tal-dashboard`

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is exposed to users.
        *   `local`: The dashboard is exposed directly via its `DASHBOARD_PORT` on the host's IP address. No reverse proxy is required. This is ideal for LAN-only access or initial setup.
        *   `traefik`: The dashboard is exposed via the Traefik reverse proxy. Traefik will route traffic based on the `DOMAIN` variable (e.g., `dash.yourdomain.com`).
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** `m3tal-dashboard`

*   **`HTTP_PORT`**
    *   **Description:** A general HTTP port, often used for internal service communication or specific configurations. In the context of the API daemon, it might represent an internal listening port if not explicitly overridden by `API_PORT`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `m3tal-api.service` (internal usage)

*   **`STATE_DIR`**
    *   **Description:** The directory where M3TAL stores its persistent state, including the SQLite database and other configuration files.
    *   **Default Value:** `./state` (relative to the M3TAL installation directory or Docker volume mount)
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`LOG_LEVEL`**
    *   **Description:** Sets the verbosity of logging for M3TAL services. Common values include `debug`, `info`, `warn`, and `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`DEBUG_MODE`**
    *   **Description:** Enables debug mode, which typically provides more detailed logging and diagnostic information for troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether M3TAL collects and exposes system and service metrics.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `m3tal-api.service`

### Auth

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used for securing the M3TAL Dashboard session and communications. **This variable is auto-generated on the first `m3tal init` command. Users should generally NOT set this manually unless they need to rotate the secret.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `your_generated_secret_here`
    *   **Used By:** `m3tal-dashboard`

*   **`API_TOKEN`**
    *   **Description:** An authentication token used to secure API requests, particularly for internal service-to-service communication. **This variable is auto-generated on the first `m3tal init` command. Users should generally NOT set this manually unless they need to rotate the token.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `your_generated_api_token`
    *   **Used By:** `m3tal-api.service` (for internal authentication)

*   **`ADMIN_PASSWORD`**
    *   **Description:** The password for the default administrator account on the M3TAL Dashboard. It's recommended to change this to a strong, unique password.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `your_strong_password`
    *   **Used By:** `m3tal-dashboard`

### Network

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will join. This ensures proper inter-service communication within the Docker environment.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal-net`
    *   **Used By:** All M3TAL Docker services

*   **`LOCAL_IP`**
    *   **Description:** The IP address of the host machine that Docker can use to communicate with containers and vice-versa. This is often used to map `host.docker.internal`.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** `m3tal-api.service` (for `host.docker.internal` mapping)

*   **`DOMAIN`**
    *   **Description:** The primary domain name configured for your M3TAL instance. Setting this variable is crucial for Traefik routing. When set, Traefik will be configured to route traffic to `api.${DOMAIN}` and `dash.${DOMAIN}`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `m3tal.example.com`
    *   **Used By:** Traefik, `m3tal-dashboard`

### Storage

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host machine where M3TAL stores all its persistent data, including media, configuration, and downloads. In production deployments, this typically defaults to `/mnt`.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt`
    *   **Used By:** `m3tal-dashboard`, `m3tal-api.service` (for Docker volume mounts)

*   **`MEDIA_PATH`**
    *   **Description:** The specific sub-path within `BASE_STORAGE_PATH` where media files are stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/media`
    *   **Used By:** `m3tal-dashboard` (for Docker volume mounts)

*   **`CONFIG_PATH`**
    *   **Description:** The specific sub-path within `BASE_STORAGE_PATH` where M3TAL's configuration files are stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Used By:** `m3tal-dashboard` (for Docker volume mounts)

*   **`DOWNLOADS_PATH`**
    *   **Description:** The specific sub-path within `BASE_STORAGE_PATH` where downloaded files are stored.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/downloads`
    *   **Used By:** `m3tal-dashboard` (for Docker volume mounts)

### User/Group IDs

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers should run as. This is important for ensuring correct file permissions when containers access host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard`, `ollama`

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers should run as. This is important for ensuring correct file permissions when containers access host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard`, `ollama`

### System

*   **`TZ`**
    *   **Description:** Specifies the timezone for the system and its services. This ensures that logs and timestamps are accurate.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Used By:** `m3tal-dashboard`, `ollama`

### Traefik

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The port on the host machine that Traefik will listen on for incoming HTTP traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** `traefik` container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The port on the host machine that Traefik will listen on for incoming HTTPS traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** `traefik` container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own dashboard is accessible (typically `8080`). This is usually mapped to `127.0.0.1:8081` on the host for local access.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `traefik` container (internal mapping)

### VPN

*   **`VPN_USER`**
    *   **Description:** The username for establishing a VPN connection. This variable is intended for use with VPN client configurations within the M3TAL environment, though it's not directly referenced in the provided Docker service definitions.
    *   **Default Value:** `user`
    *   **Example Value:** `my_vpn_user`
    *   **Used By:** (Assumed for VPN integration)

*   **`VPN_PASSWORD`**
    *   **Description:** The password for establishing a VPN connection. This variable is intended for use with VPN client configurations within the M3TAL environment, though it's not directly referenced in the provided Docker service definitions.
    *   **Default Value:** `password`
    *   **Example Value:** `my_vpn_password`
    *   **Used By:** (Assumed for VPN integration)