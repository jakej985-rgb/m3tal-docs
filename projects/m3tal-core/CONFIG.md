# M3TAL Environment Variables Reference

This document details all environment variables used by the M3TAL ecosystem. These variables are crucial for configuring M3TAL's behavior, networking, storage, and security.

All environment variables are read from `/etc/m3tal/.env` by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option. This ensures a unified and centralized configuration for your M3TAL deployment.

## Quick Reference Table

| Variable Name           | Description                                                                 | Default Value      | Example Value                                   | Used By                                       |
|-------------------------|-----------------------------------------------------------------------------|--------------------|-------------------------------------------------|-----------------------------------------------|
| **Core**                |                                                                             |                    |                                                 |                                               |
| `STATE_DIR`             | Directory for M3TAL state and database.                                     | `./state`          | `/var/lib/m3tal/state`                          | CLI, API Daemon, Dashboard                    |
| `LOG_LEVEL`             | Logging level for M3TAL services.                                           | `info`             | `debug`                                         | CLI, API Daemon, Dashboard                    |
| `PUID`                  | User ID for container processes.                                            | `1000`             | `1000`                                          | Docker Containers                             |
| `PGID`                  | Group ID for container processes.                                           | `1000`             | `1000`                                          | Docker Containers                             |
| `TZ`                    | Timezone for M3TAL services.                                                | `America/Denver`   | `UTC`                                           | Docker Containers                             |
| `DEBUG_MODE`            | Enables debug mode for M3TAL.                                               | `false`            | `true`                                          | CLI, API Daemon                               |
| `METRICS_ENABLED`       | Enables Prometheus metrics for M3TAL services.                              | `true`             | `false`                                         | API Daemon                                    |
| **Authentication**      |                                                                             |                    |                                                 |                                               |
| `DASHBOARD_SECRET`      | Secret key for dashboard session management. **Auto-generated on first `m3tal init`.** | `change_me_immediately` | `a-very-long-and-random-secret-string`          | Dashboard Container                           |
| `API_TOKEN`             | API token for authentication with the M3TAL API. **Auto-generated on first `m3tal init`.** | `change_me_api_token` | `a-secure-api-token-for-clients`                | CLI, External Clients                         |
| `ADMIN_PASSWORD`        | Password for the default admin user of the dashboard.                       | `admin_pass`       | `a-strong-admin-password`                       | Dashboard Container                           |
| **Network**             |                                                                             |                    |                                                 |                                               |
| `API_PORT`              | Port the M3TAL API daemon listens on.                                       | `5050`             | `5050`                                          | API Daemon, Dashboard, CLI                    |
| `DASHBOARD_PORT`        | Port the M3TAL dashboard listens on.                                        | `8082`             | `8082`                                          | Dashboard Container                           |
| `HTTP_PORT`             | Internal HTTP port, often used for API communication.                       | `8080`             | `8080`                                          | API Daemon (internal)                         |
| `LOCAL_IP`              | The local IP address of the host machine.                                   | `127.0.0.1`        | `192.168.1.100`                                 | API Daemon (for host.docker.internal)         |
| `NETWORK_NAME`          | Name of the Docker network M3TAL services will connect to.                  | `m3tal`            | `m3tal_network`                                 | Docker Compose                                |
| `DOMAIN`                | The primary domain for M3TAL services (e.g., Traefik routing).              | `localhost`        | `m3tal.yourdomain.com`                          | Traefik, Dashboard (Traefik mode)             |
| `TRAEFIK_WEB_PORT`      | The port Traefik listens on for HTTP traffic.                               | `80`               | `80`                                            | Traefik Container                             |
| `TRAEFIK_WEBHTTPS_PORT` | The port Traefik listens on for HTTPS traffic.                              | `443`              | `443`                                           | Traefik Container                             |
| `TRAEFIK_DASHBOARD_PORT`| The internal port Traefik uses to access its own dashboard.                 | `8080`             | `8080`                                          | Traefik Container                             |
| **Storage**             |                                                                             |                    |                                                 |                                               |
| `BASE_STORAGE_PATH`     | Base directory for all M3TAL data storage. **Defaults to `/mnt` in production.** | `./data`           | `/mnt`                                          | Docker Containers, CLI                        |
| `MEDIA_PATH`            | Path within `BASE_STORAGE_PATH` for media files.                            | `./data/media`     | `/mnt/media`                                    | Docker Containers                             |
| `CONFIG_PATH`           | Path within `BASE_STORAGE_PATH` for configuration files.                    | `./data/config`    | `/mnt/config`                                   | Docker Containers                             |
| `DOWNLOADS_PATH`        | Path within `BASE_STORAGE_PATH` for downloads.                              | `./data/downloads` | `/mnt/downloads`                                | Docker Containers                             |
| **Traefik**             |                                                                             |                    |                                                 |                                               |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed: `local` or `traefik`.              | `local`            | `traefik`                                       | CLI, Dashboard Container                      |
| **VPN**                 |                                                                             |                    |                                                 |                                               |
| `VPN_USER`              | Username for the VPN connection.                                            | `user`             | `myvpnuser`                                     | VPN Client (if configured)                    |
| `VPN_PASSWORD`          | Password for the VPN connection.                                            | `password`         | `mysecurevpnpassword`                           | VPN Client (if configured)                    |

