# Environment Variables Reference

Greetings, M3TAL Operative. This document serves as the definitive reference for all environment variables used by the M3TAL ecosystem. These variables govern the behavior of the M3TAL CLI, the API daemon, and all associated Docker Compose stacks.

All environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. Both the `m3tal` CLI and all Docker Compose stacks (managed by `m3tal up` and `m3tal dash up`) load their configurations from this file via the `--env-file` option. You can manage these variables using the `m3tal config wizard` or `m3tal config set <KEY> <VALUE>` commands.

---

### Quick Reference

| Name                    | Description                                                                                                                                                             | Default Value             | Component(s)                                   |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------ | :--------------------------------------------- |
| `API_PORT`              | Port the M3TAL Dashboard expects the API daemon to be accessible on (internally or via `host.docker.internal`).                                                         | `5050`                    | M3TAL Dashboard                                |
| `DASHBOARD_PORT`        | Internal port for the M3TAL Dashboard container, and the host port in `local` expose mode.                                                                              | `8082`                    | M3TAL Dashboard, Traefik, M3TAL CLI            |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL Dashboard is exposed: `local` (direct port) or `traefik` (via `dash.DOMAIN`).                                                                    | `local`                   | M3TAL CLI, M3TAL Dashboard compose             |
| `HTTP_PORT`             | The host port where the `m3tal-api.service` (Go daemon) listens for incoming API requests.                                                                              | `8080`                    | M3TAL API daemon                               |
| `STATE_DIR`             | Path for state files. For the API daemon, it's `/var/lib/m3tal`. For dashboard, it's internal `/docker/state` mapped from `CONFIG_PATH`. CLI uses `./state`.           | `./state`                 | M3TAL CLI, M3TAL API daemon, M3TAL Dashboard   |
| `LOG_LEVEL`             | Sets the logging verbosity for M3TAL components.                                                                                                                        | `info`                    | M3TAL CLI, M3TAL API daemon                    |
| `DASHBOARD_SECRET`      | Secret key for M3TAL Dashboard session management. **Auto-generated on `m3tal init`.**                                                                                  | `change_me_immediately`   | M3TAL Dashboard                                |
| `API_TOKEN`             | Authentication token for programmatic API access. **Auto-generated on `m3tal init`.**                                                                                   | `change_me_api_token`     | M3TAL CLI, M3TAL Dashboard, M3TAL API daemon   |
| `ADMIN_PASSWORD`        | Initial administrator password for the M3TAL Dashboard.                                                                                                                 | `admin_pass`              | M3TAL Dashboard (`users.json`)                 |
| `NETWORK_NAME`          | Name of the Docker network for M3TAL containers to communicate.                                                                                                         | `m3tal`                   | All Docker Compose stacks                      |
| `LOCAL_IP`              | Specifies a local IP address for services to bind or reference.                                                                                                         | `127.0.0.1`               | Traefik, Docker services                       |
| `DOMAIN`                | The base domain for Traefik routing. Enables `dash.DOMAIN` and `api.DOMAIN` routes.                                                                                     | `localhost`               | Traefik, M3TAL Dashboard compose               |
| `VPN_USER`              | Username for VPN client authentication.                                                                                                                                 | `user`                    | VPN Services (e.g., WireGuard)                 |
| `VPN_PASSWORD`          | Password for VPN client authentication.                                                                                                                                 | `password`                | VPN Services (e.g., WireGuard)                 |
| `BASE_STORAGE_PATH`     | Base directory for M3TAL's media and configuration data. **Defaults to `/mnt` in production.**                                                                          | `./data`                  | M3TAL Dashboard, User Docker stacks            |
| `MEDIA_PATH`            | Subdirectory within `BASE_STORAGE_PATH` for media files.                                                                                                                | `./data/media`            | M3TAL Dashboard, User Docker stacks            |
| `CONFIG_PATH`           | Subdirectory within `BASE_STORAGE_PATH` for configuration.                                                                                                              | `./data/config`           | M3TAL API daemon, M3TAL Dashboard, User stacks |
| `DOWNLOADS_PATH`        | Subdirectory within `BASE_STORAGE_PATH` for downloaded content.                                                                                                         | `./data/downloads`        | User Docker stacks                             |
| `PUID`                  | User ID for container processes.                                                                                                                                        | `1000`                    | All Docker containers                          |
| `PGID`                  | Group ID for container processes.                                                                                                                                       | `1000`                    | All Docker containers                          |
| `TZ`                    | Timezone for containers and services.                                                                                                                                   | `America/Denver`          | M3TAL Dashboard, User Docker stacks            |
| `TRAEFIK_WEB_PORT`      | The HTTP port Traefik listens on for incoming web requests.                                                                                                             | `80`                      | Traefik gateway                                |
| `TRAEFIK_WEBHTTPS_PORT` | The HTTPS port Traefik listens on for incoming web requests (if HTTPS is configured).                                                                                   | `443`                     | Traefik gateway                                |
| `TRAEFIK_DASHBOARD_PORT` | The internal container port for Traefik's own dashboard. Exposed on host port 8081.                                                                                     | `8080`                    | Traefik gateway                                |
| `DEBUG_MODE`            | Enables debug logging and potentially other debug features.                                                                                                             | `false`                   | M3TAL CLI, M3TAL API daemon                    |
| `METRICS_ENABLED`       | Enables or disables metrics collection for M3TAL components.                                                                                                            | `true`                    | M3TAL API daemon, M3TAL CLI                    |

