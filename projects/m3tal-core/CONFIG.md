# Environment Variables Reference

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. This file serves as the single source of truth for configuration, used by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option.

---

## Quick Reference Table

| Variable Name           | Description                                                        | Default Value          | Example Value            | Components                                    |
| :---------------------- | :----------------------------------------------------------------- | :--------------------- | :----------------------- | :-------------------------------------------- |
| `DASHBOARD_PORT`        | Port for the M3TAL Dashboard in `local` expose mode.               | `8082`                 | `8082`                   | `m3tal-dashboard`, `m3tal CLI`                  |
| `DASHBOARD_EXPOSE_MODE` | How the M3TAL Dashboard is exposed (`local` or `traefik`).       | `local`                | `traefik`                | `m3tal CLI`, `m3tal-dashboard`                  |
| `HTTP_PORT`             | Port for the M3TAL API daemon.                                     | `8080`                 | `9000`                   | `m3tal-api.service`, `m3tal-dashboard`, `Traefik gateway` |
| `STATE_DIR`             | Internal container path for state database and config.             | `./state`              | `/docker/state`          | `m3tal-dashboard`, `m3tal-api.service`          |
| `LOG_LEVEL`             | Minimum logging level for the M3TAL API daemon.                    | `info`                 | `debug`                  | `m3tal-api.service`                           |
| `DASHBOARD_SECRET`      | Secret key for M3TAL Dashboard sessions.                           | `change_me_immediately`| `a_random_string`        | `m3tal-dashboard`, `m3tal CLI`                  |
| `API_TOKEN`             | API token for authenticating requests to M3TAL API.                | `change_me_api_token`  | `another_random_token`   | `m3tal-api.service`, `m3tal CLI`                  |
| `ADMIN_PASSWORD`        | Initial administrator password for M3TAL Dashboard.                | `admin_pass`           | `MySecurePa$$word123`    | `m3tal-dashboard`, `m3tal CLI`                  |
| `NETWORK_NAME`          | Name of the Docker network used by M3TAL and user stacks.          | `m3tal`                | `my_m3tal_network`       | All Docker Compose stacks, `Traefik gateway`  |
| `LOCAL_IP`              | Host machine's local IP address (reference for user stacks).       | `127.0.0.1`            | `192.168.1.100`          | *User-defined stacks*                         |
| `DOMAIN`                | Base domain for Traefik routing rules (e.g., `dash.DOMAIN`).     | `localhost`            | `myhomelab.com`          | `Traefik gateway`, `m3tal-dashboard`, `m3tal CLI` |
| `VPN_USER`              | Username for an integrated VPN service.                            | `user`                 | `m3taluser`              | *VPN container*                               |
| `VPN_PASSWORD`          | Password for an integrated VPN service.                            | `password`             | `MyStrongVPNPass`        | *VPN container*                               |
| `BASE_STORAGE_PATH`     | Base host directory for M3TAL's persistent data.                   | `./data`               | `/mnt/m3tal`             | `m3tal-dashboard`, `m3tal-api.service`, *user-defined stacks* |
| `MEDIA_PATH`            | Subdirectory within `BASE_STORAGE_PATH` for media files.           | `./data/media`         | `/mnt/m3tal/media`       | `m3tal-dashboard`, *user-defined stacks*      |
| `CONFIG_PATH`           | Base host directory for M3TAL's configuration volumes.             | `./data/config`        | `/mnt/config`            | `m3tal-dashboard`, `m3tal CLI`                  |
| `DOWNLOADS_PATH`        | Subdirectory within `BASE_STORAGE_PATH` for downloaded files.      | `./data/downloads`     | `/mnt/m3tal/downloads`   | *User-defined stacks*                         |
| `PUID`                  | User ID for running containers.                                    | `1000`                 | `1000`                   | `m3tal-dashboard`, *user-defined stacks*      |
| `PGID`                  | Group ID for running containers.                                   | `1000`                 | `1000`                   | `m3tal-dashboard`, *user-defined stacks*      |
| `TZ`                    | Timezone for containers.                                           | `America/Denver`       | `Europe/London`          | `m3tal-dashboard`, *user-defined stacks*      |
| `TRAEFIK_WEB_PORT`      | Port Traefik listens on for HTTP traffic.                          | `80`                   | `8080`                   | `Traefik gateway`                             |
| `TRAEFIK_WEBHTTPS_PORT` | Port Traefik listens on for HTTPS traffic.                         | `443`                  | `8443`                   | `Traefik gateway`                             |
| `TRAEFIK_DASHBOARD_PORT`| Internal container port for Traefik's admin dashboard.             | `8080`                 | `8080`                   | `Traefik gateway`                             |
| `DEBUG_MODE`            | Enables debug logging/features in M3TAL API.                       | `false`                | `true`                   | `m3tal-api.service`                           |
| `METRICS_ENABLED`       | Enables/disables metrics endpoints for M3TAL API.                  | `true`                 | `false`                  | `m3tal-api.service`                           |

