# Environment Variables Reference

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. Both the `m3tal` CLI binary and all Docker Compose stacks (started via `m3tal up`) consume these variables by passing `--env-file /etc/m3tal/.env` to `docker compose`.

These variables control everything from network ports and storage paths to authentication secrets and logging verbosity, allowing for granular configuration of your M3TAL ecosystem.

---

## Quick Reference

| Name                    | Description                                                                  | Default                  | Component(s)                                   |
| :---------------------- | :--------------------------------------------------------------------------- | :----------------------- | :--------------------------------------------- |
| `API_PORT`              | Internal port for container communication to API daemon.                     | `5050`                   | Dashboard                                      |
| `DASHBOARD_PORT`        | Port the M3TAL Dashboard container listens on.                               | `8082`                   | Dashboard                                      |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed (local port or Traefik).               | `local`                  | CLI, Dashboard                                 |
| `HTTP_PORT`             | Host port the M3TAL API daemon listens on.                                   | `8080`                   | API daemon                                     |
| `STATE_DIR`             | Directory for the SQLite state database.                                     | `./state`                | API daemon, Dashboard                          |
| `LOG_LEVEL`             | Logging verbosity level.                                                     | `info`                   | CLI, API daemon                                |
| `DASHBOARD_SECRET`      | Secret key for dashboard session management.                                 | `change_me_immediately`  | Dashboard                                      |
| `API_TOKEN`             | Authentication token for the M3TAL API.                                      | `change_me_api_token`    | CLI, API daemon                                |
| `ADMIN_PASSWORD`        | Default password for the initial dashboard user.                             | `admin_pass`             | Dashboard                                      |
| `NETWORK_NAME`          | Default name of the Docker network for internal M3TAL services.              | `m3tal`                  | Compose Stacks                                 |
| `LOCAL_IP`              | IP address for host-level communication (e.g., `host.docker.internal`).      | `127.0.0.1`              | Dashboard, Traefik                             |
| `DOMAIN`                | Base domain for Traefik routing.                                             | `localhost`              | CLI, Traefik, Dashboard                        |
| `VPN_USER`              | Username for VPN client configuration (e.g., Cloudflared).                   | `user`                   | Cloudflared                                    |
| `VPN_PASSWORD`          | Password for VPN client configuration (e.g., Cloudflared).                   | `password`               | Cloudflared                                    |
| `BASE_STORAGE_PATH`     | Base directory for all M3TAL data volumes.                                   | `./data`                 | All Compose Stacks                             |
| `MEDIA_PATH`            | Path for media storage.                                                      | `./data/media`           | All Compose Stacks                             |
| `CONFIG_PATH`           | Path for configuration files.                                                | `./data/config`          | All Compose Stacks                             |
| `DOWNLOADS_PATH`        | Path for downloads storage.                                                  | `./data/downloads`       | All Compose Stacks                             |
| `PUID`                  | User ID for container processes.                                             | `1000`                   | All Compose Stacks                             |
| `PGID`                  | Group ID for container processes.                                            | `1000`                   | All Compose Stacks                             |
| `TZ`                    | Timezone for containers.                                                     | `America/Denver`         | All Compose Stacks                             |
| `TRAEFIK_WEB_PORT`      | Port Traefik listens on for HTTP traffic.                                    | `80`                     | Traefik                                        |
| `TRAEFIK_WEBHTTPS_PORT` | Port Traefik listens on for HTTPS traffic.                                   | `443`                    | Traefik                                        |
| `TRAEFIK_DASHBOARD_PORT`| Internal port for Traefik's own dashboard.                                   | `8080`                   | Traefik                                        |
| `DEBUG_MODE`            | Enables verbose debugging output.                                            | `false`                  | CLI, API daemon, Dashboard                     |
| `METRICS_ENABLED`       | Enables Prometheus metrics endpoint on the API daemon.                       | `true`                   | API daemon                                     |

---

## Detailed Variable Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system.

#### `LOG_LEVEL`
- **Description**: Sets the minimum logging verbosity level for the CLI and API daemon. Options typically include `debug`, `info`, `warn`, `error`, `fatal`.
- **Default Value**: `info`
- **Example Value**: `debug`
- **Used by**: CLI, M3TAL API daemon

