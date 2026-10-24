docs/ENVIRONMENT_VARIABLES.md

# M3TAL Environment Variables

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables serve as the primary configuration mechanism for the CLI, the API daemon, and all Docker Compose stacks.

All M3TAL environment variables are read from the central configuration file, `/etc/m3tal/.env`. This file is loaded by the `m3tal` CLI binary, the `m3tal-api.service`, and passed to all Docker Compose stacks via the `--env-file` argument, ensuring consistent configuration across the entire system. You can manage these variables using the `m3tal config wizard` or `m3tal config set KEY value` commands.

---

## Quick Reference

| Variable Name           | Group               | Default Value             | Components Used By                              |
|-------------------------|---------------------|---------------------------|-------------------------------------------------|
| `API_PORT`              | Core                | `5050`                    | API daemon, Dashboard                           |
| `DASHBOARD_PORT`        | Core                | `8082`                    | Dashboard, CLI                                  |
| `DASHBOARD_EXPOSE_MODE` | Core                | `local`                   | CLI, Dashboard (via Compose overrides)          |
| `HTTP_PORT`             | Core                | `8080`                    | API daemon, Traefik                             |
| `STATE_DIR`             | Core                | `./state`                 | API daemon, Dashboard                           |
| `LOG_LEVEL`             | Core                | `info`                    | CLI, API daemon, Dashboard                      |
| `DASHBOARD_SECRET`      | Auth & Security     | `change_me_immediately`   | Dashboard                                       |
| `API_TOKEN`             | Auth & Security     | `change_me_api_token`     | API daemon, CLI                                 |
| `ADMIN_PASSWORD`        | Auth & Security     | `admin_pass`              | Dashboard (initial setup)                       |
| `NETWORK_NAME`          | Network             | `m3tal`                   | All Docker containers                           |
| `LOCAL_IP`              | Network             | `127.0.0.1`               | CLI, API daemon                                 |
| `DOMAIN`                | Network             | `localhost`               | Traefik, API daemon, Dashboard                  |
| `BASE_STORAGE_PATH`     | Storage             | `./data`                  | Dashboard, User stacks                          |
| `MEDIA_PATH`            | Storage             | `./data/media`            | Dashboard, User stacks                          |
| `CONFIG_PATH`           | Storage             | `./data/config`           | Dashboard, User stacks                          |
| `DOWNLOADS_PATH`        | Storage             | `./data/downloads`        | User stacks                                     |
| `TRAEFIK_WEB_PORT`      | Traefik Gateway     | `80`                      | Traefik container                               |
| `TRAEFIK_WEBHTTPS_PORT` | Traefik Gateway     | `443`                     | Traefik container                               |
| `TRAEFIK_DASHBOARD_PORT`| Traefik Gateway     | `8080`                    | Traefik container                               |
| `VPN_USER`              | VPN Tunnel          | `user`                    | User-defined VPN client containers              |
| `VPN_PASSWORD`          | VPN Tunnel          | `password`                | User-defined VPN client containers              |
| `PUID`                  | System & Debugging  | `1000`                    | All Docker containers                           |
| `PGID`                  | System & Debugging  | `1000`                    | All Docker containers                           |
| `TZ`                    | System & Debugging  | `America/Denver`          | CLI, API daemon, Dashboard, User stacks         |
| `DEBUG_MODE`            | System & Debugging  | `false`                   | CLI, API daemon, Dashboard                      |
| `METRICS_ENABLED`       | System & Debugging  | `true`                    | API daemon                                      |

---

## Detailed Variable Reference

### Core Configuration

These variables control the fundamental operation and networking of M3TAL's core components.

#### `API_PORT`
*   **Description**: Specifies the internal port on which the M3TAL API daemon listens for communication from other services (e.g., the dashboard container). The dashboard connects to the API via `host.docker.internal:${API_PORT}`.
*   **Default Value**: `5050`
*   **Example Value**: `5050`
*   **Components Used By**: `m3tal-api.service` (listens), `m3tal-dashboard` (connects).

#### `DASHBOARD_PORT`
*   **Description**: Defines the internal port the M3TAL Dashboard container binds to. If `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host.
*   **Default Value**: `8082`
*   **Example Value**: `8082`
*   **Components Used By**: `m3tal-dashboard`, CLI (`m3tal dash up`).

#### `DASHBOARD_EXPOSE_MODE`
*   **Description**: Controls how the M3TAL Dashboard is exposed to the network.
    *   `local`: Adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the host, allowing access via `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik required. Best for LAN-only setups or first-time users.
    *   `traefik`: Configures the dashboard with Traefik labels, enabling routing via `http://dash.DOMAIN`. Requires Traefik to be running and `DOMAIN` to be set. Best for domain-based setups.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Components Used By**: CLI (`m3tal dash up`), `m3tal-dashboard` (via Compose override files).

