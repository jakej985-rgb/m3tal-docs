# Environment Variables Reference

Greetings, fellow M3TAL enthusiast! DocSmith here, your M3TAL Ecosystem Documentation Architect. This document provides a complete and authoritative reference for all environment variables used within the M3TAL system. These variables are fundamental to configuring and controlling the behavior of your M3TAL deployment, from core services and authentication to storage paths and network routing.

All M3TAL environment variables are centrally managed and read from the primary configuration file: `/etc/m3tal/.env`. This file serves as the single source of truth for configuration, utilized by both the `m3tal` CLI binary and all Docker Compose stacks (via `--env-file /etc/m3tal/.env`). You can conveniently manage these variables using the `m3tal config wizard` for guided setup or `m3tal config set <KEY> <VALUE>` for direct modifications.

---

## Quick Reference

| Name                      | Description                                                  | Default             | Example                     | Used By                                            |
| :------------------------ | :----------------------------------------------------------- | :------------------ | :-------------------------- | :------------------------------------------------- |
| `API_PORT`                | Internal port for the M3TAL API daemon.                      | `5050`              | `5050`                      | M3TAL API daemon, M3TAL Dashboard                  |
| `DASHBOARD_PORT`          | Port for the M3TAL Dashboard.                                | `8082`              | `8082`                      | M3TAL Dashboard, Traefik (if `traefik` mode)       |
| `DASHBOARD_EXPOSE_MODE`   | How the M3TAL Dashboard is exposed: `local` or `traefik`.    | `local`             | `traefik`                   | M3TAL CLI, M3TAL Dashboard Compose                 |
| `HTTP_PORT`               | External host port for the M3TAL API daemon.                 | `8080`              | `8080`                      | M3TAL API daemon, Traefik                          |
| `STATE_DIR`               | Directory for the M3TAL state database and configurations.   | `./state`           | `/var/lib/m3tal/state`      | M3TAL API daemon, M3TAL Dashboard                  |
| `LOG_LEVEL`               | Minimum logging level for M3TAL services.                    | `info`              | `debug`                     | M3TAL CLI, M3TAL API daemon                        |
| `DEBUG_MODE`              | Enables debug features for M3TAL services.                   | `false`             | `true`                      | M3TAL API daemon                                   |
| `METRICS_ENABLED`         | Enables Prometheus metrics endpoint for M3TAL API daemon.    | `true`              | `false`                     | M3TAL API daemon                                   |
| `DASHBOARD_SECRET`        | Secret key for the M3TAL Dashboard Flask application.        | `change_me_immediately` | `super_secret_key_123`      | M3TAL Dashboard                                    |
| `API_TOKEN`               | Bearer token for authenticating with the M3TAL API daemon.   | `change_me_api_token` | `secure_api_token_xyz`      | M3TAL CLI, M3TAL Dashboard                         |
| `ADMIN_PASSWORD`          | Default password for administrative CLI tasks.               | `admin_pass`        | `MyStrongPassword123`       | M3TAL CLI                                          |
| `NETWORK_NAME`            | Name of the Docker network for M3TAL services.               | `m3tal`             | `m3tal-proxy`               | All Docker Compose stacks                          |
| `LOCAL_IP`                | Local IP address for specific bindings.                      | `127.0.0.1`         | `192.168.1.100`             | Potentially user-defined services                  |
| `DOMAIN`                  | Base domain name for Traefik routing rules.                  | `localhost`         | `my-server.com`             | Traefik gateway, M3TAL Dashboard Compose, CLI      |
| `BASE_STORAGE_PATH`       | Root directory for all persistent M3TAL data.                | `./data`            | `/mnt/m3tal`                | All Docker Compose stacks                          |
| `MEDIA_PATH`              | Subdirectory for media data.                                 | `./data/media`      | `/mnt/m3tal/media`          | User Docker Compose stacks                         |
| `CONFIG_PATH`             | Subdirectory for application configurations.                 | `./data/config`     | `/mnt/m3tal/config`         | User Docker Compose stacks, M3TAL Dashboard        |
| `DOWNLOADS_PATH`          | Subdirectory for downloaded content.                         | `./data/downloads`  | `/mnt/m3tal/downloads`      | User Docker Compose stacks                         |
| `PUID`                    | User ID for running container processes.                     | `1000`              | `1001`                      | All Docker Compose stacks                          |
| `PGID`                    | Group ID for running container processes.                    | `1000`              | `1001`                      | All Docker Compose stacks                          |
| `TZ`                      | Timezone for M3TAL services.                                 | `America/Denver`    | `Europe/London`             | All Docker Compose stacks                          |
| `TRAEFIK_WEB_PORT`        | Traefik's HTTP entry point port.                             | `80`                | `80`                        | Traefik gateway                                    |
| `TRAEFIK_WEBHTTPS_PORT`   | Traefik's HTTPS entry point port.                            | `443`               | `443`                       | Traefik gateway                                    |
| `TRAEFIK_DASHBOARD_PORT`  | Internal port for Traefik's own dashboard.                   | `8080`              | `8080`                      | Traefik gateway                                    |
| `VPN_USER`                | Username for VPN connection.                                 | `user`              | `m3taluser`                 | VPN service container                              |
| `VPN_PASSWORD`            | Password for VPN connection.                                 | `password`          | `MySecureVPNPass`           | VPN service container                              |

