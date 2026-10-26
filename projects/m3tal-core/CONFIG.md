## M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables control the behavior of the CLI, the API daemon, Docker Compose stacks, and individual containers.

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. This file is automatically loaded by the M3TAL CLI and is also passed to all Docker Compose stacks via the `--env-file` option, ensuring consistent configuration across the entire system. You can manage these variables using the `m3tal config wizard` or `m3tal config set KEY value` commands.

---

### Quick Reference Table

| Name                  | Default Value           | Description                                                                                                   | Component(s)                              |
| :-------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------ | :---------------------------------------- |
| `API_PORT`            | `5050`                  | Port for internal API daemon communication (e.g., to Dashboard).                                              | `m3tal-dashboard`                         |
| `DASHBOARD_PORT`      | `8082`                  | Internal port for the M3TAL Dashboard container.                                                              | `m3tal-dashboard`                         |
| `DASHBOARD_EXPOSE_MODE` | `local`                 | How the Dashboard is exposed: `local` (port 8082) or `traefik` (via `dash.DOMAIN`).                         | `m3tal-dashboard`                         |
| `HTTP_PORT`           | `8080`                  | Port the M3TAL API daemon listens on the host.                                                                | `m3tal-api.service`, `traefik`            |
| `STATE_DIR`           | `./state`               | Path to the API's SQLite state database directory.                                                            | `m3tal-api.service`, `m3tal-dashboard`    |
| `LOG_LEVEL`           | `info`                  | Logging verbosity for M3TAL components.                                                                       | `CLI binary`, `m3tal-api.service`         |
| `DASHBOARD_SECRET`    | `change_me_immediately` | Secret key for dashboard sessions.                                                                            | `m3tal-dashboard`                         |
| `API_TOKEN`           | `change_me_api_token`   | Authentication token for CLI/Dashboard to API daemon.                                                         | `CLI binary`, `m3tal-api.service`, `m3tal-dashboard` |
| `ADMIN_PASSWORD`      | `admin_pass`            | Initial password for the dashboard admin user.                                                                | `m3tal-dashboard`                         |
| `NETWORK_NAME`        | `m3tal`                 | Name of the Docker network for inter-container communication.                                                 | All Docker Compose stacks                 |
| `LOCAL_IP`            | `127.0.0.1`             | The host machine's IP address.                                                                                | General (network configs)                 |
| `DOMAIN`              | `localhost`             | Base domain for Traefik routing (`dash.DOMAIN`, `api.DOMAIN`).                                                | `traefik`                                 |
| `VPN_USER`            | `user`                  | Username for the optional VPN client (e.g., Cloudflared).                                                     | `cloudflared`                             |
| `VPN_PASSWORD`        | `password`              | Password for the optional VPN client (e.g., Cloudflared).                                                     | `cloudflared`                             |
| `BASE_STORAGE_PATH`   | `./data`                | Base directory for all persistent media and configuration data.                                               | All Docker Compose stacks                 |
| `MEDIA_PATH`          | `./data/media`          | Path to the directory for media data.                                                                         | User-defined stacks                       |
| `CONFIG_PATH`         | `./data/config`         | Path to the directory for application configuration data.                                                     | `m3tal-dashboard`, User-defined stacks    |
| `DOWNLOADS_PATH`      | `./data/downloads`      | Path to the directory for downloaded content.                                                                 | User-defined stacks                       |
| `PUID`                | `1000`                  | Linux User ID (UID) for container processes.                                                                  | All Docker containers                     |
| `PGID`                | `1000`                  | Linux Group ID (GID) for container processes.                                                                 | All Docker containers                     |
| `TZ`                  | `America/Denver`        | Timezone for containers.                                                                                      | All Docker containers                     |
| `TRAEFIK_WEB_PORT`    | `80`                    | Host port Traefik binds for HTTP traffic.                                                                     | `traefik`                                 |
| `TRAEFIK_WEBHTTPS_PORT` | `443`                   | Host port Traefik binds for HTTPS traffic.                                                                    | `traefik`                                 |
| `TRAEFIK_DASHBOARD_PORT` | `8080`                  | Internal port Traefik's API/dashboard listens on (mapped to `127.0.0.1:8081`).                              | `traefik`                                 |
| `DEBUG_MODE`          | `false`                 | Enables debug mode for M3TAL components.                                                                      | `CLI binary`, `m3tal-api.service`, `m3tal-dashboard` |
| `METRICS_ENABLED`     | `true`                  | Controls whether M3TAL components collect and expose metrics.                                                 | `m3tal-api.service`, `m3tal-dashboard`    |

