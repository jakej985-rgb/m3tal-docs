# Environment Variables Reference

All M3TAL environment variables are centrally managed and read from the configuration file at `/etc/m3tal/.env`. This file is used by the `m3tal` CLI, the `m3tal-api.service` daemon, and all Docker Compose stacks (including the M3TAL Dashboard, Traefik, and user-deployed services) via the `--env-file` option.

Modifying these variables typically involves using the `m3tal config wizard` for interactive setup or `m3tal config set <KEY> <VALUE>` for specific changes. After modifying sensitive or critical variables, a restart of affected services (`m3tal up` or `systemctl restart m3tal-api`) may be necessary.

---

## Quick Reference

| Name                      | Default Value           | Description                                                                                                                                                                                                                                 |
|---------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ADMIN_PASSWORD            | `admin_pass`            | The initial password for the default `admin` user account in the M3TAL Dashboard.                                                                                                                                                           |
| API_TOKEN                 | `change_me_api_token`   | Token for authentication with the M3TAL API daemon. **Auto-generated on `m3tal init`.**                                                                                                                                                     |
| BASE_STORAGE_PATH         | `./data`                | The root directory for all M3TAL data storage (media, config, downloads). **Defaults to `/mnt` in production.**                                                                                                                           |
| CONFIG_PATH               | `./data/config`         | Path for configuration files, typically relative to `BASE_STORAGE_PATH`.                                                                                                                                                                    |
| DASHBOARD_EXPOSE_MODE     | `local`                 | Controls M3TAL Dashboard exposure: `local` (direct port) or `traefik` (via `dash.${DOMAIN}`).                                                                                                                                                |
| DASHBOARD_PORT            | `8082`                  | The port on which the M3TAL Dashboard container listens and is exposed in `local` mode.                                                                                                                                                     |
| DASHBOARD_SECRET          | `change_me_immediately` | Secret key for M3TAL Dashboard session management. **Auto-generated on `m3tal init`.**                                                                                                                                                      |
| DEBUG_MODE                | `false`                 | Enables debug logging and features for M3TAL components.                                                                                                                                                                                    |
| DOMAIN                    | `localhost`             | The primary domain name for Traefik routing, enabling `dash.DOMAIN` and `api.DOMAIN`.                                                                                                                                                       |
| DOWNLOADS_PATH            | `./data/downloads`      | Path for downloaded files, typically relative to `BASE_STORAGE_PATH`.                                                                                                                                                                       |
| HTTP_PORT                 | `8080`                  | The port on which the M3TAL API daemon listens.                                                                                                                                                                                             |
| LOCAL_IP                  | `127.0.0.1`             | The local IP address of the host machine, used for internal Docker mappings.                                                                                                                                                                |
| LOG_LEVEL                 | `info`                  | Minimum log level for M3TAL API daemon and other components.                                                                                                                                                                                |
| MEDIA_PATH                | `./data/media`          | Path for media files, typically relative to `BASE_STORAGE_PATH`.                                                                                                                                                                            |
| METRICS_ENABLED           | `true`                  | Controls whether internal metrics collection is enabled.                                                                                                                                                                                    |
| NETWORK_NAME              | `m3tal`                 | The name of the Docker network used by M3TAL for internal services.                                                                                                                                                                         |
| PGID                      | `1000`                  | The Group ID (GID) that container processes should run as.                                                                                                                                                                                  |
| PUID                      | `1000`                  | The User ID (UID) that container processes should run as.                                                                                                                                                                                   |
| STATE_DIR                 | `./state`               | Directory for the M3TAL API daemon's SQLite state database and runtime files.                                                                                                                                                               |
| TRAEFIK_DASHBOARD_PORT    | `8080`                  | The internal container port Traefik's API and dashboard listens on.                                                                                                                                                                         |
| TRAEFIK_WEB_PORT          | `80`                    | The host port Traefik uses for the standard HTTP entry point.                                                                                                                                                                               |
| TRAEFIK_WEBHTTPS_PORT     | `443`                   | The host port Traefik uses for the standard HTTPS entry point.                                                                                                                                                                              |
| TZ                        | `America/Denver`        | The timezone for containers, ensuring correct timestamps.                                                                                                                                                                                   |
| VPN_PASSWORD              | `password`              | Password for VPN authentication (if a VPN service is integrated).                                                                                                                                                                           |
| VPN_USER                  | `user`                  | Username for VPN authentication (if a VPN service is integrated).                                                                                                                                                                           |

