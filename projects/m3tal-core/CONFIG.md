# Environment Variable Reference

All M3TAL environment variables are read from the primary configuration file, `/etc/m3tal/.env`. This file is used by both the M3TAL CLI binary and all Docker Compose stacks (via the `--env-file` flag), ensuring consistent configuration across your M3TAL system.

The `m3tal config wizard` command provides an interactive way to manage these variables, while `m3tal config set KEY value` allows direct modification.

---

## Quick Reference Table

| Name                   | Default                  | Description                                                                                                                                                                                                | Used By                                                                         |
| :--------------------- | :----------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------ |
| `API_PORT`             | `5050`                   | Port the M3TAL API daemon is expected to be accessible on by internal components like the Dashboard.                                                                                                        | `m3tal-dashboard`                                                               |
| `DASHBOARD_PORT`       | `8082`                   | Port for the M3TAL Dashboard container.                                                                                                                                                                    | `m3tal-dashboard`, `CLI binary`                                                 |
| `DASHBOARD_EXPOSE_MODE`| `local`                  | Controls how the M3TAL Dashboard is exposed (direct port or via Traefik).                                                                                                                                  | `CLI binary`, `m3tal-dashboard`                                                 |
| `HTTP_PORT`            | `8080`                   | Port the primary M3TAL API daemon listens on for incoming requests (e.g., from Traefik).                                                                                                                   | `m3tal-api.service`, `Traefik gateway`                                          |
| `STATE_DIR`            | `./state`                | Directory where the M3TAL API stores its SQLite state database and other runtime files.                                                                                                                    | `m3tal-api.service`, `m3tal-dashboard`                                          |
| `LOG_LEVEL`            | `info`                   | Minimum logging level for the M3TAL API daemon.                                                                                                                                                            | `m3tal-api.service`                                                             |
| `DASHBOARD_SECRET`     | `change_me_immediately`  | Secret key for M3TAL Dashboard session management and encryption.                                                                                                                                          | `m3tal-dashboard`                                                               |
| `API_TOKEN`            | `change_me_api_token`    | Authentication token for programmatic access to the M3TAL API.                                                                                                                                             | `CLI binary`, `m3tal-dashboard`                                                 |
| `ADMIN_PASSWORD`       | `admin_pass`             | Initial password for the default 'admin' user of the M3TAL Dashboard.                                                                                                                                      | `m3tal-dashboard`                                                               |
| `NETWORK_NAME`         | `m3tal`                  | Name of the Docker network used by M3TAL's control plane services and user stacks.                                                                                                                         | All M3TAL compose stacks                                                        |
| `LOCAL_IP`             | `127.0.0.1`              | IP address for host-level services, often `host.docker.internal` resolution.                                                                                                                               | `Traefik gateway`, user stacks                                                  |
| `DOMAIN`               | `localhost`              | Base domain name for M3TAL services exposed via Traefik (e.g., `dash.DOMAIN`, `api.DOMAIN`).                                                                                                              | `Traefik gateway`, `m3tal-dashboard`                                            |
| `VPN_USER`             | `user`                   | Username for VPN client authentication.                                                                                                                                                                    | VPN client container                                                            |
| `VPN_PASSWORD`         | `password`               | Password for VPN client authentication.                                                                                                                                                                    | VPN client container                                                            |
| `BASE_STORAGE_PATH`    | `./data`                 | The root directory for all M3TAL-managed media and configuration data.                                                                                                                                     | `m3tal-dashboard`, `CLI binary`, user stacks                                    |
| `MEDIA_PATH`           | `./data/media`           | Path within `BASE_STORAGE_PATH` designated for media files.                                                                                                                                                | `m3tal-dashboard`, user stacks                                                  |
| `CONFIG_PATH`          | `./data/config`          | Path within `BASE_STORAGE_PATH` for configuration files (e.g., `users.json`).                                                                                                                             | `m3tal-dashboard`, user stacks                                                  |
| `DOWNLOADS_PATH`       | `./data/downloads`       | Path within `BASE_STORAGE_PATH` for downloaded content.                                                                                                                                                    | User stacks                                                                     |
| `PUID`                 | `1000`                   | User ID (UID) that Docker containers should run as, ensuring correct file permissions.                                                                                                                     | All M3TAL compose stacks                                                        |
| `PGID`                 | `1000`                   | Group ID (GID) that Docker containers should run as, ensuring correct file permissions.                                                                                                                    | All M3TAL compose stacks                                                        |
| `TZ`                   | `America/Denver`         | Timezone for containers to ensure correct timestamps and scheduling.                                                                                                                                       | All M3TAL compose stacks                                                        |
| `TRAEFIK_WEB_PORT`     | `80`                     | Host port for Traefik's HTTP entry point.                                                                                                                                                                  | `Traefik gateway`                                                               |
| `TRAEFIK_WEBHTTPS_PORT`| `443`                    | Host port for Traefik's HTTPS entry point.                                                                                                                                                                 | `Traefik gateway`                                                               |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                  | Internal port for the Traefik dashboard, exposed on `127.0.0.1:8081` on the host.                                                                                                                          | `Traefik gateway`                                                               |
| `DEBUG_MODE`           | `false`                  | Enables verbose logging and debug features across M3TAL components.                                                                                                                                        | `CLI binary`, `m3tal-api.service`, `m3tal-dashboard`                            |
| `METRICS_ENABLED`      | `true`                   | Enables or disables the exposure of Prometheus-compatible metrics endpoints.                                                                                                                               | `m3tal-api.service`, `m3tal-dashboard`                                          |

