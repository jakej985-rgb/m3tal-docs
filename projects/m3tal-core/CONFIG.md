# M3TAL Environment Variables Reference

All M3TAL environment variables are read from `/etc/m3tal/.env` by both the CLI and all compose stacks via `--env-file`. This file is managed by the `m3tal config wizard` command.

---

## Quick Reference

| Variable Name          | Description                                                         | Default Value       |
|------------------------|---------------------------------------------------------------------|---------------------|
| **Core**               |                                                                     |                     |
| `API_PORT`             | Port for the M3TAL API daemon.                                      | `5050`              |
| `HTTP_PORT`            | General HTTP port used by some services.                            | `8080`              |
| `STATE_DIR`            | Directory to store M3TAL's state (e.g., database).                  | `./state`           |
| `LOG_LEVEL`            | Logging level for M3TAL services.                                   | `info`              |
| `DEBUG_MODE`           | Enables debug mode for M3TAL services.                              | `false`             |
| `METRICS_ENABLED`      | Enables Prometheus metrics collection.                                | `true`              |
| **Auth**               |                                                                     |                     |
| `DASHBOARD_SECRET`     | Secret key for the dashboard. **Auto-generated on first `m3tal init`.** | `change_me_immediately` |
| `API_TOKEN`            | API token for authentication. **Auto-generated on first `m3tal init`.** | `change_me_api_token` |
| `ADMIN_PASSWORD`       | Password for the initial admin user.                                | `admin_pass`        |
| **Network**            |                                                                     |                     |
| `NETWORK_NAME`         | Name of the Docker network M3TAL services will join.                | `m3tal`             |
| `LOCAL_IP`             | The local IP address of the host machine.                           | `127.0.0.1`         |
| `DOMAIN`               | The domain name for accessing M3TAL services via Traefik.           | `localhost`         |
| `VPN_USER`             | Username for the VPN connection.                                    | `user`              |
| `VPN_PASSWORD`         | Password for the VPN connection.                                    | `password`          |
| **Storage**            |                                                                     |                     |
| `BASE_STORAGE_PATH`    | Base path for all M3TAL storage. **Defaults to `/mnt` in production.** | `./data`            |
| `MEDIA_PATH`           | Path for media storage within `BASE_STORAGE_PATH`.                  | `./data/media`      |
| `CONFIG_PATH`          | Path for configuration files within `BASE_STORAGE_PATH`.            | `./data/config`     |
| `DOWNLOADS_PATH`       | Path for download storage within `BASE_STORAGE_PATH`.               | `./data/downloads`  |
| `PUID`                 | User ID for container processes.                                    | `1000`              |
| `PGID`                 | Group ID for container processes.                                   | `1000`              |
| **Dashboard**          |                                                                     |                     |
| `DASHBOARD_PORT`       | Port the M3TAL dashboard listens on internally.                     | `8082`              |
| `DASHBOARD_EXPOSE_MODE`| How the dashboard is exposed: `local` (direct port) or `traefik`. | `local`             |
| **Traefik**            |                                                                     |                     |
| `TRAEFIK_WEB_PORT`     | Port Traefik listens on for HTTP traffic.                           | `80`                |
| `TRAEFIK_WEBHTTPS_PORT`| Port Traefik listens on for HTTPS traffic.                          | `443`               |
| `TRAEFIK_DASHBOARD_PORT`| Port Traefik's own dashboard listens on.                            | `8080`              |
| **System**             |                                                                     |                     |
| `TZ`                   | Timezone for M3TAL services.                                        | `America/Denver`    |

---

## Detailed Environment Variable Reference

All environment variables are intended to be managed via the `m3tal config wizard` and `m3tal config set KEY value` commands. They are stored in `/etc/m3tal/.env` and read by M3TAL services.

### Core

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon (`m3tal-api.service`) will listen.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard` (via `GO_API_URL`)

*   **`HTTP_PORT`**
    *   **Description:** A general HTTP port. In the context of Traefik, this can influence where some services are exposed if not otherwise specified.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik (as a potential internal port for services), `m3tal-api.service`

*   **`STATE_DIR`**
    *   **Description:** The directory where M3TAL stores its persistent state, including the SQLite database.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`LOG_LEVEL`**
    *   **Description:** Controls the verbosity of logging output from M3TAL services. Common values include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug mode for M3TAL services, potentially increasing log verbosity or enabling additional debugging features.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `m3tal-api.service`, `m3tal-dashboard`

*   **`METRICS_ENABLED`**
    *   **Description:** Enables or disables the collection and exposition of Prometheus metrics for M3TAL services.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `m3tal-api.service`

### Auth

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used for session management and security within the M3TAL dashboard. **This variable is auto-generated on the first `m3tal init` and should NOT be set manually unless you intend to rotate it.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `s3cr3tK3yF0rS3cur1ty`
    *   **Used By:** `m3tal-dashboard`

*   **`API_TOKEN`**
    *   **Description:** An API token used for authentication with the M3TAL API. **This variable is auto-generated on the first `m3tal init` and should NOT be set manually unless you intend to rotate it.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a1b2c3d4e5f6g7h8i9j0`
    *   **Used By:** `m3tal-api.service` (internally), external clients accessing the API.