---

### Detailed Variable Reference

#### Core Variables

These variables define fundamental aspects of the M3TAL system's operation.

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon listens for internal Docker network communication (e.g., from the Dashboard).
    *   **Default Value:** `5050`
    *   **Example Value:** `5051`
    *   **Used By:** `m3tal-dashboard` (connects to `http://host.docker.internal:${API_PORT}`)

*   **`HTTP_PORT`**
    *   **Description:** The port on which the M3TAL API daemon (systemd service) listens on the host machine. This is the port Traefik routes to for `api.${DOMAIN}`.
    *   **Default Value:** `8080`
    *   **Example Value:** `9090`
    *   **Used By:** `m3tal-api.service`, `traefik` (routes to `http://host.docker.internal:${HTTP_PORT}`)

*   **`STATE_DIR`**
    *   **Description:** Path to the directory where the M3TAL API stores its SQLite state database (`state.db`). This path is mounted into the `m3tal-dashboard` container for accessing state.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state` (Production deployments typically use `/var/lib/m3tal/state`)
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`LOG_LEVEL`**
    *   **Description:** Sets the logging verbosity for the M3TAL API daemon and CLI. Valid options typically include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `CLI binary`, `m3tal-api.service`

#### Authentication Variables

These variables are crucial for securing access to the M3TAL API and Dashboard.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A unique, cryptographically strong secret key used by the M3TAL Dashboard for session management, cookie signing, and other security-sensitive operations.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a-very-long-random-string-generated-by-m3tal-init`
    *   **Used By:** `m3tal-dashboard`
    *   **Note:** This variable is **auto-generated on first `m3tal init`** to a secure random value. Users should **NOT** set this manually unless performing a secret rotation.

*   **`API_TOKEN`**
    *   **Description:** An authentication token used by the M3TAL CLI and Dashboard to authenticate requests with the M3TAL API daemon.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `another-secure-random-token-for-api-auth`
    *   **Used By:** `CLI binary`, `m3tal-api.service`, `m3tal-dashboard`
    *   **Note:** This variable is **auto-generated on first `m3tal init`** to a secure random value. Users should **NOT** set this manually unless performing a token rotation.