---

## Detailed Environment Variable Reference

### Core Configuration

These variables control the fundamental behavior and operational parameters of the M3TAL core components, including communication ports, logging, and state management.

#### `API_PORT`

*   **Description:** Specifies the internal port on which the M3TAL API daemon (`m3tal-api.service`) listens for requests from other M3TAL services, such as the Dashboard. This port is primarily used for inter-container communication via `host.docker.internal`.
*   **Default Value:** `5050`
*   **Example Value:** `5050`
*   **Used By:** M3TAL API daemon (for internal binding), M3TAL Dashboard (to connect to the API).

#### `DASHBOARD_PORT`

*   **Description:** Defines the port on which the `m3tal-dashboard` container listens internally.
    *   When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly bound to the host, allowing direct access via `http://HOST_IP:8082`.
    *   When `DASHBOARD_EXPOSE_MODE` is set to `traefik`, Traefik routes incoming requests from `dash.${DOMAIN}` to this internal port.
*   **Default Value:** `8082`
*   **Example Value:** `8082`
*   **Used By:** M3TAL Dashboard container, M3TAL CLI (for configuring `local` port binding), Traefik gateway (when `DASHBOARD_EXPOSE_MODE=traefik`).

#### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Determines how the M3TAL Dashboard is exposed to your network.
    *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly published to the host, making it accessible on your local network. This is ideal for LAN-only setups, first-time users, and local testing.
    *   `traefik`: The dashboard is exposed via the Traefik gateway using the subdomain `dash.${DOMAIN}`. This mode requires Traefik to be running via `m3tal up` and `DOMAIN` to be configured. Best for domain-based setups and services behind a reverse proxy.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:** M3TAL CLI (to select the appropriate Docker Compose override file), M3TAL Dashboard Compose stack.

#### `HTTP_PORT`

*   **Description:** Specifies the port on the host machine where the M3TAL API daemon (`m3tal-api.service`) is externally accessible. This is the port Traefik uses to route external requests to the API daemon, typically accessed as `http://host.docker.internal:8080` from other containers.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** M3TAL API daemon (for external binding), Traefik gateway (to route to the API daemon).

#### `STATE_DIR`

*   **Description:** Defines the directory on the host filesystem where the M3TAL API daemon stores its SQLite state database (`state.db`) and other critical configuration files. This path is volume-mounted into relevant containers, ensuring data persistence.
*   **Default Value:** `./state`
*   **Example Value:** `/var/lib/m3tal/state` (recommended for production deployments)
*   **Used By:** M3TAL API daemon, M3TAL Dashboard container.