---

## Environment Variable Details

### Core Variables

These variables control fundamental aspects of the M3TAL system, including logging, debugging, and the location of the core state database.

*   **`LOG_LEVEL`**
    *   **Description:** Sets the minimum logging level for the M3TAL API daemon. Valid levels typically include `debug`, `info`, `warn`, `error`, `fatal`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `m3tal-api.service`

*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, enables verbose logging and additional debugging features across M3TAL components, useful for troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `CLI binary`, `m3tal-api.service`, `m3tal-dashboard`

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether Prometheus-compatible metrics endpoints are exposed by M3TAL services, allowing for monitoring with tools like Prometheus.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`STATE_DIR`**
    *   **Description:** Specifies the directory where the M3TAL API daemon stores its SQLite state database (`state.db`) and other runtime data. This path is volume-mounted into the dashboard container.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`API_PORT`**
    *   **Description:** The port that internal M3TAL components, such as the Dashboard, are configured to access the M3TAL API daemon on.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** `m3tal-dashboard` (via its `GO_API_URL` environment variable)

*   **`HTTP_PORT`**
    *   **Description:** The port on which the primary M3TAL API daemon (`m3tal-api.service`) listens for incoming HTTP requests from external sources, such as Traefik.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `m3tal-api.service`, `Traefik gateway` (for routing `api.DOMAIN` to the API daemon)

### Authentication Variables

These variables secure access to the M3TAL API and Dashboard.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A long, random string used by the M3TAL Dashboard for secure session management, encryption, and other cryptographic operations.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_very_long_and_random_string_of_characters`
    *   **Used By:** `m3tal-dashboard`
    *   **Note:** This variable is **auto-generated** on first `m3tal init`. Users should **NOT** set it manually unless performing a secret rotation.

*   **`API_TOKEN`**
    *   **Description:** An authentication token used for programmatic access to the M3TAL API. This token grants administrative privileges.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `another_super_secret_jwt_token_for_api_access`
    *   **Used By:** `CLI binary`, `m3tal-dashboard` (for API communication), external clients.
    *   **Note:** This variable is **auto-generated** on first `m3tal init`. Users should **NOT** set it manually unless performing a token rotation.

*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial password for the default 'admin' user of the M3TAL Dashboard. It is highly recommended to change this immediately after initial setup.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MyNewStrongP@ssw0rd!`
    *   **Used By:** `m3tal-dashboard` (for initial user creation)

### Network Variables

Variables related to Docker networking and dashboard exposure.

*   **`NETWORK_NAME`**
    *   **Description:** Defines the name of the Docker network that M3TAL's control plane services (Dashboard, Traefik, Cloudflared) and any user-defined stacks will connect to. This creates an isolated network for inter-container communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_bridge`
    *   **Used By:** `m3tal-dashboard`, `Traefik gateway`, `Cloudflared`, user Docker Compose stacks.

*   **`LOCAL_IP`**
    *   **Description:** Specifies the IP address used for host-level services, particularly for `host.docker.internal` resolution within containers to reach host-bound services like the M3TAL API daemon.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** `Traefik gateway` (when routing to host services), user Docker Compose stacks.

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly mapped to the host.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** `m3tal-dashboard` (internal container port), `CLI binary` (for `m3tal dash up` when using local mode port binding).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible.
        *   `local`: Exposes the dashboard directly on `http://HOST_IP:DASHBOARD_PORT`. No Traefik required. Best for LAN-only or initial setup.
        *   `traefik`: Configures the dashboard to be routed via Traefik at `http://dash.DOMAIN`. Requires Traefik to be running. Best for domain-based setups.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** `CLI binary` (specifically the `m3tal dash up` command to select the correct Docker Compose override file), `m3tal-dashboard` (implicitly via selected compose file).

