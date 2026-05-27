As DocSmith, the M3TAL Ecosystem Documentation Architect, here is your comprehensive reference for M3TAL environment variables.

---

# Environment Variables Reference

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. Both the `m3tal` CLI binary and all Docker Compose stacks (started with `m3tal up` or `m3tal dash up`) load these variables using the `--env-file` flag, ensuring consistent configuration across the entire ecosystem.

You can manage these variables using the `m3tal config wizard` or `m3tal config set <KEY> <value>` commands.

## Quick Reference

| Name                    | Description                                                                                                                                                                                                       | Default Value             |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------ |
| `DASHBOARD_PORT`        | The port on which the M3TAL Dashboard container listens.                                                                                                                                                          | `8082`                    |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL Dashboard is exposed: `local` for direct port binding, `traefik` for routing via Traefik using `dash.DOMAIN`.                                                                              | `local`                   |
| `HTTP_PORT`             | The port on which the M3TAL API daemon (Go binary) listens.                                                                                                                                                       | `8080`                    |
| `STATE_DIR`             | Defines the directory where the M3TAL API daemon stores its SQLite state database (`state.db`).                                                                                                                   | `./state`                 |
| `DASHBOARD_SECRET`      | A secret key used by the M3TAL Dashboard for session management and encryption. Auto-generated on `m3tal init`.                                                                                                   | `change_me_immediately`   |
| `API_TOKEN`             | An authentication token used to secure access to the M3TAL API daemon. Auto-generated on `m3tal init`.                                                                                                            | `change_me_api_token`     |
| `ADMIN_PASSWORD`        | The default password for the `admin` user on the M3TAL Dashboard.                                                                                                                                                 | `admin_pass`              |
| `NETWORK_NAME`          | The name of the Docker network used by M3TAL components and user stacks.                                                                                                                                          | `m3tal`                   |
| `LOCAL_IP`              | The IP address that `host.docker.internal` resolves to within containers, allowing them to access services on the Docker host.                                                                                     | `127.0.0.1`               |
| `DOMAIN`                | The base domain name for M3TAL services when using Traefik, enabling `dash.DOMAIN` and `api.DOMAIN` routes.                                                                                                       | `localhost`               |
| `VPN_USER`              | Username for the optional VPN service.                                                                                                                                                                            | `user`                    |
| `VPN_PASSWORD`          | Password for the optional VPN service.                                                                                                                                                                            | `password`                |
| `BASE_STORAGE_PATH`     | The root directory on the host where M3TAL stores all persistent data. Defaults to `/mnt` in production.                                                                                                          | `./data`                  |
| `MEDIA_PATH`            | The specific subdirectory for media files, relative to `BASE_STORAGE_PATH`.                                                                                                                                     | `./data/media`            |
| `CONFIG_PATH`           | The root directory on the host for M3TAL configuration files, including the dashboard's `users.json`.                                                                                                           | `./data/config`           |
| `DOWNLOADS_PATH`        | The specific subdirectory for downloaded files, relative to `BASE_STORAGE_PATH`.                                                                                                                                | `./data/downloads`        |
| `PUID`                  | The User ID (UID) used by M3TAL containers for file permissions on mounted volumes.                                                                                                                               | `1000`                    |
| `PGID`                  | The Group ID (GID) used by M3TAL containers for file permissions on mounted volumes.                                                                                                                              | `1000`                    |
| `TZ`                    | The timezone for M3TAL containers.                                                                                                                                                                                | `America/Denver`          |
| `LOG_LEVEL`             | Sets the logging verbosity for the M3TAL API daemon.                                                                                                                                                              | `info`                    |
| `TRAEFIK_WEB_PORT`      | The host port Traefik binds to for insecure HTTP traffic.                                                                                                                                                         | `80`                      |
| `TRAEFIK_WEBHTTPS_PORT` | The host port Traefik binds to for secure HTTPS traffic.                                                                                                                                                          | `443`                     |
| `TRAEFIK_DASHBOARD_PORT`| The internal port Traefik uses for its own management dashboard, typically bound to `127.0.0.1` on the host.                                                                                                      | `8080`                    |
| `DEBUG_MODE`            | Enables or disables debug-level logging and features across M3TAL components.                                                                                                                                     | `false`                   |
| `METRICS_ENABLED`       | Enables or disables the collection and exposure of operational metrics from M3TAL components.                                                                                                                     | `true`                    |

---

## Variable Reference

### Core Variables

These variables control fundamental aspects of the M3TAL system's operation and accessibility.