#### `LOG_LEVEL`

*   **Description:** Sets the minimum logging severity level for the M3TAL CLI and API daemon. Messages below this level will be ignored. Valid values typically include `debug`, `info`, `warn`, `error`, `fatal`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:** M3TAL CLI, M3TAL API daemon.

#### `DEBUG_MODE`

*   **Description:** A boolean flag that enables or disables debug-specific features, more verbose logging, or development utilities within M3TAL services. Useful for troubleshooting.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:** M3TAL API daemon.

#### `METRICS_ENABLED`

*   **Description:** A boolean flag that enables or disables the Prometheus metrics endpoint for the M3TAL API daemon. When enabled, performance and operational metrics are exposed for collection by a monitoring system.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:** M3TAL API daemon.

### Authentication & Security

These variables are fundamental for securing access to M3TAL components and ensuring system integrity.

#### `DASHBOARD_SECRET`

*   **Description:** A unique, cryptographic secret key used by the M3TAL Dashboard (a Flask application) for session management, CSRF protection, and other security-sensitive operations. **This variable is automatically generated on the first `m3tal init` command. Users should NOT set it manually unless rotating the key for security reasons.**
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `super_secret_key_123_456_789`
*   **Used By:** M3TAL Dashboard container.

#### `API_TOKEN`

*   **Description:** A bearer token used for authenticating requests made to the M3TAL API daemon. This token secures communication between the M3TAL CLI, the M3TAL Dashboard, and the API. **This variable is automatically generated on the first `m3tal init` command. Users should NOT set it manually unless rotating the token for security reasons.**
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `secure_api_token_xyz_abc_123`
*   **Used By:** M3TAL CLI, M3TAL Dashboard container.

#### `ADMIN_PASSWORD`

*   **Description:** A default password primarily intended for administrative tasks performed via the M3TAL CLI or for initial system setup. Note that dashboard user credentials are managed separately within `/docker/users.json` using the `m3tal dashpass` command.
*   **Default Value:** `admin_pass`
*   **Example Value:** `MyStrongPassword123`
*   **Used By:** M3TAL CLI.

### Network & Routing

These variables define how M3TAL services communicate over the network and how they are exposed, particularly when integrating with the Traefik gateway.

#### `NETWORK_NAME`

*   **Description:** The name of the Docker network that M3TAL creates and uses for all its core services and user-defined Docker Compose stacks. This dedicated network allows services within the M3TAL ecosystem to communicate securely and efficiently with each other.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal-proxy`
*   **Used By:** All Docker Compose stacks (`m3tal`, `routing`, user-defined).

#### `LOCAL_IP`

*   **Description:** A specific local IP address that services might bind to or reference for internal communications. While not explicitly used by all core M3TAL components in their default configurations, it can be useful for advanced network setups or custom services requiring a specific host IP.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `192.168.1.100`
*   **Used By:** Potentially user-defined services or advanced network configurations.

#### `DOMAIN`

*   **Description:** The base domain name for your M3TAL deployment. This variable is crucial as it directly controls Traefik routing rules. Setting it enables domain-based access patterns such as `dash.DOMAIN` for the M3TAL Dashboard and `api.DOMAIN` for the M3TAL API daemon when Traefik is running.
*   **Default Value:** `localhost`
*   **Example Value:** `my-server.com`
*   **Used By:** Traefik gateway (`routing-compose.yml`), M3TAL Dashboard Compose (when `DASHBOARD_EXPOSE_MODE=traefik`), M3TAL CLI.

### Persistent Storage

These variables define the filesystem paths where M3TAL stores various types of persistent data, ensuring your information remains safe across restarts and updates.

#### `BASE_STORAGE_PATH`

*   **Description:** The root directory on the host filesystem for all persistent M3TAL data, serving as the base for media, configurations, downloads, and other application data. **In production deployments, this variable typically defaults to `/mnt` (e.g., `/mnt/m3tal`), rather than `./data` as seen in development templates.** This path is mounted into containers to provide persistent storage.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal`
*   **Used By:** All Docker Compose stacks (as the base for volume mounts).

