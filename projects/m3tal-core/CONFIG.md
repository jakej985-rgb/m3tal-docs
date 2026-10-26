# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables control the configuration and behavior of M3TAL components, including the API, dashboard, and networking.

All environment variables are read from `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` and individual `m3tal config set KEY value` commands. Both the M3TAL CLI and all Docker Compose stacks utilize this file via the `--env-file` option.

## Quick Reference Table

| Variable Name            | Description                                                                   | Default Value     | Example Value                    | Used By        |
|--------------------------|-------------------------------------------------------------------------------|-------------------|----------------------------------|----------------|
| **Core**                 |                                                                               |                   |                                  |                |
| `API_PORT`               | Port for the M3TAL API daemon.                                                | `5050`            | `5050`                           | API daemon     |
| `DASHBOARD_PORT`         | Port for the M3TAL Dashboard.                                                 | `8082`            | `8082`                           | Dashboard      |
| `DASHBOARD_EXPOSE_MODE`  | Controls how the dashboard is exposed: `local` (direct port) or `traefik`. | `local`           | `traefik`                        | Dashboard      |
| `HTTP_PORT`              | Generic HTTP port, primarily used for Traefik.                                | `8080`            | `8080`                           | Traefik        |
| `STATE_DIR`              | Directory for storing M3TAL state data.                                       | `./state`         | `/var/lib/m3tal/state`           | API daemon     |
| `LOG_LEVEL`              | Logging level for M3TAL components.                                           | `info`            | `debug`                          | All            |
| `DEBUG_MODE`             | Enables or disables debug mode for M3TAL components.                          | `false`           | `true`                           | All            |
| `METRICS_ENABLED`        | Enables or disables metrics collection.                                       | `true`            | `false`                          | All            |
| **Authentication**       |                                                                               |                   |                                  |                |
| `DASHBOARD_SECRET`       | Secret key for the dashboard. **Auto-generated on first `m3tal init`**.       | `change_me_immediately` | `a_very_secret_key_generated_by_m3tal` | Dashboard      |
| `API_TOKEN`              | API authentication token. **Auto-generated on first `m3tal init`**.         | `change_me_api_token` | `super_secure_api_token_xyz123` | API daemon     |
| `ADMIN_PASSWORD`         | Password for the default admin user.                                          | `admin_pass`      | `a_strong_password`              | Dashboard      |
| **Network**              |                                                                               |                   |                                  |                |
| `NETWORK_NAME`           | Name of the Docker network M3TAL services will use.                           | `m3tal`           | `m3tal_net`                      | All Docker     |
| `LOCAL_IP`               | Local IP address to bind to.                                                  | `127.0.0.1`       | `192.168.1.100`                  | API daemon     |
| `DOMAIN`                 | The primary domain for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN`. | `localhost`       | `mydomain.com`                   | Traefik, API   |
| `VPN_USER`               | Username for VPN connection.                                                  | `user`            | `myvpnuser`                      | VPN Client     |
| `VPN_PASSWORD`           | Password for VPN connection.                                                  | `password`        | `myvpnpassword`                  | VPN Client     |
| **Storage**              |                                                                               |                   |                                  |                |
| `BASE_STORAGE_PATH`      | Base path for M3TAL storage. Defaults to `/mnt` in production.                | `./data`          | `/mnt`                           | All            |
| `MEDIA_PATH`             | Path for media storage, relative to `BASE_STORAGE_PATH`.                      | `./data/media`    | `/mnt/media`                     | All            |
| `CONFIG_PATH`            | Path for configuration files, relative to `BASE_STORAGE_PATH`.              | `./data/config`   | `/mnt/config`                    | All            |
| `DOWNLOADS_PATH`         | Path for downloads, relative to `BASE_STORAGE_PATH`.                          | `./data/downloads`| `/mnt/downloads`                 | All            |
| **System/User**          |                                                                               |                   |                                  |                |
| `PUID`                   | User ID for Docker container processes.                                       | `1000`            | `1000`                           | Docker Compose |
| `PGID`                   | Group ID for Docker container processes.                                      | `1000`            | `1000`                           | Docker Compose |
| `TZ`                     | Timezone for M3TAL components.                                                | `America/Denver`  | `UTC`                            | All            |
| **Traefik Specific**     |                                                                               |                   |                                  |                |
| `TRAEFIK_WEB_PORT`       | Port Traefik listens on for HTTP traffic.                                     | `80`              | `80`                             | Traefik        |
| `TRAEFIK_WEBHTTPS_PORT`  | Port Traefik listens on for HTTPS traffic.                                    | `443`             | `443`                            | Traefik        |
| `TRAEFIK_DASHBOARD_PORT` | Port Traefik uses internally to access its own dashboard.                     | `8080`            | `8080`                           | Traefik        |

---

## Core

### `API_PORT`

*   **Description:** The port on which the M3TAL API daemon will listen for incoming requests.
*   **Default Value:** `5050`
*   **Example Value:** `5050`
*   **Used By:** `m3tal-api.service`

### `DASHBOARD_PORT`

*   **Description:** The port on which the M3TAL Dashboard will listen internally.
*   **Default Value:** `8082`
*   **Example Value:** `8082`
*   **Used By:** `m3tal-dashboard` container

### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Controls how the M3TAL Dashboard is made accessible.
    *   `local`: Exposes the dashboard directly via a host port mapping (default behavior). Access via `http://HOST_IP:DASHBOARD_PORT`.
    *   `traefik`: Exposes the dashboard through the Traefik reverse proxy. Access via `http://dash.DOMAIN`.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:** Dashboard container, Traefik routing configuration.