*   **`DASHBOARD_PORT`**
    *   **Description**: The port on which the M3TAL Dashboard container listens internally. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host.
    *   **Default Value**: `8082`
    *   **Example Value**: `8082`
    *   **Used By**: `m3tal-dashboard` container, `m3tal-compose.local.yml`, CLI (for status messages).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description**: Determines how the M3TAL Dashboard is made accessible.
        *   `local`: The dashboard is directly accessible via `http://HOST_IP:DASHBOARD_PORT`. No Traefik required.
        *   `traefik`: The dashboard is routed via Traefik, accessible at `http://dash.DOMAIN`. Requires Traefik to be running.
    *   **Default Value**: `local`
    *   **Example Value**: `traefik`
    *   **Used By**: `m3tal dash up` command (CLI), `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`.

*   **`HTTP_PORT`**
    *   **Description**: The port on which the M3TAL API daemon (Go binary) listens. This service runs directly on the host and is accessed internally by containers (e.g., dashboard) via `host.docker.internal` or externally via Traefik.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Used By**: `m3tal-api.service` (API daemon), Traefik (for routing `api.DOMAIN` to the API).

*   **`STATE_DIR`**
    *   **Description**: Defines the host directory where the M3TAL API daemon stores its persistent SQLite state database (`state.db`).
    *   **Default Value**: `./state`
    *   **Example Value**: `/var/lib/m3tal/state` (common production path)
    *   **Used By**: `m3tal-api.service`.

### Authentication Variables

These variables are critical for securing access to M3TAL services.

*   **`DASHBOARD_SECRET`**
    *   **Description**: A unique secret key used by the M3TAL Dashboard for secure session management and internal data encryption. **This variable is automatically generated on the first `m3tal init` run**. Users should generally not set this manually unless rotating the key for security reasons.
    *   **Default Value**: `change_me_immediately`
    *   **Example Value**: `a_strong_random_string_of_at_least_32_characters`
    *   **Used By**: `m3tal-dashboard` container, `m3tal init` (CLI generates).

*   **`API_TOKEN`**
    *   **Description**: An authentication token used to secure access to the M3TAL API daemon. This token must be included in requests to the API for authorization. **This variable is automatically generated on the first `m3tal init` run**. Users should generally not set this manually unless rotating the token for security reasons.
    *   **Default Value**: `change_me_api_token`
    *   **Example Value**: `a_long_cryptographically_secure_token_for_api_access`
    *   **Used By**: `m3tal-api.service`, CLI (for API calls), `m3tal init` (CLI generates).

*   **`ADMIN_PASSWORD`**
    *   **Description**: The initial password for the default `admin` user on the M3TAL Dashboard. Users are prompted to change this during `m3tal init` or can manage it with `m3tal dashpass`.
    *   **Default Value**: `admin_pass`
    *   **Example Value**: `MySuperSecureAdminPassword123!`
    *   **Used By**: `m3tal dashpass` (CLI for management), `m3tal-dashboard` container (via `users.json` file).

### Network Variables

These variables configure how M3TAL components communicate within Docker and with the host.

*   **`NETWORK_NAME`**
    *   **Description**: The name of the Docker bridge network used by all M3TAL-managed containers. This network allows containers to communicate with each other and with Traefik.
    *   **Default Value**: `m3tal`
    *   **Example Value**: `m3tal-proxy-network`
    *   **Used By**: All Docker containers (e.g., `m3tal-dashboard`, `traefik`, `cloudflared`, user stacks), Docker Compose (`m3tal up`).