#### `MEDIA_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` specifically designated for media files (e.g., movies, music, photos). This provides a standardized location for media content across your ecosystem.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal/media`
*   **Used By:** User Docker Compose stacks (e.g., Plex, Jellyfin, PhotoPrism).

#### `CONFIG_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` used for storing application-specific configuration files that need to persist across container restarts or updates. This ensures your application settings are retained.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal/config`
*   **Used By:** User Docker Compose stacks, M3TAL Dashboard (for `users.json`).

#### `DOWNLOADS_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` specifically intended for downloaded content. This provides a dedicated and persistent location for files retrieved by download clients.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal/downloads`
*   **Used By:** User Docker Compose stacks (e.g., download clients like qBittorrent, Transmission).

### System & Environment

These variables provide general system-level and environment configurations for containers within the M3TAL ecosystem.

#### `PUID`

*   **Description:** The User ID (UID) that container processes will run as. Setting this ensures correct file ownership and permissions for volumes mounted from the host system, preventing permission issues. It should typically match the UID of a non-root user on your host machine.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:** All Docker Compose stacks.

#### `PGID`

*   **Description:** The Group ID (GID) that container processes will run as. Similar to `PUID`, this ensures correct file group ownership and permissions for mounted volumes. It should typically match the GID of a group on your host machine.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:** All Docker Compose stacks.

#### `TZ`

*   **Description:** Sets the timezone for all M3TAL containers and services. This is crucial for accurate logging, scheduling, and any time-based operations within your M3TAL ecosystem.
*   **Default Value:** `America/Denver`
*   **Example Value:** `Europe/London`
*   **Used By:** All Docker Compose stacks.

### Traefik Gateway

These variables are used to configure the Traefik reverse proxy, which acts as the intelligent gateway for external access to your M3TAL services.

#### `TRAEFIK_WEB_PORT`

*   **Description:** Specifies the port Traefik uses as its primary HTTP entry point on the host machine. This is typically port `80` for standard web traffic.
*   **Default Value:** `80`
*   **Example Value:** `80`
*   **Used By:** Traefik gateway (`routing-compose.yml`).

#### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** Specifies the port Traefik uses as its primary HTTPS entry point on the host machine. This is typically port `443` for secure web traffic (requires additional Traefik configuration, such as certificates, for HTTPS termination).
*   **Default Value:** `443`
*   **Example Value:** `443`
*   **Used By:** Traefik gateway (`routing-compose.yml`).

#### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** Defines the *internal* port within the Traefik container where its own management dashboard is accessible. While Traefik's internal dashboard runs on this port, it's typically exposed on the host via a specific local binding like `127.0.0.1:8081`.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** Traefik gateway (`routing-compose.yml`).

### VPN Service

These variables are used to configure an optional VPN service that might be integrated into the M3TAL ecosystem, enabling secure external connectivity.

#### `VPN_USER`

*   **Description:** The username required for authentication with a configured VPN service. This variable provides credentials for establishing and maintaining a secure VPN connection.
*   **Default Value:** `user`
*   **Example Value:** `m3taluser`
*   **Used By:** VPN service container (e.g., for generic OpenVPN or WireGuard clients; Cloudflared typically uses tunnel tokens).

#### `VPN_PASSWORD`

*   **Description:** The password required for authentication with a configured VPN service. This variable provides the necessary credentials alongside `VPN_USER`.
*   **Default Value:** `password`
*   **Example Value:** `MySecureVPNPass`
*   **Used By:** VPN service container.