#### `HTTP_PORT`
*   **Description**: Specifies the port on which the M3TAL API daemon (`m3tal-api.service`) runs on the host system. This is the port Traefik uses to proxy requests to the API via `http://host.docker.internal:${HTTP_PORT}`.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Components Used By**: `m3tal-api.service`, `traefik` (routes to it via dynamic config).

#### `STATE_DIR`
*   **Description**: Defines the path to the directory where the M3TAL API daemon stores its SQLite state database (`state.db`). This directory is also mounted into the dashboard container.
*   **Default Value**: `./state`
*   **Example Value**: `/var/lib/m3tal/state` (typical production path)
*   **Components Used By**: `m3tal-api.service`, `m3tal-dashboard`.

#### `LOG_LEVEL`
*   **Description**: Sets the verbosity of logging for M3TAL components. Available values typically include `debug`, `info`, `warn`, `error`.
*   **Default Value**: `info`
*   **Example Value**: `debug`
*   **Components Used By**: CLI, `m3tal-api.service`, `m3tal-dashboard`.

### Authentication & Security

These variables are critical for securing access to M3TAL's dashboard and API.

#### `DASHBOARD_SECRET`
*   **Description**: A unique secret key used by the M3TAL Dashboard for session management and securing user credentials.
    **IMPORTANT**: This variable is auto-generated on the first `m3tal init` and users should **NOT** set it manually unless they are intentionally rotating it (e.g., for security policy reasons), which will invalidate existing dashboard sessions.
*   **Default Value**: `change_me_immediately` (placeholder)
*   **Example Value**: `your_long_and_complex_generated_secret_here`
*   **Components Used By**: `m3tal-dashboard`.

#### `API_TOKEN`
*   **Description**: An authentication token used to secure access to the M3TAL API daemon. The CLI uses this token to authenticate with the API.
    **IMPORTANT**: This variable is auto-generated on the first `m3tal init` and users should **NOT** set it manually unless they are intentionally rotating it (e.g., for security policy reasons).
*   **Default Value**: `change_me_api_token` (placeholder)
*   **Example Value**: `a_very_secure_auto_generated_api_token_value`
*   **Components Used By**: `m3tal-api.service`, CLI.

#### `ADMIN_PASSWORD`
*   **Description**: The default password for the initial "admin" user account in the M3TAL Dashboard.
    **IMPORTANT**: It is highly recommended to change this immediately after first login to a strong, unique password.
*   **Default Value**: `admin_pass`
*   **Example Value**: `MyStrongAndSecurePa$$word123`
*   **Components Used By**: `m3tal-dashboard` (initial setup).

### Network Configuration

Variables related to Docker networking and host-level network settings.

#### `NETWORK_NAME`
*   **Description**: Specifies the name of the Docker bridge network used by M3TAL's compose stacks for inter-container communication. By default, this is typically set to `proxy` and is an external network.
*   **Default Value**: `m3tal`
*   **Example Value**: `proxy`
*   **Components Used By**: All Docker containers (via `docker compose` network definitions).

#### `LOCAL_IP`
*   **Description**: The local IP address of the M3TAL host system. Used for internal routing and service discovery.
*   **Default Value**: `127.0.0.1`
*   **Example Value**: `192.168.1.100`
*   **Components Used By**: CLI, `m3tal-api.service`.

#### `DOMAIN`
*   **Description**: The base domain name for your M3TAL deployment. Setting this variable activates Traefik routing rules, enabling access to the dashboard at `http://dash.DOMAIN` and the API at `http://api.DOMAIN`. Traefik uses this for `Host()` rules.
*   **Default Value**: `localhost`
*   **Example Value**: `myhomelab.com`
*   **Components Used By**: `traefik`, `m3tal-dashboard` (via Traefik labels), `m3tal-api.service` (via Traefik dynamic config).

### Storage Configuration

Variables defining paths for persistent data storage.

#### `BASE_STORAGE_PATH`
*   **Description**: The root directory where M3TAL stores all persistent media and configuration data. All other storage paths (`MEDIA_PATH`, `CONFIG_PATH`, `DOWNLOADS_PATH`) are typically subdirectories of this path.
    **IMPORTANT**: In production deployments, this variable defaults to `/mnt` to align with typical server storage mounting points for external drives. However, in the template and for local development, it defaults to `./data`. Users should adjust this to a suitable volume mount point in their production environment (e.g., `/mnt/m3tal`).
*   **Default Value**: `./data` (template/dev)
*   **Example Value**: `/mnt/m3tal` (production)
*   **Components Used By**: `m3tal-dashboard`, various user-defined stacks.

