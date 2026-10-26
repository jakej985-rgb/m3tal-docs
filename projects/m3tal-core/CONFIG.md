# Environment Variables Reference

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` and is utilized by both the `m3tal` CLI and all Docker Compose stacks via the `--env-file` option.

It is recommended to use `m3tal config wizard` or `m3tal config set KEY value` to modify these variables, ensuring proper synchronization across the system.

---

## Quick Reference

| Variable                  | Description                                                                                                   | Default Value         | Component(s)                                   |
| :------------------------ | :------------------------------------------------------------------------------------------------------------ | :-------------------- | :--------------------------------------------- |
| `API_PORT`                | Internal port for API daemon communication from Docker containers.                                            | `5050`                | `m3tal-dashboard`                              |
| `DASHBOARD_PORT`          | Port the M3TAL Dashboard container listens on internally.                                                     | `8082`                | `m3tal-dashboard`, `CLI`, `traefik`            |
| `DASHBOARD_EXPOSE_MODE`   | Controls how the M3TAL Dashboard is exposed (`local` or `traefik`).                                           | `local`               | `CLI`, `m3tal-dashboard`                       |
| `HTTP_PORT`               | Host port for the M3TAL API daemon and Traefik routing to the API.                                            | `8080`                | `m3tal-api.service`, `traefik`                 |
| `STATE_DIR`               | Internal directory path used by containers for state or temporary files.                                      | `./state`             | `m3tal-dashboard`                              |
| `LOG_LEVEL`               | Sets the logging verbosity for M3TAL components.                                                              | `info`                | `CLI`, `m3tal-api.service`, `m3tal-dashboard`  |
| `DASHBOARD_SECRET`        | Secret key for Dashboard session management. **Auto-generated.**                                              | `change_me_immediately` | `m3tal-dashboard`                              |
| `API_TOKEN`               | Authentication token for M3TAL API access. **Auto-generated.**                                                | `change_me_api_token` | `CLI`, `m3tal-api.service`, `m3tal-dashboard`  |
| `ADMIN_PASSWORD`          | Initial password for the dashboard's admin user.                                                              | `admin_pass`          | `m3tal-dashboard`                              |
| `NETWORK_NAME`            | Name of the Docker network for M3TAL services.                                                                | `m3tal`               | All Docker containers                          |
| `LOCAL_IP`                | Local IP address of the M3TAL host.                                                                           | `127.0.0.1`           | `traefik` (optional), user stacks              |
| `DOMAIN`                  | Base domain name for M3TAL services when exposed via Traefik.                                                 | `localhost`           | `CLI`, `traefik`, `m3tal-dashboard`            |
| `VPN_USER`                | Username for the optional M3TAL VPN service.                                                                  | `user`                | VPN container (if deployed)                    |
| `VPN_PASSWORD`            | Password for the optional M3TAL VPN service.                                                                  | `password`            | VPN container (if deployed)                    |
| `BASE_STORAGE_PATH`       | Root directory for all M3TAL persistent data on the host.                                                     | `./data`              | `CLI`, `m3tal-api.service`, `m3tal-dashboard`  |
| `MEDIA_PATH`              | Subdirectory for media files.                                                                                 | `./data/media`        | `m3tal-dashboard`, user stacks                 |
| `CONFIG_PATH`             | Subdirectory for application configuration and persistent state.                                              | `./data/config`       | `CLI`, `m3tal-dashboard`                       |
| `DOWNLOADS_PATH`          | Subdirectory for downloaded content.                                                                          | `./data/downloads`    | User stacks                                    |
| `PUID`                    | User ID (UID) for Docker containers.                                                                          | `1000`                | All Docker containers                          |
| `PGID`                    | Group ID (GID) for Docker containers.                                                                         | `1000`                | All Docker containers                          |
| `TZ`                      | Timezone setting for M3TAL components and containers.                                                         | `America/Denver`      | All M3TAL components and Docker containers     |
| `TRAEFIK_WEB_PORT`        | Host port for Traefik's HTTP entry point.                                                                     | `80`                  | `traefik`                                      |
| `TRAEFIK_WEBHTTPS_PORT`   | Host port for Traefik's HTTPS entry point.                                                                    | `443`                 | `traefik`                                      |
| `TRAEFIK_DASHBOARD_PORT`  | Internal container port for Traefik's own dashboard.                                                          | `8080`                | `traefik`                                      |
| `DEBUG_MODE`              | Enables debug-level logging and features.                                                                     | `false`               | `CLI`, `m3tal-api.service`, `m3tal-dashboard`  |
| `METRICS_ENABLED`         | Enables or disables exposing application metrics.                                                             | `true`                | `CLI`, `m3tal-api.service`, `m3tal-dashboard`  |

---

## Detailed Variable Reference

### Core

#### `API_PORT`
*   **Description:** The internal port for the M3TAL API daemon when it's addressed by other Docker containers via `host.docker.internal` (e.g., by the dashboard). Note that the API daemon (`m3tal-api.service`) listens on `HTTP_PORT` (default `8080`) on the host.
*   **Default Value:** `5050`
*   **Example Value:** `5050`
*   **Component(s) Used:** `m3tal-dashboard` (via `GO_API_URL`)

#### `DASHBOARD_PORT`
*   **Description:** The internal port on which the `m3tal-dashboard` container listens. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host.
*   **Default Value:** `8082`
*   **Example Value:** `8082`
*   **Component(s) Used:** `m3tal-dashboard`, `CLI` (for `m3tal dash up` local mode), `traefik` (when `DASHBOARD_EXPOSE_MODE=traefik`)

#### `DASHBOARD_EXPOSE_MODE`
*   **Description:** Controls how the M3TAL Dashboard is made accessible.
    *   `local`: The dashboard is exposed directly on `http://HOST_IP:DASHBOARD_PORT`. No Traefik required.
    *   `traefik`: The dashboard is routed via Traefik to `http://dash.DOMAIN`. Requires Traefik to be running.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Component(s) Used:** `CLI` (specifically `m3tal dash up` to select compose overrides), `m3tal-dashboard` (indirectly via selected compose file)