---

## Detailed Variable Reference

### Core Variables

These variables define fundamental aspects of the M3TAL ecosystem's operation.

#### `DASHBOARD_PORT`
- **Description**: The port on which the M3TAL Dashboard container listens internally and is exposed to the host machine when `DASHBOARD_EXPOSE_MODE` is set to `local`.
- **Default Value**: `8082`
- **Example Value**: `8082`
- **Component(s) Used**: `m3tal-dashboard` container, `m3tal dash up` command, `m3tal-compose.local.yml`

#### `DASHBOARD_EXPOSE_MODE`
- **Description**: Controls how the M3TAL Dashboard is made accessible.
    - `local`: The dashboard is exposed directly on the host's `DASHBOARD_PORT`. Best for LAN-only setups or initial configuration.
    - `traefik`: The dashboard is exposed via Traefik, accessible at `http://dash.${DOMAIN}`. Requires Traefik to be running.
- **Default Value**: `local`
- **Example Value**: `traefik`
- **Component(s) Used**: `m3tal` CLI, `m3tal dash up` command, `m3tal-compose.yml` (via overrides)

#### `HTTP_PORT`
- **Description**: The port on which the M3TAL API daemon (`m3tal-api.service`) listens for incoming HTTP requests. This is typically only accessible locally on the host, or via Traefik when `DOMAIN` is configured (routing `api.${DOMAIN}` to `http://host.docker.internal:${HTTP_PORT}`).
- **Default Value**: `8080`
- **Example Value**: `8080`
- **Component(s) Used**: `m3tal-api.service` (API daemon)

#### `STATE_DIR`
- **Description**: The directory where the M3TAL API daemon stores its SQLite state database (`state.db`) and other runtime files. For the dashboard container, this path is mounted into `/docker/state`.
- **Default Value**: `./state`
- **Example Value**: `/var/lib/m3tal/state` (typical production path)
- **Component(s) Used**: `m3tal-api.service` (API daemon), `m3tal-dashboard` container (via volume mount)

### Authentication Variables

These variables are crucial for securing access to the M3TAL API and Dashboard.

#### `ADMIN_PASSWORD`
- **Description**: The initial password for the default `admin` user account used to log into the M3TAL Dashboard. This can be managed and changed using the `m3tal dashpass` command.
- **Default Value**: `admin_pass`
- **Example Value**: `mySuperSecurePa$$w0rd`
- **Component(s) Used**: `m3tal-dashboard` container (for `/docker/state/config/users.json`), `m3tal` CLI (`m3tal dashpass`)

#### `API_TOKEN`
- **Description**: An API token used for authentication when interacting with the M3TAL API daemon.
- **Default Value**: `change_me_api_token`
- **Example Value**: `aSuperLongAndComplexRandomTokenString`
- **Important Note**: This token is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a token rotation.
- **Component(s) Used**: `m3tal-api.service` (API daemon), `m3tal` CLI (`m3tal init`)

#### `DASHBOARD_SECRET`
- **Description**: A secret key used by the M3TAL Dashboard for session management, encryption, and other security-sensitive operations.
- **Default Value**: `change_me_immediately`
- **Example Value**: `anotherVeryStrongRandomStringForSecurity`
- **Important Note**: This secret is **auto-generated** on the first `m3tal init` run. Users should **NOT** set it manually unless performing a secret rotation.
- **Component(s) Used**: `m3tal-dashboard` container, `m3tal` CLI (`m3tal init`)