#### `STATE_DIR`
- **Description**: Specifies the directory where the M3TAL API daemon stores its SQLite state database (`state.db`).
- **Default Value**: `./state`
- **Example Value**: `/var/lib/m3tal` (The actual database file will be `/var/lib/m3tal/state.db`)
- **Used by**: M3TAL API daemon, M3TAL Dashboard

#### `DEBUG_MODE`
- **Description**: When set to `true`, enables additional verbose debugging output across M3TAL components, useful for troubleshooting.
- **Default Value**: `false`
- **Example Value**: `true`
- **Used by**: CLI, M3TAL API daemon, M3TAL Dashboard

#### `METRICS_ENABLED`
- **Description**: Controls whether the M3TAL API daemon exposes a Prometheus-compatible metrics endpoint.
- **Default Value**: `true`
- **Example Value**: `false`
- **Used by**: M3TAL API daemon

### Authentication & Security

These variables manage access and security within the M3TAL ecosystem.

#### `DASHBOARD_SECRET`
- **Description**: A cryptographic secret used by the M3TAL Dashboard for session management and securing user data. **This value is automatically generated on first `m3tal init` and should NOT be set manually unless you are intentionally rotating it.**
- **Default Value**: `change_me_immediately`
- **Example Value**: `your_long_random_and_secure_secret_key`
- **Used by**: M3TAL Dashboard container

#### `API_TOKEN`
- **Description**: An authentication token used to secure communication with the M3TAL API daemon. **This value is automatically generated on first `m3tal init` and should NOT be set manually unless you are intentionally rotating it.**
- **Default Value**: `change_me_api_token`
- **Example Value**: `another_long_random_and_secure_api_token`
- **Used by**: CLI, M3TAL API daemon

#### `ADMIN_PASSWORD`
- **Description**: Sets the initial default password for the `admin` user on the M3TAL Dashboard. You should change this immediately after first login.
- **Default Value**: `admin_pass`
- **Example Value**: `my_secure_admin_password`
- **Used by**: M3TAL Dashboard container

### Network & Ports

Variables for configuring network access and service ports.

#### `API_PORT`
- **Description**: The *internal* port that containerized M3TAL components (like the Dashboard) use to access the host-running M3TAL API daemon via `host.docker.internal`. The API daemon itself listens on `HTTP_PORT`.
- **Default Value**: `5050`
- **Example Value**: `5050`
- **Used by**: M3TAL Dashboard container (via `GO_API_URL`)

#### `DASHBOARD_PORT`
- **Description**: The port on which the M3TAL Dashboard container listens *internally*. This port is either directly exposed on the host (in `local` mode) or routed via Traefik.
- **Default Value**: `8082`
- **Example Value**: `8082`
- **Used by**: M3TAL Dashboard container

#### `HTTP_PORT`
- **Description**: The actual host port on which the M3TAL API daemon listens for all local traffic, including from Traefik.
- **Default Value**: `8080`
- **Example Value**: `8080`
- **Used by**: M3TAL API daemon, Traefik gateway

#### `NETWORK_NAME`
- **Description**: The default name for internal Docker networks created by M3TAL compose stacks. While many critical services (especially those integrated with Traefik) use an externally defined `proxy` network, this variable can define a default for other internal components.
- **Default Value**: `m3tal`
- **Example Value**: `m3tal_internal`
- **Used by**: All M3TAL compose stacks (for internal network definition)

#### `LOCAL_IP`
- **Description**: Used to specify the IP address for host-level communication within Docker containers (e.g., when `host.docker.internal` is mapped).
- **Default Value**: `127.0.0.1`
- **Example Value**: `192.168.1.10`
- **Used by**: M3TAL Dashboard container, Traefik gateway (for routing to host-local services)

### Dashboard Access Mode

This critical variable dictates how the M3TAL Dashboard is made accessible.

#### `DASHBOARD_EXPOSE_MODE`
- **Description**: Controls how the M3TAL Dashboard container is exposed.
    - `local`: (Default) The dashboard port (`DASHBOARD_PORT`) is directly mapped to the host, making it accessible via `http://HOST_IP:8082` or `http://localhost:8082`. This mode does not require Traefik.
    - `traefik`: The dashboard is configured with Traefik labels, making it accessible via `http://dash.DOMAIN` (Traefik must be running). This mode uses the `proxy` network.