#### `HTTP_PORT`
*   **Description:** The port on the host where the `m3tal-api.service` daemon listens for incoming connections. This port is also used by Traefik to route `api.DOMAIN` requests to the API daemon via `host.docker.internal`.
*   **Default Value:** `8080`
*   **Example Value:** `80` (if 8080 is already in use)
*   **Component(s) Used:** `m3tal-api.service` (implied), `traefik` (for routing to API daemon)

#### `STATE_DIR`
*   **Description:** Controls the internal directory path used by some containers for state or temporary files.
    *   Note: The M3TAL API daemon's SQLite state database is located at `/var/lib/m3tal/state.db` on the host.
    *   The Dashboard container mounts its persistent state from `${CONFIG_PATH}/m3tal/state` on the host to an internal `/docker/state` path.
*   **Default Value:** `./state`
*   **Example Value:** `/docker/state` (internal container path)
*   **Component(s) Used:** `m3tal-dashboard` (influences internal path, though `/docker/state` is specifically set within the container's environment)

#### `LOG_LEVEL`
*   **Description:** Sets the logging verbosity for M3TAL components. Common values include `debug`, `info`, `warn`, `error`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Component(s) Used:** `CLI`, `m3tal-api.service`, `m3tal-dashboard`, user stacks

### Auth

#### `DASHBOARD_SECRET`
*   **Description:** A secret key used by the M3TAL Dashboard for secure session management and encryption.
*   **IMPORTANT:** This variable is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a secret rotation, which requires careful planning.
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_long_random_string_of_characters_for_security`
*   **Component(s) Used:** `m3tal-dashboard`

#### `API_TOKEN`
*   **Description:** An authentication token used for programmatic access to the M3TAL API. This token authenticates requests from the CLI and Dashboard to the API daemon.
*   **IMPORTANT:** This variable is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a token rotation.
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `another_long_random_string_of_characters_for_api_auth`
*   **Component(s) Used:** `CLI`, `m3tal-api.service` (validates tokens), `m3tal-dashboard`

#### `ADMIN_PASSWORD`
*   **Description:** The initial password for the default 'admin' user within the M3TAL Dashboard. After initialization, this is managed via the `m3tal dashpass` command.
*   **Default Value:** `admin_pass`
*   **Example Value:** `MySecureDashboardPassword123`
*   **Component(s) Used:** `m3tal-dashboard` (reads from `users.json`, which is populated using this value)

### Network

#### `NETWORK_NAME`
*   **Description:** The name of the Docker network that all M3TAL core services and user-defined Docker Compose stacks will connect to. This enables seamless inter-container communication.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal_prod_network`
*   **Component(s) Used:** All Docker containers (`m3tal-dashboard`, `traefik`, `cloudflared`, any user stack)

#### `LOCAL_IP`
*   **Description:** Specifies the local IP address of the M3TAL host. Can be used for specific network bindings or internal routing configurations.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `192.168.1.100`
*   **Component(s) Used:** `traefik` (optional), user stacks (for host binding)

#### `DOMAIN`
*   **Description:** The base domain name that M3TAL services will use when exposed through Traefik. Setting this enables Traefik to route traffic for subdomains like `dash.DOMAIN` (to the dashboard) and `api.DOMAIN` (to the API daemon).
*   **Default Value:** `localhost`
*   **Example Value:** `example.com`
*   **Component(s) Used:** `CLI` (when generating Traefik config), `traefik`, `m3tal-dashboard` (when `DASHBOARD_EXPOSE_MODE=traefik`)

### Storage

#### `BASE_STORAGE_PATH`
*   **Description:** The root directory on the host filesystem where M3TAL stores all persistent application data, including media, configurations, and downloads.
*   **IMPORTANT:** In production deployments, this variable typically defaults to `/mnt`, not `./data` as in development templates.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal-data` or `/mnt`
*   **Component(s) Used:** `CLI`, `m3tal-api.service`, `m3tal-dashboard`, user stacks (for volume mounts)

#### `MEDIA_PATH`
*   **Description:** A subdirectory within `BASE_STORAGE_PATH` designated for storing user media files.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal-data/media`
*   **Component(s) Used:** `m3tal-dashboard` (volume mount), user stacks

#### `CONFIG_PATH`
*   **Description:** A subdirectory within `BASE_STORAGE_PATH` used for application-specific configuration files and persistent state. For the dashboard, it forms the host path `${CONFIG_PATH}/m3tal/state`, which is mounted into the container.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal-data/config`
*   **Component(s) Used:** `CLI`, `m3tal-dashboard` (volume mount)

#### `DOWNLOADS_PATH`
*   **Description:** A subdirectory within `BASE_STORAGE_PATH` intended for downloaded content.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal-data/downloads`
*   **Component(s) Used:** User stacks (e.g., download clients)

### Traefik

#### `TRAEFIK_WEB_PORT`
*   **Description:** The host port on which Traefik listens for incoming HTTP traffic (entrypoint `web`).
*   **Default Value:** `80`
*   **Example Value:** `8080` (if port 80 is in use by another service)
*   **Component(s) Used:** `traefik`

#### `TRAEFIK_WEBHTTPS_PORT`
*   **Description:** The host port on which Traefik listens for incoming HTTPS traffic (entrypoint `websecure`, if configured).
*   **Default Value:** `443`
*   **Example Value:** `8443` (if port 443 is in use)
*   **Component(s) Used:** `traefik`

#### `TRAEFIK_DASHBOARD_PORT`
*   **Description:** The internal container port that Traefik uses for its own administration dashboard. By default, this is mapped to `127.0.0.1:8081` on the host.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Component(s) Used:** `traefik`

### VPN

#### `VPN_USER`
*   **Description:** The username for an optional VPN service (e.g., WireGuard) managed by M3TAL.
*   **Default Value:** `user`
*   **Example Value:** `john_doe`
*   **Component(s) Used:** VPN container (if deployed)

#### `VPN_PASSWORD`
*   **Description:** The password for an optional VPN service (e.g., WireGuard) managed by M3TAL.
*   **Default Value:** `password`
*   **Example Value:** `MySecureVPNPassword123`
*   **Component(s) Used:** VPN container (if deployed)

### System

#### `PUID`
*   **Description:** The User ID (UID) that Docker containers managed by M3TAL will run as. This ensures correct file permissions when containers interact with host-mounted volumes.
*   **Default Value:** `1000`
*   **Example Value:** `1001` (if your primary user has a different UID)
*   **Component(s) Used:** `m3tal-dashboard`, all user Docker containers

#### `PGID`
*   **Description:** The Group ID (GID) that Docker containers managed by M3TAL will run as. This ensures correct file permissions when containers interact with host-mounted volumes.
*   **Default Value:** `1000`
*   **Example Value:** `1001` (if your primary user has a different GID)
*   **Component(s) Used:** `m3tal-dashboard`, all user Docker containers

#### `TZ`
*   **Description:** Sets the timezone for M3TAL components and Docker containers, ensuring accurate timestamps in logs and applications. Uses standard IANA timezone names (e.g., `America/New_York`, `Europe/London`).
*   **Default Value:** `America/Denver`
*   **Example Value:** `Europe/London`
*   **Component(s) Used:** `CLI`, `m3tal-api.service`, `m3tal-dashboard`, all Docker containers

#### `DEBUG_MODE`
*   **Description:** When set to `true`, enables debug-level logging and potentially other debug features across M3TAL components. Useful for troubleshooting.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Component(s) Used:** `CLI`, `m3tal-api.service`, `m3tal-dashboard`, user stacks

#### `METRICS_ENABLED`
*   **Description:** When set to `true`, enables exposing application metrics (e.g., in Prometheus format) for monitoring.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Component(s) Used:** `CLI`, `m3tal-api.service`, `m3tal-dashboard`, user stacks