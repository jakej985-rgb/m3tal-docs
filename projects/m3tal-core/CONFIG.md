# Environment Variables Reference

All M3TAL environment variables are centrally managed within the `/etc/m3tal/.env` file. This file serves as the single source of truth for configuration, read by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` argument.

To modify these variables, it is recommended to use the `m3tal config wizard` or `m3tal config set <KEY> <VALUE>` commands.

---

## Quick Reference Table

| Name                  | Description                                            | Default Value             | Component(s)                                   |
| :-------------------- | :----------------------------------------------------- | :------------------------ | :--------------------------------------------- |
| `DASHBOARD_PORT`      | Port for the M3TAL Dashboard container                 | `8082`                    | Dashboard, CLI, Traefik                        |
| `DASHBOARD_EXPOSE_MODE` | Dashboard exposure mode (`local` or `traefik`)         | `local`                   | CLI                                            |
| `HTTP_PORT`           | Port for the M3TAL API daemon                          | `8080`                    | API Daemon                                     |
| `STATE_DIR`           | Path to the M3TAL state directory                      | `./state`                 | API Daemon, Dashboard                          |
| `LOG_LEVEL`           | Logging verbosity for M3TAL components                 | `info`                    | API Daemon, CLI                                |
| `DASHBOARD_SECRET`    | Secret key for Dashboard session encryption            | `change_me_immediately`   | Dashboard                                      |
| `API_TOKEN`           | Authentication token for the M3TAL API                 | `change_me_api_token`     | API Daemon, CLI                                |
| `ADMIN_PASSWORD`      | Initial password for the Dashboard admin user          | `admin_pass`              | Dashboard                                      |
| `NETWORK_NAME`        | Name of the Docker network for M3TAL services          | `m3tal`                   | All Compose Stacks                             |
| `LOCAL_IP`            | Local IP address for internal routing                  | `127.0.0.1`               | API Daemon, Traefik                            |
| `DOMAIN`              | Base domain for Traefik routing                        | `localhost`               | Traefik, CLI                                   |
| `VPN_USER`            | Username for VPN services                              | `user`                    | VPN Containers                                 |
| `VPN_PASSWORD`        | Password for VPN services                              | `password`                | VPN Containers                                 |
| `BASE_STORAGE_PATH`   | Root path for all M3TAL persistent data                | `./data`                  | Dashboard, User Containers                     |
| `MEDIA_PATH`          | Subdirectory for media data                            | `./data/media`            | Dashboard, User Containers                     |
| `CONFIG_PATH`         | Subdirectory for configuration files                   | `./data/config`           | Dashboard, User Containers                     |
| `DOWNLOADS_PATH`      | Subdirectory for downloaded content                    | `./data/downloads`        | User Containers                                |
| `PUID`                | User ID for Docker containers (for file permissions)   | `1000`                    | Dashboard, User Containers                     |
| `PGID`                | Group ID for Docker containers (for file permissions)  | `1000`                    | Dashboard, User Containers                     |
| `TZ`                  | Timezone for Docker containers                         | `America/Denver`          | Dashboard, User Containers                     |
| `TRAEFIK_WEB_PORT`    | HTTP entry point port for Traefik                      | `80`                      | Traefik Gateway                                |
| `TRAEFIK_WEBHTTPS_PORT` | HTTPS entry point port for Traefik                     | `443`                     | Traefik Gateway                                |
| `TRAEFIK_DASHBOARD_PORT` | Internal port for Traefik's own dashboard              | `8080`                    | Traefik Gateway                                |
| `DEBUG_MODE`          | Enables debug logging and features                     | `false`                   | API Daemon, CLI, Dashboard                     |
| `METRICS_ENABLED`     | Enables/disables M3TAL API metrics collection          | `true`                    | API Daemon                                     |

---

## Detailed Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system, including API communication, logging, and general user/group IDs for container processes.

*   **`HTTP_PORT`**
    *   **Description:** The network port on which the M3TAL API daemon listens for incoming connections. This port is typically accessed internally by other M3TAL components (like the Dashboard) or via Traefik.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `m3tal-api.service` (Go API daemon)

*   **`STATE_DIR`**
    *   **Description:** Specifies the directory where M3TAL stores its persistent state, including the SQLite database (`state.db`) and other configuration files.
    *   **Default Value:** `./state` (relative to the API daemon's working directory)
    *   **Example Value:** `/var/lib/m3tal/state` (recommended production path)
    *   **Used By:** `m3tal-api.service` (Go API daemon), `m3tal-dashboard` container

*   **`LOG_LEVEL`**
    *   **Description:** Sets the verbosity of logging output for M3TAL components. Options include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `m3tal-api.service` (Go API daemon), CLI binary

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers will use to run their processes. This is crucial for ensuring proper file ownership and permissions for persistent data volumes.
    *   **Default Value:** `1000` (typically the first non-root user on Linux)
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard` container, other user-deployed Docker containers

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers will use to run their processes. Similar to `PUID`, this ensures correct group ownership for data volumes.
    *   **Default Value:** `1000` (typically the primary group for the first non-root user)
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard` container, other user-deployed Docker containers

*   **`TZ`**
    *   **Description:** Sets the timezone for Docker containers. This ensures that logs, timestamps, and scheduled tasks within containers reflect the correct local time.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Used By:** `m3tal-dashboard` container, other user-deployed Docker containers

*   **`DEBUG_MODE`**
    *   **Description:** A boolean flag to enable debug features and additional logging in M3TAL components. Useful for troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `m3tal-api.service` (Go API daemon), CLI binary, `m3tal-dashboard` container

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether the M3TAL API daemon collects and exposes internal metrics (e.g., for Prometheus).
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `m3tal-api.service` (Go API daemon)

### Authentication

These variables manage authentication and authorization for the M3TAL Dashboard and API.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A strong, randomly generated secret key used by the M3TAL Dashboard for session management and encryption. **Users should NOT set this manually unless performing a secret rotation.** It is automatically generated upon the first `m3tal init`.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_very_long_and_random_string_of_characters_1234567890`
    *   **Used By:** `m3tal-dashboard` container