### Storage Variables

These variables define the base paths for data storage within the M3TAL ecosystem.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host filesystem for all M3TAL-managed media, configuration, and other application data.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal_data`
    *   **Used By:** `m3tal-dashboard`, `CLI binary`, user Docker Compose stacks (as volume mounts).
    *   **Note:** In production deployments (e.g., on `/mnt` for dedicated storage), this variable typically defaults to `/mnt` rather than `./data` as seen in development templates. This is where media and configuration paths are mounted.

*   **`MEDIA_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` designated for storing media files managed by M3TAL or user applications.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `${BASE_STORAGE_PATH}/media`, `/mnt/media`
    *   **Used By:** `m3tal-dashboard` (for media management), user Docker Compose stacks.

*   **`CONFIG_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` for storing configuration files, such as the `users.json` file for dashboard credentials.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `${BASE_STORAGE_PATH}/config`, `/mnt/config`
    *   **Used By:** `m3tal-dashboard` (to access `users.json`), user Docker Compose stacks.

*   **`DOWNLOADS_PATH`**
    *   **Description:** A subdirectory within `BASE_STORAGE_PATH` designated for downloaded content by various services or user applications.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `${BASE_STORAGE_PATH}/downloads`, `/mnt/downloads`
    *   **Used By:** User Docker Compose stacks (e.g., download clients).

### Traefik Variables

These variables configure the Traefik reverse proxy for routing external traffic.

*   **`DOMAIN`**
    *   **Description:** The base domain name that M3TAL services will use when exposed via Traefik. For example, if `DOMAIN` is `example.com`, the dashboard would be accessible at `dash.example.com` and the API at `api.example.com`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `myhome.arpa`, `example.com`
    *   **Used By:** `Traefik gateway` (to define routing rules in `routing-compose.yml` and `dynamic/api.yml`), `m3tal-dashboard` (via `m3tal-compose.traefik.yml` labels).
    *   **Note:** Setting this variable enables `dash.DOMAIN` and `api.DOMAIN` routes when Traefik is running.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port on which Traefik listens for incoming HTTP traffic (entry point `web`).
    *   **Default Value:** `80`
    *   **Example Value:** `8080` (if port 80 is already in use)
    *   **Used By:** `Traefik gateway`

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port on which Traefik listens for incoming HTTPS traffic (entry point `websecure`, if configured).
    *   **Default Value:** `443`
    *   **Example Value:** `8443` (if port 443 is already in use)
    *   **Used By:** `Traefik gateway`

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal container port for Traefik's own management dashboard. This internal port is typically mapped to `127.0.0.1:8081` on the host to prevent public exposure.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `Traefik gateway` (as an internal service port)

### VPN Variables

These variables are used for configuring an optional VPN client container.

*   **`VPN_USER`**
    *   **Description:** The username required for authentication with a VPN service, used by the `cloudflared` container or other VPN client containers.
    *   **Default Value:** `user`
    *   **Example Value:** `jake`
    *   **Used By:** VPN client container (e.g., `cloudflared` or similar in user stacks)

*   **`VPN_PASSWORD`**
    *   **Description:** The password required for authentication with a VPN service, used by the `cloudflared` container or other VPN client containers.
    *   **Default Value:** `password`
    *   **Example Value:** `MySecureVPNPass`
    *   **Used By:** VPN client container (e.g., `cloudflared` or similar in user stacks)

### System Variables

General system-wide variables, often related to user permissions and localization.

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers should run as. This ensures containers have appropriate permissions to interact with host-mounted volumes, matching the permissions of a user on the host system.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used By:** All M3TAL Docker Compose stacks (`m3tal-dashboard`, user stacks).

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers should run as. Similar to `PUID`, this ensures correct group-based file permissions for host-mounted volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used By:** All M3TAL Docker Compose stacks (`m3tal-dashboard`, user stacks).

*   **`TZ`**
    *   **Description:** Sets the timezone for Docker containers. This ensures that logs, timestamps, and scheduled tasks within containers are aligned with the desired geographical region.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`, `Asia/Tokyo`
    *   **Used By:** All M3TAL Docker Compose stacks (`m3tal-dashboard`, user stacks).