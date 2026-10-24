# Environment Variables Reference

All M3TAL environment variables are centrally managed in the `/etc/m3tal/.env` file. This file serves as the single source of truth for configuration, being read by both the `m3tal` CLI binary and all Docker Compose stacks via the `--env-file` option.

You can manage these variables using the `m3tal config wizard` for interactive setup, or `m3tal config set KEY value` for direct modification.

---

### Quick Reference

| Variable Name               | Default Value           | Description                                                                     |
| :-------------------------- | :---------------------- | :------------------------------------------------------------------------------ |
| `API_PORT`                  | `5050`                  | Internal API daemon port. (See `HTTP_PORT` for external).                       |
| `DASHBOARD_PORT`            | `8082`                  | Internal port for the M3TAL Dashboard container.                                |
| `DASHBOARD_EXPOSE_MODE`     | `local`                 | Controls dashboard access: `local` (direct port) or `traefik` (domain routing). |
| `HTTP_PORT`                 | `8080`                  | The primary port the M3TAL API daemon listens on. Traefik routes to this port.  |
| `STATE_DIR`                 | `./state`               | Directory for the M3TAL API's SQLite database.                                  |
| `LOG_LEVEL`                 | `info`                  | Logging verbosity for CLI and API.                                              |
| `DASHBOARD_SECRET`          | `change_me_immediately` | Session secret for the M3TAL Dashboard. **Auto-generated.**                     |
| `API_TOKEN`                 | `change_me_api_token`   | Auth token for CLI/Dashboard to API daemon. **Auto-generated.**                 |
| `ADMIN_PASSWORD`            | `admin_pass`            | Default password for the `admin` dashboard user.                                |
| `NETWORK_NAME`              | `m3tal`                 | Default Docker network name for services.                                       |
| `LOCAL_IP`                  | `127.0.0.1`             | The local IP address of the host.                                               |
| `DOMAIN`                    | `localhost`             | Primary domain for Traefik routing (e.g., `dash.DOMAIN`, `api.DOMAIN`).         |
| `VPN_USER`                  | `user`                  | Username for VPN service integration.                                           |
| `VPN_PASSWORD`              | `password`              | Password for VPN service integration.                                           |
| `BASE_STORAGE_PATH`         | `./data`                | Root directory for all M3TAL data. Defaults to `/mnt` in production.            |
| `MEDIA_PATH`                | `./data/media`          | Sub-path for media files.                                                       |
| `CONFIG_PATH`               | `./data/config`         | Sub-path for configuration files.                                               |
| `DOWNLOADS_PATH`            | `./data/downloads`      | Sub-path for downloaded files.                                                  |
| `PUID`                      | `1000`                  | User ID for container processes.                                                |
| `PGID`                      | `1000`                  | Group ID for container processes.                                               |
| `TZ`                        | `America/Denver`        | Timezone for containers.                                                        |
| `TRAEFIK_WEB_PORT`          | `80`                    | Host port for Traefik's HTTP entry point.                                       |
| `TRAEFIK_WEBHTTPS_PORT`     | `443`                   | Host port for Traefik's HTTPS entry point.                                      |
| `TRAEFIK_DASHBOARD_PORT`    | `8080`                  | Internal port for Traefik's own dashboard. (Host-mapped to `127.0.0.1:8081`).   |
| `DEBUG_MODE`                | `false`                 | Enables debug logging and features.                                             |
| `METRICS_ENABLED`           | `true`                  | Enables Prometheus-compatible metrics for the API daemon.                       |

---

### Detailed Environment Variable Reference

#### Core Configuration

These variables control fundamental aspects of the M3TAL system's operation and user context.

*   **`STATE_DIR`**
    *   **Description:** The directory where the M3TAL API daemon stores its SQLite state database (`state.db`) and other runtime files. In production deployments, this typically resolves to `/var/lib/m3tal/state`.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used by:** M3TAL API daemon, M3TAL Dashboard container

*   **`LOG_LEVEL`**
    *   **Description:** Sets the logging verbosity for the M3TAL CLI and API daemon. Valid values include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used by:** CLI, M3TAL API daemon

*   **`PUID`**
    *   **Description:** The User ID (UID) that containers should run as to match host filesystem permissions. This ensures that files created by containers have the correct ownership on the host system.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used by:** All Docker containers configured with `user: "${PUID:-1000}:${PGID:-1000}"`

*   **`PGID`**
    *   **Description:** The Group ID (GID) that containers should run as to match host filesystem permissions. This ensures that files created by containers have the correct group ownership on the host system.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used by:** All Docker containers configured with `user: "${PUID:-1000}:${PGID:-1000}"`

*   **`TZ`**
    *   **Description:** Sets the timezone inside containers, ensuring correct timestamping and time-dependent operations. Uses standard `tzdata` format.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Used by:** All Docker containers (especially M3TAL Dashboard, user stacks)

*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, enables verbose debug logging and potentially other debug features across M3TAL components. Useful for troubleshooting.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used by:** CLI, M3TAL API daemon, M3TAL Dashboard

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether the M3TAL API daemon collects and exposes Prometheus-compatible metrics for monitoring.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used by:** M3TAL API daemon

#### Authentication & Security

These variables are critical for securing access to M3TAL components.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used by the M3TAL Dashboard for session management, cookie signing, and securing user credentials. This variable is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a key rotation.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `your_secure_random_string_here_12345`
    *   **Used by:** M3TAL Dashboard container