- **Default Value**: `local`
- **Example Value**: `traefik`
- **Used by**: CLI (`m3tal dash up` command), M3TAL Dashboard container (via compose override selection)

### Storage Paths

These variables define the base directories for M3TAL's data storage volumes.

#### `BASE_STORAGE_PATH`
- **Description**: The primary directory where all M3TAL data volumes (media, config, downloads) are stored. **In production deployments, this typically defaults to `/mnt` rather than `./data` to leverage a dedicated data partition.**
- **Default Value**: `./data`
- **Example Value**: `/mnt` (Recommended for production)
- **Used by**: All M3TAL compose stacks

#### `MEDIA_PATH`
- **Description**: The subdirectory within `BASE_STORAGE_PATH` designated for media storage.
- **Default Value**: `./data/media`
- **Example Value**: `/mnt/media`
- **Used by**: All M3TAL compose stacks

#### `CONFIG_PATH`
- **Description**: The subdirectory within `BASE_STORAGE_PATH` designated for M3TAL-specific configuration files (e.g., `users.json`).
- **Default Value**: `./data/config`
- **Example Value**: `/mnt/config`
- **Used by**: All M3TAL compose stacks

#### `DOWNLOADS_PATH`
- **Description**: The subdirectory within `BASE_STORAGE_PATH` designated for downloads.
- **Default Value**: `./data/downloads`
- **Example Value**: `/mnt/downloads`
- **Used by**: All M3TAL compose stacks

### Traefik & Routing

Variables specific to the Traefik reverse proxy gateway.

#### `DOMAIN`
- **Description**: The base domain name used by Traefik for routing rules. Setting this enables routes like `dash.DOMAIN` for the dashboard and `api.DOMAIN` for the API daemon. If Traefik is not in use, it defaults to `localhost`.
- **Default Value**: `localhost`
- **Example Value**: `yourhomelab.com`
- **Used by**: CLI, Traefik gateway, M3TAL Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)

#### `TRAEFIK_WEB_PORT`
- **Description**: The host port Traefik listens on for incoming HTTP (`web`) traffic.
- **Default Value**: `80`
- **Example Value**: `80`
- **Used by**: Traefik gateway

#### `TRAEFIK_WEBHTTPS_PORT`
- **Description**: The host port Traefik listens on for incoming HTTPS (`websecure`) traffic.
- **Default Value**: `443`
- **Example Value**: `443`
- **Used by**: Traefik gateway

#### `TRAEFIK_DASHBOARD_PORT`
- **Description**: The internal container port for Traefik's own administrative dashboard. This port is typically mapped to `127.0.0.1:8081` on the host.
- **Default Value**: `8080`
- **Example Value**: `8080`
- **Used by**: Traefik gateway

### VPN Configuration

Variables for setting up VPN-related services, such as Cloudflared tunnels.

#### `VPN_USER`
- **Description**: The username used for configuring a VPN client, such as Cloudflared.
- **Default Value**: `user`
- **Example Value**: `mycloudflareduser`
- **Used by**: Cloudflared container

#### `VPN_PASSWORD`
- **Description**: The password used for configuring a VPN client, such as Cloudflared.
- **Default Value**: `password`
- **Example Value**: `mysecurecloudflaredpassword`
- **Used by**: Cloudflared container

### System & Container Defaults

General system and Docker container-specific variables.

#### `PUID`
- **Description**: Sets the User ID (UID) for processes running inside M3TAL Docker containers, ensuring proper file permissions on mounted volumes.
- **Default Value**: `1000`
- **Example Value**: `1001` (Often matches the UID of the host user running Docker)
- **Used by**: All M3TAL compose stacks

#### `PGID`
- **Description**: Sets the Group ID (GID) for processes running inside M3TAL Docker containers, ensuring proper file permissions on mounted volumes.
- **Default Value**: `1000`
- **Example Value**: `1001` (Often matches the GID of the host user running Docker)
- **Used by**: All M3TAL compose stacks

#### `TZ`
- **Description**: Sets the timezone for M3TAL Docker containers, ensuring accurate timestamps in logs and internal operations.
- **Default Value**: `America/Denver`
- **Example Value**: `Europe/London`
- **Used by**: All M3TAL compose stacks