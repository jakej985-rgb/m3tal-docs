# Environment Variables Reference

As the M3TAL Ecosystem Documentation Architect, I'm here to provide a comprehensive reference for all environment variables used by your M3TAL system. These variables control the behavior of the CLI, the M3TAL API daemon, the Dashboard, Traefik, and all Docker Compose stacks.

All environment variables for your M3TAL instance are stored in the `/etc/m3tal/.env` file. Both the `m3tal` CLI and all Docker Compose stacks load these variables via the `--env-file` option, ensuring a consistent configuration across the entire ecosystem. You can manage these variables using the `m3tal config wizard` or `m3tal config set <KEY> <value>` commands.

---

### Quick Reference Table

| Name                    | Description                                                                  | Default            | Example                      | Category                   |
| :---------------------- | :--------------------------------------------------------------------------- | :----------------- | :--------------------------- | :------------------------- |
| `API_TOKEN`             | Auth token for API daemon. **Auto-generated.**                             | `change_me_api_token` | `secure_api_token_123`       | API Authentication         |
| `ADMIN_PASSWORD`        | Default password for Dashboard's `admin` user.                               | `admin_pass`       | `MyStrongPass123`            | Dashboard Configuration    |
| `BASE_STORAGE_PATH`     | Host path for all M3TAL persistent data.                                     | `./data`           | `/mnt/m3tal`                 | Storage & Paths            |
| `CONFIG_PATH`           | Sub-directory for configuration files within `BASE_STORAGE_PATH`.            | `./data/config`    | `/mnt/m3tal/config`          | Storage & Paths            |
| `DASHBOARD_EXPOSE_MODE` | How the Dashboard is exposed: `local` (direct port) or `traefik` (via Traefik). | `local`            | `traefik`                    | Dashboard Configuration    |
| `DASHBOARD_PORT`        | Port for the M3TAL Dashboard container.                                      | `8082`             | `8000`                       | Dashboard Configuration    |
| `DASHBOARD_SECRET`      | Secret key for Dashboard sessions. **Auto-generated.**                       | `change_me_immediately` | `super_secret_key_abc`       | Dashboard Configuration    |
| `DEBUG_MODE`            | Enables debug logging and features.                                          | `false`            | `true`                       | Core M3TAL Configuration   |
| `DOMAIN`                | Primary domain for Traefik routing (`dash.DOMAIN`, `api.DOMAIN`).            | `localhost`        | `m3tal.example.com`          | Network & Routing          |
| `DOWNLOADS_PATH`        | Sub-directory for downloads within `BASE_STORAGE_PATH`.                      | `./data/downloads` | `/mnt/m3tal/downloads`       | Storage & Paths            |
| `HTTP_PORT`             | Port for the M3TAL API daemon.                                               | `8080`             | `5050`                       | Core M3TAL Configuration   |
| `LOCAL_IP`              | Local IP of the M3TAL host.                                                  | `127.0.0.1`        | `192.168.1.100`              | Core M3TAL Configuration   |
| `LOG_LEVEL`             | Logging verbosity for the API daemon (`debug`, `info`, `warn`, `error`).     | `info`             | `debug`                      | Core M3TAL Configuration   |
| `MEDIA_PATH`            | Sub-directory for media files within `BASE_STORAGE_PATH`.                    | `./data/media`     | `/mnt/m3tal/media`           | Storage & Paths            |
| `METRICS_ENABLED`       | Enables/disables system metrics collection.                                  | `true`             | `false`                      | Core M3TAL Configuration   |
| `NETWORK_NAME`          | Custom Docker network name for user stacks.                                  | `m3tal`            | `my_custom_net`              | Network & Routing          |
| `PGID`                  | Group ID (GID) for container processes.                                      | `1000`             | `999`                        | Container Runtime (System) |
| `PUID`                  | User ID (UID) for container processes.                                       | `1000`             | `999`                        | Container Runtime (System) |
| `STATE_DIR`             | Generic path for container state data.                                       | `./state`          | `/var/lib/my_stack/state`    | Storage & Paths            |
| `TRAEFIK_DASHBOARD_PORT`| Internal port for Traefik's own dashboard.                                   | `8080`             | `8081`                       | Traefik Gateway            |
| `TRAEFIK_WEBHTTPS_PORT` | Port Traefik listens on for HTTPS.                                           | `443`              | `8443`                       | Traefik Gateway            |
| `TRAEFIK_WEB_PORT`      | Port Traefik listens on for HTTP.                                            | `80`               | `8080`                       | Traefik Gateway            |
| `TZ`                    | Timezone for containers.                                                     | `America/Denver`   | `Europe/London`              | Container Runtime (System) |
| `VPN_PASSWORD`          | Password for VPN services.                                                   | `password`         | `securevpnpass`              | VPN Services               |
| `VPN_USER`              | Username for VPN services.                                                   | `user`             | `vpnadmin`                   | VPN Services               |