*   **`API_TOKEN`**
    *   **Description:** An authentication token used by the M3TAL CLI and Dashboard to communicate securely with the M3TAL API daemon. This variable is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a key rotation.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a_long_cryptographic_token_for_api_auth`
    *   **Used by:** CLI, M3TAL API daemon, M3TAL Dashboard container

*   **`ADMIN_PASSWORD`**
    *   **Description:** The default password for the `admin` user on the M3TAL Dashboard. It is highly recommended to change this during `m3tal config wizard` or using the `m3tal dashpass` command after initial setup.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MyStrongAndSecurePassword123!`
    *   **Used by:** M3TAL Dashboard container

#### Network & Ports

Variables defining how M3TAL services communicate and are exposed over the network.

*   **`API_PORT`**
    *   **Description:** An internal configuration variable for the M3TAL API daemon, potentially for an alternate or internal API endpoint. The primary external HTTP API for the daemon is exposed via `HTTP_PORT` (default 8080), which Traefik routes to. Changing `API_PORT` alone will not change the port the systemd service exposes or that Traefik routes to without additional configuration changes.
    *   **Default Value:** `5050`
    *   **Example Value:** `5051`
    *   **Used by:** M3TAL API daemon (internal configuration)

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL Dashboard container listens for HTTP requests. This port is either directly exposed on the host in `local` mode or routed by Traefik in `traefik` mode.
    *   **Default Value:** `8082`
    *   **Example Value:** `8090`
    *   **Used by:** M3TAL Dashboard container, Traefik (in `traefik` mode)

*   **`HTTP_PORT`**
    *   **Description:** The port used by the M3TAL API daemon to serve HTTP requests. This is the port the `m3tal-api.service` listens on, and the port Traefik routes to for `api.DOMAIN`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used by:** M3TAL API daemon, Traefik

*   **`NETWORK_NAME`**
    *   **Description:** The default Docker network name for M3TAL components. Note that core M3TAL services and Traefik typically use an external network named `proxy` for inter-service communication. This variable might be used for user-defined Docker Compose stacks that rely on a default network name.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `my_custom_docker_network`
    *   **Used by:** User-defined Docker Compose stacks (potentially), M3TAL API daemon (when creating default networks)

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the host machine. This variable can be used for internal routing, direct service access, or for services that need to bind to a specific host interface.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used by:** Various services needing host IP, internal routing

#### Storage Paths

These variables define the base directories for various types of data managed by M3TAL.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory for all M3TAL media, configuration, and user data. **Important:** While the template defaults to `./data`, in production deployments (e.g., on a server with dedicated storage), this variable defaults to `/mnt` (e.g., `/mnt/data`) to align with common server storage conventions.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal_data`
    *   **Used by:** All Docker containers needing persistent storage (M3TAL Dashboard, user stacks)

*   **`MEDIA_PATH`**
    *   **Description:** The sub-path for storing media files, relative to `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `${BASE_STORAGE_PATH}/media` (or `/mnt/media`)
    *   **Used by:** User-defined media services

*   **`CONFIG_PATH`**
    *   **Description:** The sub-path for storing M3TAL configuration files and persistent data, relative to `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `${BASE_STORAGE_PATH}/config` (or `/mnt/config`)
    *   **Used by:** M3TAL Dashboard container, M3TAL API daemon (for some config), user-defined services

*   **`DOWNLOADS_PATH`**
    *   **Description:** The sub-path for storing downloaded files, relative to `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `${BASE_STORAGE_PATH}/downloads` (or `/mnt/downloads`)
    *   **Used by:** User-defined download services

#### Traefik & Routing

Variables specific to the Traefik reverse proxy and domain-based routing.

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is exposed.
        *   `local`: Exposes the dashboard directly on `http://HOST_IP:DASHBOARD_PORT`. No Traefik required.
        *   `traefik`: Configures Traefik to route `http://dash.DOMAIN` to the dashboard. Traefik must be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used by:** CLI (for `m3tal dash up` command), M3TAL Dashboard container (via compose overrides)

*   **`DOMAIN`**
    *   **Description:** The primary domain name for M3TAL services when using Traefik. Setting this variable enables `dash.DOMAIN` and `api.DOMAIN` routes. If using Traefik, this should be a public domain or a local DNS entry configured on your network.
    *   **Default Value:** `localhost`
    *   **Example Value:** `example.com`
    *   **Used by:** Traefik gateway, M3TAL Dashboard (in `traefik` mode), M3TAL API daemon (indirectly via Traefik routing)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port for Traefik's primary HTTP entry point. This is the port where external HTTP traffic is received.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used by:** Traefik gateway

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port for Traefik's primary HTTPS entry point. This is the port where external HTTPS traffic is received.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used by:** Traefik gateway

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik's own dashboard listens on. This internal port is typically mapped to `127.0.0.1:8081` on the host, making it accessible only locally for security reasons.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used by:** Traefik gateway

#### VPN Integration

Variables for configuring an optional VPN service if integrated with M3TAL.

*   **`VPN_USER`**
    *   **Description:** The username for authenticating with a VPN service if one is configured as part of your M3TAL deployment.
    *   **Default Value:** `user`
    *   **Example Value:** `vpnadmin`
    *   **Used by:** VPN container (e.g., WireGuard, OpenVPN if integrated)

*   **`VPN_PASSWORD`**
    *   **Description:** The password for authenticating with a VPN service if one is configured as part of your M3TAL deployment.
    *   **Default Value:** `password`
    *   **Example Value:** `my_strong_vpn_password`
    *   **Used by:** VPN container (e.g., WireGuard, OpenVPN if integrated)