*   **`API_TOKEN`**
    *   **Description:** An authentication token used to secure communication with the M3TAL API. **Users should NOT set this manually unless performing a token rotation.** It is automatically generated upon the first `m3tal init`.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `another_super_secure_api_key_xyz_98765`
    *   **Used By:** `m3tal-api.service` (Go API daemon), CLI binary

*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial password for the default 'admin' user of the M3TAL Dashboard. This is used when the `users.json` file is first created.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MyStrongAndSecureDashboardPassword123!`
    *   **Used By:** `m3tal-dashboard` container (for initial user setup)

### Network Configuration

Variables related to how M3TAL components communicate over Docker networks and host interfaces.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the custom Docker network that M3TAL creates and uses for all its services. This provides isolated communication between containers.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal`
    *   **Used By:** All M3TAL Docker Compose stacks

*   **`LOCAL_IP`**
    *   **Description:** Specifies the local IP address of the host machine. This is used by Traefik for internal routing to host-bound services (like the M3TAL API daemon) via `host.docker.internal`.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100` (if your host has a specific static IP)
    *   **Used By:** `m3tal-api.service` (for listening), Traefik gateway

### Storage Paths

These variables define the base and sub-directories for M3TAL's persistent data storage.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host filesystem where all M3TAL-related persistent data (media, config, downloads, etc.) will be stored. **In production deployments, this defaults to `/mnt`, not `./data` as seen in development templates.**
    *   **Default Value:** `./data` (for development/local testing)
    *   **Example Value:** `/mnt/m3tal_data` (for production systems)
    *   **Used By:** `m3tal-dashboard` container, all other user-deployed containers that require persistent storage.

*   **`MEDIA_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` designated for media files (e.g., movies, TV shows, music).
    *   **Default Value:** `./data/media`
    *   **Example Value:** `${BASE_STORAGE_PATH}/media` (e.g., `/mnt/m3tal_data/media`)
    *   **Used By:** `m3tal-dashboard` container, media server containers (e.g., Jellyfin, Plex)

*   **`CONFIG_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` for storing configuration files of various services.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `${BASE_STORAGE_PATH}/config` (e.g., `/mnt/m3tal_data/config`)
    *   **Used By:** `m3tal-dashboard` container, other user-deployed containers

*   **`DOWNLOADS_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` for storing downloaded content.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `${BASE_STORAGE_PATH}/downloads` (e.g., `/mnt/m3tal_data/downloads`)
    *   **Used By:** Download client containers (e.g., qBittorrent, Transmission)

### Traefik Gateway

Variables specific to the Traefik reverse proxy, controlling how external traffic is routed to M3TAL services.

*   **`DASHBOARD_PORT`**
    *   **Description:** The network port on which the M3TAL Dashboard container listens. This is used for direct port mapping in `local` exposure mode or for Traefik to route to in `traefik` exposure mode.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** `m3tal-dashboard` container, CLI (`m3tal dash up`), Traefik gateway (when `DASHBOARD_EXPOSE_MODE=traefik`)

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Determines how the M3TAL Dashboard is exposed.
        *   `local`: The dashboard is directly exposed on `http://HOST_IP:${DASHBOARD_PORT}`. No Traefik required. Best for LAN-only or initial setup.
        *   `traefik`: The dashboard is exposed via Traefik at `http://dash.${DOMAIN}`. Requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** CLI (`m3tal dash up`) to select the appropriate Docker Compose override file.

*   **`DOMAIN`**
    *   **Description:** The base domain name for M3TAL services. **Setting this enables Traefik routing rules like `dash.DOMAIN` and `api.DOMAIN`.** If not set, Traefik will default to `localhost` for its rules.
    *   **Default Value:** `localhost`
    *   **Example Value:** `mycloud.com`
    *   **Used By:** Traefik gateway for generating routing rules, CLI when configuring dashboard Traefik exposure.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The port on the host machine that Traefik listens on for HTTP (non-secure) traffic. This is Traefik's primary HTTP entry point.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik gateway container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The port on the host machine that Traefik listens on for HTTPS (secure) traffic. This is Traefik's primary HTTPS entry point.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik gateway container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port within the Traefik container where its own dashboard UI is exposed. This is typically mapped to `127.0.0.1:8081` on the host for local access only.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik gateway container (internal mapping)

### VPN Services

Variables for configuring Virtual Private Network (VPN) client containers within the M3TAL ecosystem.

*   **`VPN_USER`**
    *   **Description:** The username required for authenticating with VPN services when deploying VPN client containers.
    *   **Default Value:** `user`
    *   **Example Value:** `john_doe`
    *   **Used By:** VPN client containers (e.g., Wireguard, OpenVPN)

*   **`VPN_PASSWORD`**
    *   **Description:** The password required for authenticating with VPN services when deploying VPN client containers.
    *   **Default Value:** `password`
    *   **Example Value:** `Sup3rS3cur3VpNP@ssw0rd`
    *   **Used By:** VPN client containers (e.g., Wireguard, OpenVPN)