---

## Variable Descriptions

### Core

*   **`STATE_DIR`**
    *   **Description:** Specifies the directory where M3TAL stores its state, including the SQLite database and other persistent data.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** CLI, API Daemon, Dashboard Container

*   **`LOG_LEVEL`**
    *   **Description:** Controls the verbosity of logging output from M3TAL services. Common values include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** CLI, API Daemon, Dashboard Container

*   **`PUID`**
    *   **Description:** The User ID (UID) to be used by processes running inside Docker containers. This helps manage file permissions on the host system.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Docker Containers (specified in Docker Compose files)

*   **`PGID`**
    *   **Description:** The Group ID (GID) to be used by processes running inside Docker containers. This helps manage file permissions on the host system.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Docker Containers (specified in Docker Compose files)

*   **`TZ`**
    *   **Description:** The timezone to be used by M3TAL services. This affects timestamp logging and any time-sensitive operations.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `UTC`
    *   **Used By:** Docker Containers (specified in Docker Compose files)

*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug mode for M3TAL. This may provide more verbose output or enable specific debugging features.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** CLI, API Daemon

*   **`METRICS_ENABLED`**
    *   **Description:** Enables or disables the Prometheus metrics endpoint for M3TAL services, allowing for monitoring and collection of performance data.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** API Daemon

### Authentication

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used for securing dashboard sessions. **This variable is auto-generated on the first `m3tal init` command.** Users should *not* manually set this unless rotating the secret.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a-very-long-and-random-secret-string`
    *   **Used By:** Dashboard Container

*   **`API_TOKEN`**
    *   **Description:** A token used for authenticating with the M3TAL API. **This variable is auto-generated on the first `m3tal init` command.** Users should *not* manually set this unless rotating the token.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a-secure-api-token-for-clients`
    *   **Used By:** CLI, External Clients

*   **`ADMIN_PASSWORD`**
    *   **Description:** The password for the default administrative user of the M3TAL dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `a-strong-admin-password`
    *   **Used By:** Dashboard Container

### Network

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon (Go binary) listens for incoming requests.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** API Daemon, Dashboard Container, CLI

*   **`DASHBOARD_PORT`**
    *   **Description:** The port on which the M3TAL dashboard container listens for incoming requests.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** Dashboard Container

*   **`HTTP_PORT`**
    *   **Description:** An internal HTTP port, often used for communication between services or for internal API endpoints.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** API Daemon (internal)

*   **`LOCAL_IP`**
    *   **Description:** The IP address of the host machine that Docker can use to resolve `host.docker.internal`. This is crucial for containers to communicate with the host.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** API Daemon (for `host.docker.internal` resolution in Docker Compose)

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will join. This allows for inter-container communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_network`
    *   **Used By:** Docker Compose

*   **`DOMAIN`**
    *   **Description:** The primary domain name configured for your M3TAL deployment. This is used by Traefik for routing rules, enabling access to services like `api.DOMAIN` and `dash.DOMAIN`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `m3tal.yourdomain.com`
    *   **Used By:** Traefik, Dashboard Container (when `DASHBOARD_EXPOSE_MODE=traefik`)

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The port on which Traefik acts as the HTTP entry point, accepting incoming web traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik Container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The port on which Traefik acts as the HTTPS entry point, accepting incoming secure web traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik Container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own dashboard is accessible.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik Container

### Storage

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory where M3TAL stores all its persistent data, including configuration, media, and downloads. **In production deployments, this defaults to `/mnt`. In development or template setups, it may default to `./data`.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt`
    *   **Used By:** Docker Containers, CLI

*   **`MEDIA_PATH`**
    *   **Description:** The sub-path within `BASE_STORAGE_PATH` dedicated to storing media files.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/media`
    *   **Used By:** Docker Containers

*   **`CONFIG_PATH`**
    *   **Description:** The sub-path within `BASE_STORAGE_PATH` where configuration files are stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Used By:** Docker Containers

*   **`DOWNLOADS_PATH`**
    *   **Description:** The sub-path within `BASE_STORAGE_PATH` used for storing downloaded files.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/downloads`
    *   **Used By:** Docker Containers

### Traefik

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Determines how the M3TAL dashboard is made accessible.
        *   `local`: Exposes the dashboard directly via a port binding (e.g., `http://localhost:8082`). This mode does not require Traefik.
        *   `traefik`: Exposes the dashboard through Traefik, making it accessible via `http://dash.DOMAIN`. This mode requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** CLI, Dashboard Container

### VPN

*   **`VPN_USER`**
    *   **Description:** The username to be used when establishing a VPN connection. This is applicable if you are configuring M3TAL to operate behind a VPN.
    *   **Default Value:** `user`
    *   **Example Value:** `myvpnuser`
    *   **Used By:** VPN Client (if configured)

*   **`VPN_PASSWORD`**
    *   **Description:** The password to be used when establishing a VPN connection. This is applicable if you are configuring M3TAL to operate behind a VPN.
    *   **Default Value:** `password`
    *   **Example Value:** `mysecurevpnpassword`
    *   **Used By:** VPN Client (if configured)