*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial password for the default admin user of the M3TAL Dashboard. After initial setup, it's recommended to manage dashboard users and passwords using the `m3tal dashpass` command.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `mySuperSecurePassword123`
    *   **Used By:** `m3tal-dashboard` (for creating `/docker/users.json` on first run if it doesn't exist)

#### Networking Variables

These variables define how M3TAL components communicate and are exposed.

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL Dashboard container listens. This port is directly bound to the host in `local` expose mode.
    *   **Default Value:** `8082`
    *   **Example Value:** `9000`
    *   **Used By:** `m3tal-dashboard` (container listener, host port binding)

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible.
        *   `local` (default): The dashboard is directly exposed on `HOST_IP:${DASHBOARD_PORT}` (e.g., `http://localhost:8082`). No Traefik required.
        *   `traefik`: The dashboard is exposed via Traefik at `http://dash.${DOMAIN}`. Traefik must be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** `m3tal dash up` command (selects `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` override).

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network used by M3TAL to allow containers to communicate with each other, especially critical for Traefik to discover services.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_prod_network`
    *   **Used By:** All Docker Compose stacks (e.g., `routing-compose.yml`, `m3tal-compose.yml`)

*   **`LOCAL_IP`**
    *   **Description:** The IP address of the host machine. While `host.docker.internal` often suffices for inter-service communication, this can be useful in specific network configurations.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** General (can be referenced in custom network configurations)

#### Storage & Paths Variables

These variables define the filesystem locations for persistent data.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host filesystem where M3TAL stores all persistent media, configuration, and other application data for its services.
    *   **Default Value:** `./data` (relative to `/docker` for development/testing)
    *   **Example Value:** `/mnt/m3tal-data`
    *   **Used By:** All Docker Compose stacks (as the base for volume mounts).
    *   **Note:** In production deployments, this variable typically defaults to `/mnt` (e.g., `/mnt/m3tal-data`), providing a robust location for bulk storage, rather than `./data` which is common for local development.

*   **`MEDIA_PATH`**
    *   **Description:** Path to the directory designated for media files (e.g., movies, TV shows, music). This is usually a subdirectory of `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal-data/media`
    *   **Used By:** User-defined media management stacks (e.g., Plex, Jellyfin, Sonarr, Radarr)

*   **`CONFIG_PATH`**
    *   **Description:** Path to the directory where various application configuration data is stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal-data/config`
    *   **Used By:** `m3tal-dashboard` (for `users.json`), user-defined application stacks

*   **`DOWNLOADS_PATH`**
    *   **Description:** Path to the directory designated for downloaded content before it's processed or moved.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal-data/downloads`
    *   **Used By:** User-defined download client stacks (e.g., qBittorrent, Transmission, Sabnzbd)

#### Traefik Gateway Variables

These variables configure the Traefik reverse proxy for routing external traffic.

*   **`DOMAIN`**
    *   **Description:** The base domain name that Traefik uses to define routing rules. Setting this variable enables access to `dash.${DOMAIN}` for the dashboard and `api.${DOMAIN}` for the M3TAL API. If not set, Traefik defaults to `localhost` for routing rules.
    *   **Default Value:** `localhost`
    *   **Example Value:** `example.com`
    *   **Used By:** `traefik` (via `routing-compose.yml` and `m3tal-compose.traefik.yml` labels, and dynamic configuration files)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host machine port that Traefik binds to for handling incoming HTTP (non-HTTPS) traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `8080` (if port 80 is already in use by another service)
    *   **Used By:** `traefik`

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host machine port that Traefik binds to for handling incoming HTTPS traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `8443`
    *   **Used By:** `traefik`

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own API and dashboard listen within its container. This is mapped to `127.0.0.1:8081` on the host by default.
    *   **Default Value:** `8080` (Traefik's internal default)
    *   **Example Value:** `8000` (Rarely changed, as the host mapping `8081` is usually sufficient)
    *   **Used By:** `traefik`

#### VPN (Cloudflared) Variables

These variables are for configuring the optional Cloudflared tunnel client.

*   **`VPN_USER`**
    *   **Description:** Username for authenticating with the optional VPN client, typically used with services like Cloudflared for secure tunnel creation.
    *   **Default Value:** `user`
    *   **Example Value:** `mycloudflareduser`
    *   **Used By:** `cloudflared` (if configured in `routing-compose.yml`)

*   **`VPN_PASSWORD`**
    *   **Description:** Password for authenticating with the optional VPN client.
    *   **Default Value:** `password`
    *   **Example Value:** `mycloudflaredsecret`
    *   **Used By:** `cloudflared` (if configured in `routing-compose.yml`)

#### System & Runtime Variables

These variables affect the operating environment and behavior of M3TAL components and containers.

*   **`PUID`**
    *   **Description:** The Linux User ID (UID) that container processes should run as. Setting this correctly ensures that containers have appropriate read/write permissions for host-mounted volumes, preventing permission errors.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001` (Often the UID of the first non-root user on a Linux system)
    *   **Used By:** All Docker containers that define a `user` (e.g., `m3tal-dashboard`)

*   **`PGID`**
    *   **Description:** The Linux Group ID (GID) that container processes should run as. Similar to `PUID`, this ensures correct group-based file permissions for host-mounted volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used By:** All Docker containers that define a `user` (e.g., `m3tal-dashboard`)

*   **`TZ`**
    *   **Description:** Sets the timezone inside containers, ensuring that timestamps for logs, file modifications, and scheduling are accurate and consistent with the host system. Uses standard IANA timezone database names.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`, `Asia/Tokyo`
    *   **Used By:** All Docker containers

*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, enables debug logging and potentially other developer-focused features across various M3TAL components. This can increase log verbosity significantly.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `CLI binary`, `m3tal-api.service`, `m3tal-dashboard`

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether M3TAL components collect and expose operational metrics, typically in a format compatible with monitoring systems like Prometheus.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`