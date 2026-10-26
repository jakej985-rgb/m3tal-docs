# Environment Variables Reference

All M3TAL system configurations are managed through environment variables stored in the primary configuration file: `/etc/m3tal/.env`. This file is centrally managed by the `m3tal config wizard` and `m3tal config set` commands. Both the `m3tal` CLI binary and all Docker Compose stacks read their configurations from `/etc/m3tal/.env` via the `--env-file` argument, ensuring a consistent and unified configuration across the entire ecosystem.

---

## Quick Reference

| Name                    | Description                                                            | Default Value           | Example Value             |
| :---------------------- | :--------------------------------------------------------------------- | :---------------------- | :------------------------ |
| `API_PORT`              | Internal listener port for the API daemon.                             | `5050`                  | `8080`                    |
| `DASHBOARD_PORT`        | Internal port for the M3TAL Dashboard container.                       | `8082`                  | `8082`                    |
| `DASHBOARD_EXPOSE_MODE` | Controls Dashboard exposure: `local` (direct) or `traefik` (via proxy). | `local`                 | `traefik`                 |
| `HTTP_PORT`             | Default external HTTP port for services.                               | `8080`                  | `80`                      |
| `STATE_DIR`             | General reference path for state data.                                 | `./state`               | `/var/lib/m3tal/state`    |
| `LOG_LEVEL`             | Logging verbosity for M3TAL components.                                | `info`                  | `debug`                   |
| `DASHBOARD_SECRET`      | Secret key for Dashboard session management.                           | `change_me_immediately` | `a_long_random_string`    |
| `API_TOKEN`             | Authentication token for CLI-API communication.                        | `change_me_api_token`   | `another_long_string`     |
| `ADMIN_PASSWORD`        | Initial password for the `admin` user on the Dashboard.                | `admin_pass`            | `myStrongPassword123!`    |
| `NETWORK_NAME`          | Name of the Docker network for M3TAL services.                         | `m3tal`                 | `m3tal_net`               |
| `LOCAL_IP`              | Host IP for containers to resolve `host.docker.internal`.              | `127.0.0.1`             | `192.168.1.100`           |
| `DOMAIN`                | Base domain name for M3TAL services.                                   | `localhost`             | `example.com`             |
| `VPN_USER`              | Username for optional VPN component.                                   | `user`                  | `m3tal_vpn_user`          |
| `VPN_PASSWORD`          | Password for optional VPN component.                                   | `password`              | `secure_vpn_pass`         |
| `BASE_STORAGE_PATH`     | Base directory for all M3TAL media and configuration data.             | `./data`                | `/mnt/m3tal_data`         |
| `MEDIA_PATH`            | Subdirectory for user-generated media.                                 | `./data/media`          | `/mnt/m3tal_data/media`   |
| `CONFIG_PATH`           | Subdirectory for M3TAL configuration files and persistent data.        | `./data/config`         | `/mnt/m3tal_data/config`  |
| `DOWNLOADS_PATH`        | Subdirectory for downloaded content.                                   | `./data/downloads`      | `/mnt/m3tal_data/downloads` |
| `PUID`                  | User ID for Docker containers.                                         | `1000`                  | `1001`                    |
| `PGID`                  | Group ID for Docker containers.                                        | `1000`                  | `1001`                    |
| `TZ`                    | Timezone setting for M3TAL services.                                   | `America/Denver`        | `Europe/London`           |
| `TRAEFIK_WEB_PORT`      | Host port Traefik uses for HTTP entrypoint.                            | `80`                    | `8080`                    |
| `TRAEFIK_WEBHTTPS_PORT` | Host port Traefik uses for HTTPS entrypoint.                           | `443`                   | `8443`                    |
| `TRAEFIK_DASHBOARD_PORT`| Internal Traefik UI port (mapped to `127.0.0.1:8081` on host).       | `8080`                  | `8080`                    |
| `DEBUG_MODE`            | Enables verbose debugging output.                                      | `false`                 | `true`                    |
| `METRICS_ENABLED`       | Enables or disables metrics collection.                                | `true`                  | `false`                   |

---

## Detailed Environment Variable Reference

### Core

These variables control fundamental aspects of the M3TAL system's operation and core service communication.

*   **`API_PORT`**
    *   **Description**: The internal port on which the M3TAL API daemon (`m3tal-api.service`) is configured to listen. While this variable specifies the internal listener port, the API daemon is typically exposed on port `8080` on the host and accessed via Traefik or `http://localhost:8080`.
    *   **Default Value**: `5050`
    *   **Example Value**: `8080`
    *   **Component Uses It**: `m3tal-api.service`, CLI (if configured to use this port for direct connection)