---

### Detailed Variable Reference

#### Core M3TAL Configuration

These variables control the fundamental behavior of the M3TAL CLI and API daemon.

*   **`HTTP_PORT`**
    *   **Description**: The port on which the M3TAL API daemon (Go binary) listens for incoming requests.
    *   **Default**: `8080`
    *   **Example**: `5050`
    *   **Used by**: M3TAL API daemon (`m3tal-api.service`), Traefik gateway (for `api.DOMAIN` routing).
*   **`LOG_LEVEL`**
    *   **Description**: The logging verbosity for the M3TAL API daemon. Valid options include `debug`, `info`, `warn`, `error`, and `fatal`.
    *   **Default**: `info`
    *   **Example**: `debug`
    *   **Used by**: M3TAL API daemon (`m3tal-api.service`).
*   **`DEBUG_MODE`**
    *   **Description**: Enables or disables debug logging and features across the M3TAL system, including the CLI and API daemon.
    *   **Default**: `false`
    *   **Example**: `true`
    *   **Used by**: M3TAL CLI, M3TAL API daemon.
*   **`METRICS_ENABLED`**
    *   **Description**: Determines whether system metrics collection and exposure are active for the M3TAL API daemon.
    *   **Default**: `true`
    *   **Example**: `false`
    *   **Used by**: M3TAL API daemon.
*   **`LOCAL_IP`**
    *   **Description**: The local IP address of the M3TAL host. This is primarily used for internal routing by some containers to reach host services (e.g., `host.docker.internal` resolution if not natively supported).
    *   **Default**: `127.0.0.1`
    *   **Example**: `192.168.1.100`
    *   **Used by**: Traefik (for routing to host-bound services), M3TAL Dashboard (to reach the API daemon).

#### Dashboard Configuration

These variables specifically pertain to the M3TAL Dashboard container and its exposure.

*   **`DASHBOARD_PORT`**
    *   **Description**: The internal port on which the M3TAL Dashboard container listens. If `DASHBOARD_EXPOSE_MODE` is set to `local`, this port will also be directly exposed on the host.
    *   **Default**: `8082`
    *   **Example**: `9000`
    *   **Used by**: `m3tal-dashboard` container, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`.
*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description**: Controls how the M3TAL Dashboard is made accessible.
        *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly bound to the host, accessible via `http://HOST_IP:DASHBOARD_PORT`. Best for LAN-only or initial setup.
        *   `traefik`: The dashboard is exposed via the Traefik gateway at `http://dash.DOMAIN`. Requires Traefik to be running.
    *   **Default**: `local`
    *   **Example**: `traefik`
    *   **Used by**: `m3tal dash up` command (to select the appropriate compose override), `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`.
*   **`DASHBOARD_SECRET`**
    *   **Description**: A secret key used by the M3TAL Dashboard for session management and securing sensitive operations within the UI.
    *   **Default**: `change_me_immediately`
    *   **Example**: `a_very_long_random_string_here_12345`
    *   **Note**: This variable is **auto-generated on the first `m3tal init`** and stored securely in `/etc/m3tal/.env`. Users should **NOT set this manually unless rotating it** for security purposes.
    *   **Used by**: `m3tal-dashboard` container.
*   **`ADMIN_PASSWORD`**
    *   **Description**: The initial default password for the `admin` user in the M3TAL Dashboard. This is used on first login and can be changed later using the `m3tal dashpass` command.
    *   **Default**: `admin_pass`
    *   **Example**: `MySecureAdminPassword123!`
    *   **Used by**: `m3tal-dashboard` (initial user setup).

#### API Authentication

*   **`API_TOKEN`**
    *   **Description**: An authentication token used to secure communications with the M3TAL API daemon.
    *   **Default**: `change_me_api_token`
    *   **Example**: `another_long_random_string_for_api_auth`
    *   **Note**: This variable is **auto-generated on the first `m3tal init`** and stored securely in `/etc/m3tal/.env`. Users should **NOT set this manually unless rotating it** for security purposes.
    *   **Used by**: M3TAL API daemon (for authenticating requests), M3TAL CLI (when interacting with the API).

#### Network & Routing

These variables define networking configurations, especially for the Traefik gateway.

*   **`DOMAIN`**
    *   **Description**: The primary domain name for your M3TAL services. Setting this variable enables Traefik routing rules, allowing access to services like the M3TAL Dashboard (`dash.DOMAIN`) and M3TAL API (`api.DOMAIN`) via domain names instead of IP addresses and ports.
    *   **Default**: `localhost`
    *   **Example**: `example.com`
    *   **Used by**: Traefik gateway (`routing-compose.yml`, `m3tal-compose.traefik.yml`).
*   **`NETWORK_NAME`**
    *   **Description**: A variable to define a custom Docker network name, if desired, for user-defined compose stacks. M3TAL's core components (Dashboard, Traefik) utilize an external Docker network named `proxy` for inter-service communication by default.
    *   **Default**: `m3tal`
    *   **Example**: `my_custom_net`
    *   **Used by**: User-defined Docker Compose stacks.