---

### Detailed Variable Reference

This section provides an in-depth explanation for each environment variable.

#### Core System & API

These variables control the fundamental behavior and communication of the M3TAL API daemon and CLI.

*   **`API_PORT`**
    *   **Description:** Specifies the port that the M3TAL Dashboard (and potentially other services) expects the M3TAL API daemon to be accessible on. The dashboard specifically targets `http://host.docker.internal:${API_PORT}`. Note that the API daemon itself runs on the host at `HTTP_PORT` (default 8080).
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Component(s):** M3TAL Dashboard
*   **`HTTP_PORT`**
    *   **Description:** The host port on which the `m3tal-api.service` (Go daemon) listens for incoming API requests from the CLI, Traefik, and other local services.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component(s):** M3TAL API daemon
*   **`STATE_DIR`**
    *   **Description:** Defines the base directory for M3TAL's internal state management.
        *   For the **M3TAL API daemon (`m3tal-api.service`)**, the SQLite state database is explicitly stored at `/var/lib/m3tal/state.db`. This variable is not directly used by the production API daemon for its database location.
        *   For the **M3TAL Dashboard container**, its internal `/docker/state` path is volume-mounted from `${CONFIG_PATH}/m3tal/state` on the host.
        *   For the **M3TAL CLI** or non-production API daemon instances, this variable indicates where relative state files (e.g., `./state.db`) would be stored.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal`
    *   **Component(s):** M3TAL CLI, M3TAL API daemon (non-production/development), M3TAL Dashboard (indirectly via `CONFIG_PATH` mapping).
*   **`LOG_LEVEL`**
    *   **Description:** Controls the verbosity of log output from M3TAL components. Available levels typically include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Component(s):** M3TAL CLI, M3TAL API daemon
*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, enables detailed debugging information and potentially alters behavior for easier troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Component(s):** M3TAL CLI, M3TAL API daemon
*   **`METRICS_ENABLED`**
    *   **Description:** When set to `true`, M3TAL components will collect and expose operational metrics for monitoring.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Component(s):** M3TAL CLI, M3TAL API daemon

#### Authentication

These variables are crucial for securing access to the M3TAL dashboard and API.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A long, randomly generated secret string used by the M3TAL Dashboard for secure session management, cookie signing, and other cryptographic operations.
    *   **Important:** This variable is **auto-generated on first `m3tal init`**. Users should **NOT set it manually** unless they are intentionally rotating the secret for security reasons.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `k3l9x8m2-s7p4t0j1-q5r6w3v8-e0c1a2z9`
    *   **Component(s):** M3TAL Dashboard
*   **`API_TOKEN`**
    *   **Description:** An authentication token used by the M3TAL CLI and Dashboard to authenticate with the M3TAL API daemon. It provides programmatic access control.
    *   **Important:** This variable is **auto-generated on first `m3tal init`**. Users should **NOT set it manually** unless they are intentionally rotating the token for security reasons.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6`
    *   **Component(s):** M3TAL CLI, M3TAL Dashboard, M3TAL API daemon