---

## Detailed Environment Variable Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system, including dashboard access and API operations.

*   **`DASHBOARD_PORT`**
    *   **Description:** Specifies the host port on which the M3TAL Dashboard will be directly accessible when `DASHBOARD_EXPOSE_MODE` is set to `local`.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Components:** `m3tal-dashboard` container, `m3tal CLI` (when managing the dashboard in local mode).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Determines how the M3TAL Dashboard is exposed to the network.
        *   `local`: The dashboard is exposed directly on `DASHBOARD_PORT` (e.g., `http://HOST_IP:8082`). This is ideal for LAN-only setups or initial configuration.
        *   `traefik`: The dashboard is exposed via the Traefik reverse proxy at `http://dash.${DOMAIN}`. This requires Traefik to be running and a `DOMAIN` to be configured.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Components:** `m3tal CLI` (specifically `m3tal dash up`), `m3tal-dashboard` container (via compose override files).

*   **`HTTP_PORT`**
    *   **Description:** Defines the internal port on which the M3TAL API daemon (`m3tal-api.service`) listens for incoming HTTP requests.
    *   **Default Value:** `8080`
    *   **Example Value:** `9000`
    *   **Components:** `m3tal-api.service`, `m3tal-dashboard` container (via `GO_API_URL`), `Traefik gateway` (for routing to the API).

*   **`STATE_DIR`**
    *   **Description:** This variable defines the *internal container path* where the M3TAL dashboard (and implicitly, the API) expects to find persistent state data. This includes the core SQLite database (`state.db`) and user configuration (`users.json`). On the host, this directory is typically mapped via a Docker volume to `"${CONFIG_PATH:-/mnt/config}/m3tal/state"`.
    *   **Default Value:** `./state`
    *   **Example Value:** `/docker/state`
    *   **Components:** `m3tal-dashboard` container, `m3tal-api.service` (implicitly, as it uses the same underlying data).

*   **`LOG_LEVEL`**
    *   **Description:** Sets the minimum logging level for the M3TAL API daemon. Supported values typically include `debug`, `info`, `warn`, `error`, `fatal`, `panic`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Components:** `m3tal-api.service`

*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, enables additional debug logging or features within the M3TAL API daemon, useful for troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Components:** `m3tal-api.service`

*   **`METRICS_ENABLED`**
    *   **Description:** When set to `true`, enables the exposure of metrics endpoints (e.g., Prometheus format) from the M3TAL API daemon.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Components:** `m3tal-api.service`

### Authentication

These variables are critical for securing access to the M3TAL Dashboard and API.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used to secure user sessions for the M3TAL Dashboard. It's crucial for protecting against session hijacking.
    *   **Important:** This variable is **auto-generated on first `m3tal init`** and users should **NOT set it manually** unless they are intentionally rotating the secret for security reasons.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `my_very_long_and_random_dashboard_secret_key_12345`
    *   **Components:** `m3tal-dashboard` container, `m3tal CLI` (for `m3tal init` and `m3tal config set`).

*   **`API_TOKEN`**
    *   **Description:** An authentication token used to secure requests made directly to the M3TAL API daemon. Required for programmatic interaction.
    *   **Important:** This variable is **auto-generated on first `m3tal init`** and users should **NOT set it manually** unless they are intentionally rotating the token for security reasons.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a_long_and_complex_api_token_string_for_secure_access_56789`
    *   **Components:** `m3tal-api.service`, `m3tal CLI` (for `m3tal init` and `m3tal config set`).

*   **`ADMIN_PASSWORD`**
    *   **Description:** Sets the initial administrator password for the M3TAL Dashboard. This is typically managed via the `m3tal dashpass` command.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MySecurePa$$word123`
    *   **Components:** `m3tal-dashboard` (influences the `users.json` file), `m3tal CLI` (specifically `m3tal dashpass`).

### Network Configuration

Variables controlling network aspects, including Docker networks and domain routing.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL's core components and all user-defined Docker Compose stacks will connect to. This allows services to communicate with each other.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `my_m3tal_network`
    *   **Components:** All Docker Compose stacks, `Traefik gateway`.

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the host machine. While not directly used by M3TAL's core components for internal routing (which often relies on `host.docker.internal`), this variable can be useful for user-defined stacks that need to reference the host's LAN IP for specific network configurations.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Components:** *User-defined stacks*, *potential future M3TAL features*.