#### `MEDIA_PATH`
*   **Description**: Subdirectory within `BASE_STORAGE_PATH` designated for storing media files managed by M3TAL or user-defined services.
*   **Default Value**: `./data/media`
*   **Example Value**: `/mnt/m3tal/media`
*   **Components Used By**: `m3tal-dashboard` (if managing media), user-defined stacks.

#### `CONFIG_PATH`
*   **Description**: Subdirectory within `BASE_STORAGE_PATH` for storing general configuration files for services. The dashboard mounts `/etc/m3tal/.env` into this path.
*   **Default Value**: `./data/config`
*   **Example Value**: `/mnt/m3tal/config`
*   **Components Used By**: `m3tal-dashboard`, user-defined stacks.

#### `DOWNLOADS_PATH`
*   **Description**: Subdirectory within `BASE_STORAGE_PATH` designated for temporary or permanent download storage by services.
*   **Default Value**: `./data/downloads`
*   **Example Value**: `/mnt/m3tal/downloads`
*   **Components Used By**: User-defined stacks.

### Traefik Gateway

Variables specific to the Traefik reverse proxy.

#### `TRAEFIK_WEB_PORT`
*   **Description**: The host port that Traefik listens on for unencrypted HTTP traffic (using the `web` entrypoint).
*   **Default Value**: `80`
*   **Example Value**: `80`
*   **Components Used By**: `traefik` container.

#### `TRAEFIK_WEBHTTPS_PORT`
*   **Description**: The host port that Traefik listens on for encrypted HTTPS traffic (typically using a `websecure` entrypoint, though not explicitly configured in the provided minimal `routing-compose.yml`).
*   **Default Value**: `443`
*   **Example Value**: `443`
*   **Components Used By**: `traefik` container.

#### `TRAEFIK_DASHBOARD_PORT`
*   **Description**: The internal port Traefik uses to expose its own administration dashboard. By default, this is mapped to `127.0.0.1:8081` on the host.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Components Used By**: `traefik` container.

### VPN Tunnel

These variables are placeholders for configuring user-defined VPN client containers, such as those that might utilize credentials.

#### `VPN_USER`
*   **Description**: Username for connecting to an optional user-defined VPN service (e.g., a custom WireGuard client container). Note that the `cloudflared` tunnel typically uses `TUNNEL_TOKEN` or `TUNNEL_ID` and `TUNNEL_SECRET`, not a username/password.
*   **Default Value**: `user`
*   **Example Value**: `myvpnuser`
*   **Components Used By**: User-defined VPN client containers.

#### `VPN_PASSWORD`
*   **Description**: Password for connecting to an optional user-defined VPN service (e.g., a custom WireGuard client container).
*   **Default Value**: `password`
*   **Example Value**: `mypassword`
*   **Components Used By**: User-defined VPN client containers.

### System & Debugging

General system-level configuration and debugging toggles.

#### `PUID`
*   **Description**: The User ID (UID) that containers within the M3TAL ecosystem should run as. This ensures proper file ownership and permissions for mounted volumes. Typically, this is the UID of the user who owns the `/opt/m3tal` directory and storage paths.
*   **Default Value**: `1000`
*   **Example Value**: `1000` (common for the first non-root user)
*   **Components Used By**: All Docker containers that define `user: "${PUID}:${PGID}"`.

#### `PGID`
*   **Description**: The Group ID (GID) that containers within the M3TAL ecosystem should run as. This ensures proper file ownership and permissions for mounted volumes. Typically, this is the GID of the primary group for the user defined by `PUID`.
*   **Default Value**: `1000`
*   **Example Value**: `1000` (common for the first non-root user's primary group)
*   **Components Used By**: All Docker containers that define `user: "${PUID}:${PGID}"`.

#### `TZ`
*   **Description**: Sets the timezone for M3TAL components and containers, ensuring consistent timestamps across logs and services. Uses standard timezone names (e.g., `America/New_York`, `Europe/London`).
*   **Default Value**: `America/Denver`
*   **Example Value**: `Europe/Berlin`
*   **Components Used By**: CLI, `m3tal-api.service`, `m3tal-dashboard`, user-defined stacks.

#### `DEBUG_MODE`
*   **Description**: Enables or disables debugging features and potentially more verbose logging across the system. Setting this to `true` may expose more detailed information useful for troubleshooting.
*   **Default Value**: `false`
*   **Example Value**: `true`
*   **Components Used By**: CLI, `m3tal-api.service`, `m3tal-dashboard`.

#### `METRICS_ENABLED`
*   **Description**: Controls whether system and service metrics collection is enabled within the M3TAL API daemon. If set to `true`, the API may expose a metrics endpoint.
*   **Default Value**: `true`
*   **Example Value**: `false`
*   **Components Used By**: `m3tal-api.service`.