#### Storage & Paths

These variables define the host filesystem paths where M3TAL and other containers store their persistent data.

*   **`BASE_STORAGE_PATH`**
    *   **Description**: The base directory on the host filesystem where all M3TAL persistent data, including media, configurations, and downloads, is stored.
    *   **Default**: `./data`
    *   **Example**: `/mnt/m3tal-data`
    *   **Note**: In production deployments, this typically defaults to `/mnt` (e.g., `/mnt/m3tal-data`) for better data isolation and management, rather than `./data` which is common in development or local setups.
    *   **Used by**: `m3tal-dashboard` container (as a volume mount base), other user-defined compose stacks.
*   **`MEDIA_PATH`**
    *   **Description**: A sub-directory within `BASE_STORAGE_PATH` designated for storing media files.
    *   **Default**: `./data/media`
    *   **Example**: `/mnt/media`
    *   **Used by**: User-defined media management stacks (e.g., Plex, Jellyfin, Sonarr).
*   **`CONFIG_PATH`**
    *   **Description**: A sub-directory within `BASE_STORAGE_PATH` used for storing configuration files, including the Dashboard's `users.json` credential store.
    *   **Default**: `./data/config`
    *   **Example**: `/opt/m3tal/config`
    *   **Used by**: `m3tal-dashboard` container (for volume mounts), other user-defined configuration directories.
*   **`DOWNLOADS_PATH`**
    *   **Description**: A sub-directory within `BASE_STORAGE_PATH` dedicated to storing downloaded files.
    *   **Default**: `./data/downloads`
    *   **Example**: `/mnt/downloads`
    *   **Used by**: User-defined download clients (e.g., qBittorrent, Transmission).
*   **`STATE_DIR`**
    *   **Description**: A generic path variable intended for defining where state data might be stored for various components. This variable can be leveraged by user-defined compose stacks for their state directories.
    *   **Default**: `./state`
    *   **Example**: `/var/lib/my_stack/state`
    *   **Note**: The M3TAL API daemon uses a fixed path (`/var/lib/m3tal/state.db`) for its state database. The M3TAL Dashboard's internal state directory (`/docker/state`) is mounted from the host at `${CONFIG_PATH}/m3tal/state`.

#### Container Runtime (System)

These variables configure how Docker containers run, affecting user and group IDs and timezones.

*   **`PUID`**
    *   **Description**: The user ID (UID) that Docker containers should use to run processes. Setting this ensures correct file permissions on mounted volumes, preventing permission denied errors.
    *   **Default**: `1000`
    *   **Example**: `999`
    *   **Used by**: `m3tal-dashboard` container, other user-defined containers.
*   **`PGID`**
    *   **Description**: The group ID (GID) that Docker containers should use to run processes. Similar to `PUID`, this ensures correct file permissions on mounted volumes.
    *   **Default**: `1000`
    *   **Example**: `999`
    *   **Used by**: `m3tal-dashboard` container, other user-defined containers.
*   **`TZ`**
    *   **Description**: The timezone for containers, which impacts logging timestamps, scheduled tasks, and any time-sensitive operations within the containers.
    *   **Default**: `America/Denver`
    *   **Example**: `Europe/London`
    *   **Used by**: `m3tal-dashboard` container, other user-defined containers.

#### Traefik Gateway

Variables specific to the Traefik reverse proxy.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description**: The host port on which Traefik listens for unencrypted HTTP traffic (entrypoint `web`).
    *   **Default**: `80`
    *   **Example**: `8080`
    *   **Used by**: `traefik` container (`routing-compose.yml`).
*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description**: The host port on which Traefik listens for encrypted HTTPS traffic (entrypoint `websecure`, if enabled).
    *   **Default**: `443`
    *   **Example**: `8443`
    *   **Used by**: `traefik` container (`routing-compose.yml`).
*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description**: The internal port Traefik uses for its own administration dashboard. This dashboard is exposed on the host at `127.0.0.1:8081` by default, regardless of this variable's value in `.env`.
    *   **Default**: `8080`
    *   **Example**: `8081`
    *   **Used by**: `traefik` container (`routing-compose.yml`).

#### VPN Services

These variables are placeholders for configuring optional VPN services if deployed as part of your M3TAL ecosystem.

*   **`VPN_USER`**
    *   **Description**: The username for VPN services, if a VPN stack is deployed.
    *   **Default**: `user`
    *   **Example**: `vpnuser`
    *   **Used by**: VPN stack (if deployed by the user).
*   **`VPN_PASSWORD`**
    *   **Description**: The password for VPN services, if a VPN stack is deployed.
    *   **Default**: `password`
    *   **Example**: `securevpnpass`
    *   **Used by**: VPN stack (if deployed by the user).