*   **`DASHBOARD_PORT`**
    *   **Description**: The internal port on which the `m3tal-dashboard` container listens. This port is used for direct local access (`DASHBOARD_EXPOSE_MODE=local`) or for Traefik to route traffic to the Dashboard (`DASHBOARD_EXPOSE_MODE=traefik`).
    *   **Default Value**: `8082`
    *   **Example Value**: `8082`
    *   **Component Uses It**: `m3tal-dashboard`, Docker Compose overrides, Traefik

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description**: Controls how the M3TAL Dashboard is exposed to the network.
        *   `local`: The dashboard container binds directly to `HOST_IP:DASHBOARD_PORT`, making it accessible via `http://HOST_IP:8082`. This is ideal for LAN-only setups and initial deployment.
        *   `traefik`: The dashboard is configured with Traefik labels, making it accessible via `http://dash.DOMAIN` through the Traefik reverse proxy. Requires `DOMAIN` to be set and Traefik to be running.
    *   **Default Value**: `local`
    *   **Example Value**: `traefik`
    *   **Component Uses It**: CLI (`m3tal dash up`), Docker Compose overrides (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`)

*   **`HTTP_PORT`**
    *   **Description**: A general-purpose default external HTTP port for various M3TAL services or custom stack configurations. It aligns with the typical external exposure of the API daemon on port `8080`.
    *   **Default Value**: `8080`
    *   **Example Value**: `80`
    *   **Component Uses It**: `m3tal-api.service` (externally), other Docker Compose stacks

*   **`STATE_DIR`**
    *   **Description**: A general reference path for state-related data. While the `m3tal-api.service` manages the primary SQLite state database at `/var/lib/m3tal/state.db` on the host, and the Dashboard mounts its state data to `/docker/state` (sourced from `${CONFIG_PATH}/m3tal/state`), this variable provides a configurable base for other state-related paths or internal component configurations.
    *   **Default Value**: `./state`
    *   **Example Value**: `/var/lib/m3tal/state`
    *   **Component Uses It**: CLI (as a hint for some operations), `m3tal-dashboard` (for internal paths related to state mounts)

### Authentication

These variables manage access and security for M3TAL components.

*   **`DASHBOARD_SECRET`**
    *   **Description**: A critical secret key used by the `m3tal-dashboard` for session management, cryptographic operations, and overall security.
    *   **Important Note**: This variable is automatically generated on the first `m3tal init`. Users should **NOT** set this manually unless performing a secret rotation and understanding the implications.
    *   **Default Value**: `change_me_immediately`
    *   **Example Value**: `sdFJKl34jKLm5N6oP7Q8Rs9TuVwXyZabCDEFGH` (a long, random string)
    *   **Component Uses It**: `m3tal-dashboard`

*   **`API_TOKEN`**
    *   **Description**: An authentication token used by the `m3tal` CLI to securely communicate with the `m3tal-api.service` daemon.
    *   **Important Note**: This variable is automatically generated on the first `m3tal init`. Users should **NOT** set this manually unless performing a token rotation and understanding the implications.
    *   **Default Value**: `change_me_api_token`
    *   **Example Value**: `ZyxWvUtSrQpOnMlKjIhGfEdCbA9876543210` (another long, random string)
    *   **Component Uses It**: CLI, `m3tal-api.service`

*   **`ADMIN_PASSWORD`**
    *   **Description**: The initial default password for the `admin` user on the M3TAL Dashboard. After first login, dashboard user credentials are managed by `m3tal dashpass` and stored in `/docker/users.json`.
    *   **Default Value**: `admin_pass`
    *   **Example Value**: `mySuperSecureDashPass!`
    *   **Component Uses It**: `m3tal-dashboard` (initial setup)

### Network

These variables define networking configurations for M3TAL services.

*   **`NETWORK_NAME`**
    *   **Description**: The name of the Docker network (`proxy` by default as per `m3tal-compose.yml`) that all M3TAL-managed containers join for internal communication. This network is typically created as `external: true` so it can be shared across multiple Docker Compose stacks.
    *   **Default Value**: `m3tal`
    *   **Example Value**: `my_m3tal_network`
    *   **Component Uses It**: Docker Compose (all stacks)

*   **`LOCAL_IP`**
    *   **Description**: The IP address used to resolve `host.docker.internal` from within containers. This is crucial for containers to communicate with services running directly on the Docker host (e.g., the `m3tal-api.service`). Adjust if `host.docker.internal` doesn't resolve to the correct host IP in your environment.
    *   **Default Value**: `127.0.0.1`
    *   **Example Value**: `192.168.1.100`
    *   **Component Uses It**: Docker Compose (`extra_hosts` directive), M3TAL containers connecting to host services

### Storage

These variables define paths for persistent data storage on the host filesystem.

*   **`BASE_STORAGE_PATH`**
    *   **Description**: The absolute base directory on the host filesystem where all M3TAL-related media, configuration, and other data are stored. Subdirectories like `media`, `config`, and `downloads` are typically created under this path.
    *   **Important Note**: While the default in the template is `./data`, in production M3TAL deployments, this variable typically defaults to `/mnt` to leverage dedicated storage mounts.
    *   **Default Value**: `./data`
    *   **Example Value**: `/mnt/m3tal_data`
    *   **Component Uses It**: Docker Compose (all stacks with volume mounts), CLI

*   **`MEDIA_PATH`**
    *   **Description**: The subdirectory under `BASE_STORAGE_PATH` designated for storing user-generated media files and other content.
    *   **Default Value**: `./data/media`
    *   **Example Value**: `/mnt/m3tal_data/media`
    *   **Component Uses It**: Docker Compose (stacks handling media, e.g., dashboard, media servers), CLI

*   **`CONFIG_PATH`**
    *   **Description**: The subdirectory under `BASE_STORAGE_PATH` used for storing M3TAL's internal configuration files and persistent application data (e.g., dashboard state, user settings).
    *   **Default Value**: `./data/config`
    *   **Example Value**: `/mnt/m3tal_data/config`
    *   **Component Uses It**: Docker Compose (dashboard, other configuration-reliant stacks), CLI

*   **`DOWNLOADS_PATH`**
    *   **Description**: The subdirectory under `BASE_STORAGE_PATH` where downloaded content from various services (e.g., download clients) is stored.
    *   **Default Value**: `./data/downloads`
    *   **Example Value**: `/mnt/m3tal_data/downloads`
    *   **Component Uses It**: Docker Compose (stacks managing downloads), CLI

### Traefik

These variables configure the Traefik reverse proxy for routing external traffic to M3TAL services.

*   **`DOMAIN`**
    *   **Description**: The primary domain name for your M3TAL deployment. Setting this variable enables Traefik to automatically create routing rules for `dash.DOMAIN` (for the Dashboard) and `api.DOMAIN` (for the M3TAL API daemon).
    *   **Important Note**: This variable controls Traefik routing rules; setting it enables domain-based access to M3TAL services via Traefik.
    *   **Default Value**: `localhost`
    *   **Example Value**: `example.com`
    *   **Component Uses It**: Traefik (`routing-compose.yml`, dynamic config), `m3tal-dashboard` (when `DASHBOARD_EXPOSE_MODE=traefik`)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description**: The host port that Traefik binds to for its primary HTTP (`web`) entrypoint. This is typically port `80` for standard HTTP traffic.
    *   **Default Value**: `80`
    *   **Example Value**: `8080` (if port 80 is in use by another service on the host)
    *   **Component Uses It**: Traefik (`routing-compose.yml`)

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description**: The host port that Traefik binds to for its HTTPS entrypoint (if HTTPS/TLS is enabled, often requiring additional configuration like Let's Encrypt). This is typically port `443` for standard HTTPS traffic.
    *   **Default Value**: `443`
    *   **Example Value**: `8443`
    *   **Component Uses It**: Traefik (`routing-compose.yml`)

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description**: The internal port within the Traefik container that exposes its own web UI. By default, this internal port is mapped to `127.0.0.1:8081` on the host, making the Traefik dashboard accessible locally.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Component Uses It**: Traefik (`routing-compose.yml`)

### VPN

These variables are placeholders for configuring optional VPN client or server services within the M3TAL ecosystem.

*   **`VPN_USER`**
    *   **Description**: The username for authenticating with an optional VPN component (e.g., WireGuard, OpenVPN client).
    *   **Default Value**: `user`
    *   **Example Value**: `m3tal_vpn_user`
    *   **Component Uses It**: VPN (if a VPN stack is added to `/docker/`)

*   **`VPN_PASSWORD`**
    *   **Description**: The password for authenticating with an optional VPN component.
    *   **Default Value**: `password`
    *   **Example Value**: `secure_vpn_pass`
    *   **Component Uses It**: VPN (if a VPN stack is added to `/docker/`)

### System

These variables control system-wide settings, user/group IDs, and debugging.

*   **`PUID`**
    *   **Description**: The User ID (UID) that Docker containers will use when accessing mounted volumes. Setting this ensures that containers have correct read/write permissions on files owned by a specific user on the host system.
    *   **Default Value**: `1000`
    *   **Example Value**: `1001`
    *   **Component Uses It**: Docker Compose (all stacks with `user` directive)

*   **`PGID`**
    *   **Description**: The Group ID (GID) that Docker containers will use when accessing mounted volumes. Similar to `PUID`, this ensures correct file permissions on shared storage.
    *   **Default Value**: `1000`
    *   **Example Value**: `1001`
    *   **Component Uses It**: Docker Compose (all stacks with `user` directive)

*   **`TZ`**
    *   **Description**: The timezone setting for all M3TAL containers and services. This ensures consistent timestamps and time-based operations across the system.
    *   **Default Value**: `America/Denver`
    *   **Example Value**: `Europe/London`, `Asia/Tokyo`
    *   **Component Uses It**: Docker Compose (all stacks), `m3tal-api.service`

*   **`DEBUG_MODE`**
    *   **Description**: A boolean flag to enable or disable verbose debugging output for M3TAL components. Setting this to `true` can provide more detailed logs for troubleshooting.
    *   **Default Value**: `false`
    *   **Example Value**: `true`
    *   **Component Uses It**: CLI, `m3tal-api.service`, `m3tal-dashboard`, other M3TAL components

*   **`METRICS_ENABLED`**
    *   **Description**: A boolean flag to enable or disable metrics collection for M3TAL components. When `true`, components might expose Prometheus-compatible metrics endpoints.
    *   **Default Value**: `true`
    *   **Example Value**: `false`
    *   **Component Uses It**: CLI, `m3tal-api.service`, `m3tal-dashboard`, other M3TAL components