### Network Variables

These variables configure how M3TAL services communicate over the network, both internally and externally.

#### `DOMAIN`
- **Description**: The primary domain name for Traefik routing. Setting this value enables Traefik to create routing rules for `dash.${DOMAIN}` (for the M3TAL Dashboard) and `api.${DOMAIN}` (for the M3TAL API daemon).
- **Default Value**: `localhost`
- **Example Value**: `myhomelab.com`
- **Component(s) Used**: `traefik` container, `m3tal-compose.traefik.yml`, `routing-compose.yml` (specifically `dynamic/api.yml`)

#### `LOCAL_IP`
- **Description**: The local IP address of the host machine. This variable is used to configure `extra_hosts` entries in Docker Compose files, such as mapping `host.docker.internal` to the host's actual IP, which allows containers to communicate with host-bound services (like the `m3tal-api.service`).
- **Default Value**: `127.0.0.1`
- **Example Value**: `192.168.1.100`
- **Component(s) Used**: `m3tal-dashboard` container, `routing-compose.yml` (for `dynamic/api.yml` to target `host.docker.internal`)

#### `NETWORK_NAME`
- **Description**: The name of the Docker network M3TAL uses to connect its internal services and user-deployed stacks. All containers that need to communicate with each other (e.g., Traefik and Dashboard, or Traefik and user stacks) must be on this network.
- **Default Value**: `m3tal`
- **Example Value**: `m3tal-proxy-network`
- **Component(s) Used**: `m3tal` CLI, all Docker Compose stacks (`networks: proxy: external: true` reference this)

### Storage Variables

These variables define the filesystem paths where M3TAL stores its data, configuration, and user-generated content.

#### `BASE_STORAGE_PATH`
- **Description**: The base directory where M3TAL stores all persistent data, including media, configuration, and downloads. All other storage paths (`MEDIA_PATH`, `CONFIG_PATH`, `DOWNLOADS_PATH`) are typically defined relative to this path.
- **Default Value**: `./data`
- **Example Value**: `/mnt`
- **Important Note**: While the template defaults to `./data` for local development, in production deployments, this variable **defaults to `/mnt`** to align with typical Linux file system layouts for persistent storage.
- **Component(s) Used**: `m3tal-dashboard` container (volume mount), `m3tal` CLI, user-deployed Docker Compose stacks

#### `CONFIG_PATH`
- **Description**: The directory used for storing M3TAL's internal configuration files and potentially configuration for user-deployed stacks. This path is typically relative to `BASE_STORAGE_PATH`. For the dashboard, `${CONFIG_PATH}/m3tal/state` is mounted into `/docker/state/config`.
- **Default Value**: `./data/config`
- **Example Value**: `/mnt/config`
- **Component(s) Used**: `m3tal-dashboard` container (volume mount), `m3tal` CLI, user-deployed Docker Compose stacks

#### `DOWNLOADS_PATH`
- **Description**: The directory designated for downloaded files. This path is typically relative to `BASE_STORAGE_PATH`.
- **Default Value**: `./data/downloads`
- **Example Value**: `/mnt/downloads`
- **Component(s) Used**: User-deployed Docker Compose stacks (e.g., download clients)

#### `MEDIA_PATH`
- **Description**: The directory where media files managed by M3TAL or user-deployed stacks are stored. This path is typically relative to `BASE_STORAGE_PATH`.
- **Default Value**: `./data/media`
- **Example Value**: `/mnt/media`
- **Component(s) Used**: User-deployed Docker Compose stacks

### Traefik Variables

These variables specifically configure the Traefik reverse proxy, which handles external routing to M3TAL services.

#### `TRAEFIK_DASHBOARD_PORT`
- **Description**: The internal container port that Traefik's own API and dashboard listens on. This port is typically mapped to `127.0.0.1:8081` on the host to provide local access to the Traefik dashboard.
- **Default Value**: `8080`
- **Example Value**: `8080` (container internal port)
- **Component(s) Used**: `traefik` container in `routing-compose.yml`