*   **`LOCAL_IP`**
    *   **Description**: The IP address that `host.docker.internal` resolves to within containers, allowing them to access services running directly on the Docker host (e.g., the M3TAL API daemon).
    *   **Default Value**: `127.0.0.1`
    *   **Example Value**: `192.168.1.100` (Your host's LAN IP)
    *   **Used By**: Traefik (for routing to `host.docker.internal:8080`), `m3tal-dashboard` (for API access via `host.docker.internal`).

### Storage Variables

These variables define the host paths for persistent data storage, which are then mounted into containers.

*   **`BASE_STORAGE_PATH`**
    *   **Description**: The primary root directory on the host where M3TAL stores all persistent data, including media, configuration, and downloads. **In production deployments, this defaults to `/mnt` to leverage dedicated storage mounts, rather than `./data` (which is suitable for local development/testing).**
    *   **Default Value**: `./data`
    *   **Example Value**: `/mnt/m3tal_data`
    *   **Used By**: `m3tal-dashboard` container (as volume mount for `/mnt`), user stacks (for their data volumes).

*   **`MEDIA_PATH`**
    *   **Description**: The specific subdirectory on the host for storing media files, relative to `BASE_STORAGE_PATH`.
    *   **Default Value**: `./data/media`
    *   **Example Value**: `/mnt/media` (if `BASE_STORAGE_PATH` is `/mnt`)
    *   **Used By**: User stacks (e.g., media servers), potentially CLI.

*   **`CONFIG_PATH`**
    *   **Description**: The root directory on the host where M3TAL stores configuration files specific to its services, such as the dashboard's `users.json`. The full path for dashboard configuration is typically `${CONFIG_PATH}/m3tal/state/config`.
    *   **Default Value**: `./data/config`
    *   **Example Value**: `/var/lib/m3tal/config` (common production path)
    *   **Used By**: `m3tal-dashboard` container (for `users.json` volume mount), `m3tal dashpass` (CLI for user management).

*   **`DOWNLOADS_PATH`**
    *   **Description**: The specific subdirectory on the host for storing downloaded files, relative to `BASE_STORAGE_PATH`.
    *   **Default Value**: `./data/downloads`
    *   **Example Value**: `/mnt/downloads`
    *   **Used By**: User stacks (e.g., download clients), potentially CLI.

### Traefik Variables

These variables control the behavior and configuration of the Traefik reverse proxy.

*   **`DOMAIN`**
    *   **Description**: The base domain name for M3TAL services when Traefik is enabled. Setting this variable activates Traefik routing rules, allowing access to the dashboard at `dash.DOMAIN` and the API at `api.DOMAIN`.
    *   **Default Value**: `localhost`
    *   **Example Value**: `myhomelab.com`
    *   **Used By**: `traefik` container, `m3tal-compose.traefik.yml`, Traefik dynamic configuration (`dynamic/api.yml`), CLI (for status messages).

*   **`TRAEFIK_WEB_PORT`**
    *   **Description**: The host port Traefik binds to for handling incoming insecure HTTP (web) traffic.
    *   **Default Value**: `80`
    *   **Example Value**: `8080` (if port 80 is already in use)
    *   **Used By**: `traefik` container, Docker Compose (`routing-compose.yml`).

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description**: The host port Traefik binds to for handling incoming secure HTTPS (websecure) traffic.
    *   **Default Value**: `443`
    *   **Example Value**: `8443`
    *   **Used By**: `traefik` container, Docker Compose (`routing-compose.yml`).

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description**: The internal port Traefik uses for its own management dashboard. This port is typically exposed on the host's loopback interface (`127.0.0.1`) for local administrative access.
    *   **Default Value**: `8080`
    *   **Example Value**: `8081` (as seen in `routing-compose.yml` for host binding)
    *   **Used By**: `traefik` container.

### VPN Variables

These variables are used for configuring an optional VPN service if it is integrated with M3TAL.

*   **`VPN_USER`**
    *   **Description**: The username for authenticating with the optional VPN service.
    *   **Default Value**: `user`
    *   **Example Value**: `johndoe`
    *   **Used By**: VPN service (if deployed).

*   **`VPN_PASSWORD`**
    *   **Description**: The password for authenticating with the optional VPN service.
    *   **Default Value**: `password`
    *   **Example Value**: `MyStrongVpnPass123`
    *   **Used By**: VPN service (if deployed).

### System Variables

These variables control general system behavior, logging, and container execution.

*   **`PUID`**
    *   **Description**: The User ID (UID) that M3TAL containers will use when accessing mounted volumes on the host. This ensures correct file ownership and permissions, preventing permission errors. It should ideally match the UID of a non-root user on your host system.
    *   **Default Value**: `1000`
    *   **Example Value**: `1001`
    *   **Used By**: `m3tal-dashboard` container, user stacks.

*   **`PGID`**
    *   **Description**: The Group ID (GID) that M3TAL containers will use when accessing mounted volumes on the host. This complements `PUID` to ensure correct group ownership and permissions. It should ideally match the GID of a group on your host system.
    *   **Default Value**: `1000`
    *   **Example Value**: `1001`
    *   **Used By**: `m3tal-dashboard` container, user stacks.

*   **`TZ`**
    *   **Description**: Sets the timezone for M3TAL containers. This affects timestamps in logs, cron jobs, and any time-sensitive operations within the containers.
    *   **Default Value**: `America/Denver`
    *   **Example Value**: `America/New_York`
    *   **Used By**: `m3tal-dashboard` container, user stacks.

*   **`LOG_LEVEL`**
    *   **Description**: Sets the logging verbosity for the M3TAL API daemon. Available levels typically include `debug`, `info`, `warn`, `error`, `fatal`.
    *   **Default Value**: `info`
    *   **Example Value**: `debug`
    *   **Used By**: `m3tal-api.service`.

*   **`DEBUG_MODE`**
    *   **Description**: A flag to enable or disable debug-level logging and potentially other debug features across M3TAL components (CLI, API, Dashboard). Set to `true` for detailed diagnostics.
    *   **Default Value**: `false`
    *   **Example Value**: `true`
    *   **Used By**: CLI, `m3tal-api.service`, `m3tal-dashboard`.

*   **`METRICS_ENABLED`**
    *   **Description**: A flag to enable or disable the collection and exposure of operational metrics from M3TAL components, useful for monitoring and observability tools.
    *   **Default Value**: `true`
    *   **Example Value**: `false`
    *   **Used By**: `m3tal-api.service`.