### `HTTP_PORT`

*   **Description:** A general HTTP port, primarily used by Traefik for its entry point.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** Traefik container.

### `STATE_DIR`

*   **Description:** The directory where M3TAL stores its state database and other persistent data.
*   **Default Value:** `./state` (relative to the current directory where M3TAL is run, typically `/opt/m3tal/stack` for production)
*   **Example Value:** `/var/lib/m3tal/state`
*   **Used By:** `m3tal-api.service`, M3TAL Dashboard container.

### `LOG_LEVEL`

*   **Description:** Sets the logging verbosity for all M3TAL components.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:** All M3TAL components (CLI, API, Dashboard).

### `DEBUG_MODE`

*   **Description:** Enables or disables debug mode for M3TAL components, which may increase logging verbosity or enable additional diagnostic features.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:** All M3TAL components.

### `METRICS_ENABLED`

*   **Description:** Enables or disables the collection and exposure of Prometheus metrics by M3TAL components.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:** All M3TAL components.

## Authentication

### `DASHBOARD_SECRET`

*   **Description:** A secret key used for signing session cookies and other security-sensitive operations within the M3TAL Dashboard. **This variable is auto-generated by `m3tal init` and should not be set manually unless rotating the secret.**
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_very_secret_key_generated_by_m3tal`
*   **Used By:** `m3tal-dashboard` container.

### `API_TOKEN`

*   **Description:** An API token used for authenticating requests to the M3TAL API. **This variable is auto-generated by `m3tal init` and should not be set manually unless rotating the token.**
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `super_secure_api_token_xyz123`
*   **Used By:** `m3tal-api.service`, any services that need to authenticate with the API.

### `ADMIN_PASSWORD`

*   **Description:** The password for the default administrative user of the M3TAL Dashboard.
*   **Default Value:** `admin_pass`
*   **Example Value:** `a_strong_password`
*   **Used By:** `m3tal-dashboard` container.

## Network

### `NETWORK_NAME`

*   **Description:** The name of the Docker network that M3TAL services will connect to.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal_net`
*   **Used By:** All M3TAL Docker services.

### `LOCAL_IP`

*   **Description:** The local IP address that certain services might bind to.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `192.168.1.100`
*   **Used By:** `m3tal-api.service`.

### `DOMAIN`

*   **Description:** The primary domain name for your M3TAL instance. Setting this variable enables Traefik to correctly configure routing rules for `api.DOMAIN` and `dash.DOMAIN`.
*   **Default Value:** `localhost`
*   **Example Value:** `mydomain.com`
*   **Used By:** Traefik routing configuration, `m3tal-api.service` (for generating correct API URLs), `m3tal-dashboard` container.

### `VPN_USER`

*   **Description:** The username for establishing a VPN connection.
*   **Default Value:** `user`
*   **Example Value:** `myvpnuser`
*   **Used By:** VPN client component (if configured).

### `VPN_PASSWORD`

*   **Description:** The password for establishing a VPN connection.
*   **Default Value:** `password`
*   **Example Value:** `myvpnpassword`
*   **Used By:** VPN client component (if configured).

## Storage

### `BASE_STORAGE_PATH`

*   **Description:** The base directory where M3TAL will store all its persistent data, including configuration, media, and downloads. **In production deployments, this defaults to `/mnt`.**
*   **Default Value:** `./data`
*   **Example Value:** `/mnt`
*   **Used By:** All M3TAL components and Docker containers.

### `MEDIA_PATH`

*   **Description:** The directory for storing media files, relative to `BASE_STORAGE_PATH`.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/media`
*   **Used By:** All M3TAL components and Docker containers.

### `CONFIG_PATH`

*   **Description:** The directory for storing M3TAL configuration files, relative to `BASE_STORAGE_PATH`.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/config`
*   **Used By:** All M3TAL components and Docker containers.

### `DOWNLOADS_PATH`

*   **Description:** The directory for storing downloaded files, relative to `BASE_STORAGE_PATH`.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/downloads`
*   **Used By:** All M3TAL components and Docker containers.

## System / User

### `PUID`

*   **Description:** The User ID (UID) that Docker container processes will run as. This is important for file permissions.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:** Docker Compose for `m3tal-dashboard` and other potential containers.

### `PGID`

*   **Description:** The Group ID (GID) that Docker container processes will run as. This is important for file permissions.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:** Docker Compose for `m3tal-dashboard` and other potential containers.

### `TZ`

*   **Description:** The timezone to be used by M3TAL components. This affects log timestamps and any time-based operations.
*   **Default Value:** `America/Denver`
*   **Example Value:** `UTC`
*   **Used By:** All M3TAL components and Docker containers.

## Traefik Specific

### `TRAEFIK_WEB_PORT`

*   **Description:** The port on the host machine that Traefik will use as its HTTP entry point.
*   **Default Value:** `80`
*   **Example Value:** `80`
*   **Used By:** Traefik container.

### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The port on the host machine that Traefik will use as its HTTPS entry point.
*   **Default Value:** `443`
*   **Example Value:** `443`
*   **Used By:** Traefik container.

### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The internal port on which Traefik's own dashboard service is exposed. This is typically accessed locally via `127.0.0.1:8081`.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** Traefik container.