*   **`DOMAIN`**
    *   **Description:** The base domain name used by Traefik for routing rules. When set, Traefik will expose the M3TAL Dashboard at `dash.${DOMAIN}` and the M3TAL API at `api.${DOMAIN}`. This is essential for domain-based access and enabling `DASHBOARD_EXPOSE_MODE=traefik`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `myhomelab.com`
    *   **Components:** `Traefik gateway`, `m3tal-dashboard` (via Traefik labels), `m3tal CLI` (for generating Traefik dynamic configuration).

### Storage Paths

These variables define where M3TAL stores its various types of persistent data on the host filesystem.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The primary host directory where all M3TAL-related persistent data will be stored. This includes media, configuration, downloads, and other user-managed files. This path is frequently mounted directly into containers.
    *   **Important:** While the template defaults to `./data` for development, in **production M3TAL deployments**, this variable typically defaults to `/mnt` to align with common Linux filesystem practices for mounting storage devices.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal`
    *   **Components:** `m3tal-dashboard` container (for volumes), `m3tal-api.service` (for managing host paths), *user-defined stacks*.

*   **`MEDIA_PATH`**
    *   **Description:** Defines a subdirectory within `BASE_STORAGE_PATH` specifically designated for user media data.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal/media`
    *   **Components:** `m3tal-dashboard` container (via mounts), *user-defined stacks*.

*   **`CONFIG_PATH`**
    *   **Description:** Defines the base host path for M3TAL's *configuration-related volumes*. For instance, the M3TAL Dashboard's `/docker/state` (which contains `users.json` at `/docker/state/config/users.json`) is mounted from `${CONFIG_PATH}/m3tal/state`. While the `.env` file itself resides at `/etc/m3tal/.env`, this variable influences where *other* configuration files, like `users.json`, are stored on the host for container access.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Components:** `m3tal-dashboard` container (for `users.json`), `m3tal CLI`.

*   **`DOWNLOADS_PATH`**
    *   **Description:** Defines a subdirectory within `BASE_STORAGE_PATH` intended for downloaded files by various services.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal/downloads`
    *   **Components:** *User-defined stacks* (e.g., download clients).

### Traefik Gateway

These variables configure the behavior and exposed ports of the Traefik reverse proxy.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik will listen on for incoming HTTP traffic (typically port 80).
    *   **Default Value:** `80`
    *   **Example Value:** `8080` (if port 80 is already in use by another service)
    *   **Components:** `Traefik gateway`

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik will listen on for incoming HTTPS traffic (typically port 443).
    *   **Default Value:** `443`
    *   **Example Value:** `8443` (if port 443 is already in use by another service)
    *   **Components:** `Traefik gateway`

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port used by the Traefik container for its administration dashboard. The host mapping for this port is typically hardcoded to `127.0.0.1:8081` in `routing-compose.yml`, making the Traefik dashboard accessible locally on port 8081. This variable itself, if changed, would primarily affect Traefik's internal configuration rather than the host binding.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Components:** `Traefik gateway`

### VPN Integration

Variables for integrating a VPN service within your M3TAL ecosystem.

*   **`VPN_USER`**
    *   **Description:** The username to be used for authenticating with an integrated VPN server, if a VPN stack is deployed.
    *   **Default Value:** `user`
    *   **Example Value:** `m3taluser`
    *   **Components:** *VPN container* (if a VPN stack is added).

*   **`VPN_PASSWORD`**
    *   **Description:** The password corresponding to `VPN_USER` for authenticating with an integrated VPN server.
    *   **Default Value:** `password`
    *   **Example Value:** `MyStrongVPNPass`
    *   **Components:** *VPN container* (if a VPN stack is added).

### System & User

General system-level configurations often used by multiple containers.

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers will use to run their processes. This is crucial for ensuring correct file permissions when containers interact with host-mounted volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Components:** `m3tal-dashboard` container, *user-defined stacks*.

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers will use to run their processes. Similar to `PUID`, this ensures proper group-level file permissions on mounted volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Components:** `m3tal-dashboard` container, *user-defined stacks*.

*   **`TZ`**
    *   **Description:** Sets the timezone for containers. This affects timestamps in logs and internal operations.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `America/New_York` or `Europe/London`
    *   **Components:** `m3tal-dashboard` container, *user-defined stacks*.