#### `TRAEFIK_WEB_PORT`
- **Description**: The host port that Traefik uses for the standard HTTP (non-HTTPS) entry point. This is the port users connect to for HTTP access to services routed by Traefik.
- **Default Value**: `80`
- **Example Value**: `80`
- **Component(s) Used**: `traefik` container in `routing-compose.yml`

#### `TRAEFIK_WEBHTTPS_PORT`
- **Description**: The host port that Traefik uses for the standard HTTPS entry point. This is the port users connect to for secure HTTPS access to services routed by Traefik.
- **Default Value**: `443`
- **Example Value**: `443`
- **Component(s) Used**: `traefik` container in `routing-compose.yml`

### VPN Variables

These variables are placeholders for potential future VPN integration with M3TAL or user services.

#### `VPN_PASSWORD`
- **Description**: Placeholder for the password used for VPN authentication, if a VPN service is integrated or planned within the M3TAL ecosystem.
- **Default Value**: `password`
- **Example Value**: `mySecureVpnPassword123`
- **Component(s) Used**: Currently not explicitly used in core M3TAL components but available for user-defined VPN stacks.

#### `VPN_USER`
- **Description**: Placeholder for the username used for VPN authentication, if a VPN service is integrated or planned within the M3TAL ecosystem.
- **Default Value**: `user`
- **Example Value**: `m3tal_vpn_user`
- **Component(s) Used**: Currently not explicitly used in core M3TAL components but available for user-defined VPN stacks.

### System Variables

These variables control general system-wide settings, user/group IDs for containers, and debugging options.

#### `DEBUG_MODE`
- **Description**: When set to `true`, enables verbose debug logging and potentially other debug-specific features across M3TAL components, assisting in troubleshooting.
- **Default Value**: `false`
- **Example Value**: `true`
- **Component(s) Used**: `m3tal` CLI, `m3tal-api.service` (API daemon), `m3tal-dashboard` container (potentially)

#### `LOG_LEVEL`
- **Description**: The minimum level of logs to output for the M3TAL API daemon and other compatible components. Common levels include `debug`, `info`, `warn`, `error`, and `fatal`.
- **Default Value**: `info`
- **Example Value**: `debug`
- **Component(s) Used**: `m3tal-api.service` (API daemon), `m3tal` CLI

#### `METRICS_ENABLED`
- **Description**: Controls whether internal metrics collection is enabled for M3TAL components, allowing for performance monitoring and analysis.
- **Default Value**: `true`
- **Example Value**: `false`
- **Component(s) Used**: `m3tal-api.service` (API daemon), `m3tal-dashboard` container (potentially)

#### `PGID`
- **Description**: The Group ID (GID) that container processes should run as inside the container. This ensures correct file ownership and permissions for volumes mounted from the host system. It should typically match the GID of the user account on the host that owns the data directories.
- **Default Value**: `1000`
- **Example Value**: `1001`
- **Component(s) Used**: `m3tal-dashboard` container, other user-deployed Docker Compose containers

#### `PUID`
- **Description**: The User ID (UID) that container processes should run as inside the container. This ensures correct file ownership and permissions for volumes mounted from the host system. It should typically match the UID of the user account on the host that owns the data directories.
- **Default Value**: `1000`
- **Example Value**: `1001`
- **Component(s) Used**: `m3tal-dashboard` container, other user-deployed Docker Compose containers

#### `TZ`
- **Description**: The timezone setting for containers. Setting this ensures that logs, timestamps, and scheduled tasks within containers reflect the correct local time. Use standard timezone names (e.g., `America/New_York`, `Europe/London`).
- **Default Value**: `America/Denver`
- **Example Value**: `Europe/Berlin`
- **Component(s) Used**: `m3tal-dashboard` container, other user-deployed Docker Compose containers