*   **`ADMIN_PASSWORD`**
    *   **Description:** The password for the initial administrative user account created during setup.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MyStr0ngP@ssw0rd`
    *   **Used By:** `m3tal-api.service` (for initial user creation)

### Network

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will use for communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_internal_net`
    *   **Used By:** Docker Compose files (`m3tal-compose.yml`, `routing-compose.yml`)

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the host machine. This is often used for internal service discovery or for services needing to communicate with the host.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** `m3tal-dashboard` (for `GO_API_URL`), Traefik configuration.

*   **`DOMAIN`**
    *   **Description:** The base domain name used for routing M3TAL services via Traefik. Setting this variable enables `dash.DOMAIN` and `api.DOMAIN` routes.
    *   **Default Value:** `localhost`
    *   **Example Value:** `my.m3tal.site`
    *   **Used By:** Traefik configuration (`routing-compose.yml`, `dynamic/api.yml`), `m3tal-compose.traefik.yml`

*   **`VPN_USER`**
    *   **Description:** Username for a configured VPN connection, if M3TAL is set up to use one.
    *   **Default Value:** `user`
    *   **Example Value:** `vpnuser123`
    *   **Used By:** VPN client configuration (e.g., cloudflared if configured with VPN).

*   **`VPN_PASSWORD`**
    *   **Description:** Password for a configured VPN connection, if M3TAL is set up to use one.
    *   **Default Value:** `password`
    *   **Example Value:** `vPnP@ssw0rd987`
    *   **Used By:** VPN client configuration (e.g., cloudflared if configured with VPN).

### Storage

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory where M3TAL will store all its persistent data, including media, configuration, and downloads. **In production deployments, this defaults to `/mnt`. In template or development environments, it may default to `./data`.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt`
    *   **Used By:** Docker Compose files for volume mounts (`m3tal-dashboard`, other user stacks).

*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal/media`
    *   **Used By:** Docker Compose files (volume mounts).

*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL's configuration files (excluding `.env`) are stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal/config`
    *   **Used By:** Docker Compose files (volume mounts).

*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal/downloads`
    *   **Used By:** Docker Compose files (volume mounts).

*   **`PUID`**
    *   **Description:** The User ID (UID) that container processes should run as. This is important for file permissions when mounting volumes from the host.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard` and other Docker containers.

*   **`PGID`**
    *   **Description:** The Group ID (GID) that container processes should run as. This is important for file permissions when mounting volumes from the host.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** `m3tal-dashboard` and other Docker containers.

### Dashboard

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL dashboard service listens.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** `m3tal-dashboard` (internal listening port), Traefik configuration.

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Determines how the M3TAL dashboard is made accessible.
        *   `local`: The dashboard is exposed directly via a port binding (controlled by `DASHBOARD_PORT`). Accessible via `http://HOST_IP:DASHBOARD_PORT` or `http://localhost:DASHBOARD_PORT`. No Traefik is required for this mode.
        *   `traefik`: The dashboard is exposed via Traefik. Accessible via `http://dash.DOMAIN`. Traefik must be running and configured to route to the dashboard.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** `m3tal-dashboard` (via compose override file selection), Traefik configuration.

### Traefik

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik will use for its HTTP entry point. This is typically port 80.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** `routing-compose.yml` (Traefik container port mapping).

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik will use for its HTTPS entry point. This is typically port 443.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** `routing-compose.yml` (Traefik container port mapping).

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik listens on for its own dashboard interface. This is usually exposed on `127.0.0.1:8080` by default.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `routing-compose.yml` (Traefik container port mapping `127.0.0.1:${TRAEFIK_DASHBOARD_PORT}:8080`).

### System

*   **`TZ`**
    *   **Description:** The timezone setting for M3TAL services, ensuring correct timestamp logging and scheduling.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `UTC`
    *   **Used By:** `m3tal-dashboard`, `m3tal-api.service` (if configured to use it).