*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial default password for the `admin` user account in the M3TAL Dashboard, used when the `users.json` file is first created. It is highly recommended to change this immediately after first login via the dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MySuperSecurePassword123!`
    *   **Component(s):** M3TAL Dashboard (`/docker/users.json`)

#### Network Configuration

Variables defining the networking environment for M3TAL services.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the custom Docker network that M3TAL uses for inter-container communication. All M3TAL managed containers (dashboard, Traefik, user stacks) are connected to this network.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `my_m3tal_network`
    *   **Component(s):** All Docker Compose stacks
*   **`LOCAL_IP`**
    *   **Description:** Specifies a local IP address that services might use to bind to a specific host interface or for internal referencing within the network.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Component(s):** Traefik, Docker services

#### Storage & Permissions

These variables control where M3TAL stores its data and with which user permissions containers operate.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The absolute host path that serves as the root directory for all M3TAL-related media, configuration, and other persistent data. Child paths like `MEDIA_PATH`, `CONFIG_PATH`, and `DOWNLOADS_PATH` are typically relative to this base.
    *   **Important:** While the template defaults to `./data`, in **production deployments, this path typically defaults to `/mnt`** for media and persistent storage.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal_data`
    *   **Component(s):** M3TAL Dashboard, User Docker stacks
*   **`MEDIA_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` designated for media files (e.g., videos, music, images).
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal_data/media`
    *   **Component(s):** M3TAL Dashboard, User Docker stacks
*   **`CONFIG_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` used for storing configuration files and other persistent application data (e.g., the dashboard's `users.json`, state files).
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal_data/config`
    *   **Component(s):** M3TAL API daemon (for state.db if mapped here), M3TAL Dashboard, User Docker stacks
*   **`DOWNLOADS_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` intended for downloaded content.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal_data/downloads`
    *   **Component(s):** User Docker stacks
*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers should use to run their processes. This ensures that files created by containers have the correct ownership on the host filesystem, preventing permission issues.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component(s):** All Docker containers
*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers should use to run their processes. This works in conjunction with `PUID` for proper file permissions.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component(s):** All Docker containers

#### M3TAL Dashboard

Variables specific to configuring the M3TAL Dashboard container and its exposure.

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly mapped to the host.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Component(s):** M3TAL Dashboard, M3TAL CLI, Traefik
*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible from the host network.
        *   **`local` (default):** The dashboard port (`DASHBOARD_PORT`) is directly bound to the host, allowing access via `http://HOST_IP:8082` or `http://localhost:8082`. This mode is ideal for LAN-only setups and initial deployments, requiring no Traefik configuration.
        *   **`traefik`:** The dashboard container is configured with Traefik labels, enabling Traefik to route `http://dash.${DOMAIN}` to the dashboard. This mode requires Traefik to be running via `m3tal up` and is best for domain-based access.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Component(s):** M3TAL CLI (`m3tal dash up`), M3TAL Dashboard compose files

#### Traefik & Routing

Variables for configuring the Traefik reverse proxy and its routing rules.

*   **`DOMAIN`**
    *   **Description:** The base domain name used by Traefik for routing rules. Setting this value enables convenient access patterns like `http://dash.${DOMAIN}` for the M3TAL Dashboard and `http://api.${DOMAIN}` for the M3TAL API daemon.
    *   **Default Value:** `localhost`
    *   **Example Value:** `example.com`
    *   **Component(s):** Traefik gateway, M3TAL Dashboard compose (when `DASHBOARD_EXPOSE_MODE=traefik`)
*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik listens on for incoming HTTP web requests. This is Traefik's primary entry point for unencrypted traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Component(s):** Traefik gateway
*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik listens on for incoming HTTPS web requests. This is used when TLS/SSL certificates are configured for secure traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Component(s):** Traefik gateway
*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal container port on which the Traefik management dashboard listens. On the host, this is typically mapped to `127.0.0.1:8081`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component(s):** Traefik gateway

#### VPN Services

Variables for configuring optional VPN services within the M3TAL ecosystem.

*   **`VPN_USER`**
    *   **Description:** The username for authenticating with a VPN service (e.g., WireGuard, OpenVPN client).
    *   **Default Value:** `user`
    *   **Example Value:** `vpnclient`
    *   **Component(s):** VPN services (e.g., `cloudflared` tunnel, other user VPN stacks)
*   **`VPN_PASSWORD`**
    *   **Description:** The password for authenticating with a VPN service.
    *   **Default Value:** `password`
    *   **Example Value:** `secureVPNpass123`
    *   **Component(s):** VPN services (e.g., `cloudflared` tunnel, other user VPN stacks)

#### Locale

*   **`TZ`**
    *   **Description:** Sets the timezone for all containers and services, ensuring consistent time display and logging. Use standard IANA timezone identifiers (e.g., `America/New_York`, `Europe/London`).
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Component(s):** M3TAL Dashboard, User Docker stacks