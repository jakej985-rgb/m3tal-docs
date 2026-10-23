# M3TAL Environment Variables Reference

All M3TAL configuration is managed via environment variables, read from `/etc/m3tal/.env` by both the CLI and all Docker Compose stacks via `--env-file`. This file is automatically managed by the `m3tal config wizard` and `m3tal config set` commands.

The following table provides a quick reference for all available environment variables.

| Variable Name        | Description                                                                                                   | Default Value       | Example Value                                    | Used By                                                                   |
|----------------------|---------------------------------------------------------------------------------------------------------------|---------------------|--------------------------------------------------|---------------------------------------------------------------------------|
| **Core**             |                                                                                                               |                     |                                                  |                                                                           |
| `API_PORT`           | The port the M3TAL API daemon listens on.                                                                     | `5050`              | `5050`                                           | CLI, API Daemon, Dashboard (via `GO_API_URL`)                             |
| `DASHBOARD_PORT`     | The port the M3TAL dashboard container listens on.                                                            | `8082`              | `8082`                                           | Dashboard                                                                 |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed: `local` (direct port binding) or `traefik` (via Traefik routing). | `local`             | `traefik`                                        | CLI, Dashboard                                                            |
| `HTTP_PORT`          | The internal HTTP port for services. Generally not user-configurable.                                           | `8080`              | `8080`                                           | API Daemon                                                                |
| `STATE_DIR`          | The directory for the M3TAL state database and other internal data.                                           | `./state`           | `/var/lib/m3tal/state`                           | API Daemon, Dashboard                                                     |
| `LOG_LEVEL`          | The logging level for M3TAL services.                                                                         | `info`              | `debug`                                          | API Daemon, Dashboard                                                     |
| `DEBUG_MODE`         | Enables debug mode for M3TAL services.                                                                        | `false`             | `true`                                           | API Daemon, Dashboard                                                     |
| `METRICS_ENABLED`    | Enables or disables metrics collection for M3TAL services.                                                    | `true`              | `false`                                          | API Daemon                                                                |
| **Auth**             |                                                                                                               |                     |                                                  |                                                                           |
| `DASHBOARD_SECRET`   | Secret key for session management in the dashboard. **Auto-generated on first `m3tal init`.**                | `change_me_immediately` | `a_very_secure_random_string`                    | Dashboard                                                                 |
| `API_TOKEN`          | API token for authenticating requests to the M3TAL API. **Auto-generated on first `m3tal init`.**            | `change_me_api_token` | `another_secure_random_token`                    | CLI, API Daemon (for internal auth)                                       |
| `ADMIN_PASSWORD`     | The password for the initial admin user of the dashboard.                                                     | `admin_pass`        | `a_strong_password`                              | Dashboard                                                                 |
| **Network**          |                                                                                                               |                     |                                                  |                                                                           |
| `NETWORK_NAME`       | The name of the Docker network M3TAL services will use.                                                       | `m3tal`             | `m3tal`                                          | CLI, Docker Compose Stacks                                                |
| `LOCAL_IP`           | The local IP address M3TAL services should bind to.                                                           | `127.0.0.1`         | `127.0.0.1`                                      | API Daemon, Dashboard                                                     |
| `DOMAIN`             | The domain name used for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes.         | `localhost`         | `m3tal.yourdomain.com`                           | Traefik, API Daemon (for routing)                                         |
| **Storage**          |                                                                                                               |                     |                                                  |                                                                           |
| `BASE_STORAGE_PATH`  | The base directory for storing M3TAL data, including media, configurations, and downloads. **Defaults to `/mnt` in production deployments.** | `./data`            | `/mnt`                                           | CLI, API Daemon, Dashboard                                                |
| `MEDIA_PATH`         | The subdirectory within `BASE_STORAGE_PATH` for storing media files.                                          | `./data/media`      | `/mnt/media`                                     | API Daemon, Dashboard                                                     |
| `CONFIG_PATH`        | The subdirectory within `BASE_STORAGE_PATH` for storing M3TAL configuration files.                            | `./data/config`     | `/mnt/config`                                    | API Daemon, Dashboard                                                     |
| `DOWNLOADS_PATH`     | The subdirectory within `BASE_STORAGE_PATH` for storing downloaded files.                                     | `./data/downloads`  | `/mnt/downloads`                                 | API Daemon, Dashboard                                                     |
| **User/Group IDs**   |                                                                                                               |                     |                                                  |                                                                           |
| `PUID`               | The user ID to run containers as.                                                                             | `1000`              | `1000`                                           | Docker Compose Stacks                                                     |
| `PGID`               | The group ID to run containers as.                                                                            | `1000`              | `1000`                                           | Docker Compose Stacks                                                     |
| **System**           |                                                                                                               |                     |                                                  |                                                                           |
| `TZ`                 | The timezone to use for M3TAL services.                                                                       | `America/Denver`    | `UTC`                                            | API Daemon, Dashboard                                                     |
| **Traefik**          |                                                                                                               |                     |                                                  |                                                                           |
| `TRAEFIK_WEB_PORT`   | The host port Traefik listens on for HTTP traffic.                                                            | `80`                | `80`                                             | Traefik                                                                   |
| `TRAEFIK_WEBHTTPS_PORT` | The host port Traefik listens on for HTTPS traffic.                                                         | `443`               | `443`                                            | Traefik                                                                   |
| `TRAEFIK_DASHBOARD_PORT` | The host port Traefik exposes its own dashboard on (accessed via `127.0.0.1:8081`).                         | `8080`              | `8080`                                           | Traefik                                                                   |
| **VPN**              |                                                                                                               |                     |                                                  |                                                                           |
| `VPN_USER`           | Username for the VPN connection.                                                                              | `user`              | `myvpnuser`                                      | VPN Client (if configured)                                                |
| `VPN_PASSWORD`       | Password for the VPN connection.                                                                              | `password`          | `mysecurevpnpassword`                            | VPN Client (if configured)                                                |

---

## Detailed Descriptions

### Core Variables

*   **`API_PORT`**: Specifies the port on which the M3TAL API daemon listens for incoming requests. This port is typically exposed internally within the Docker network.
    *   **Default Value**: `5050`
    *   **Example Value**: `5050`
    *   **Used By**: CLI, API Daemon, Dashboard (via `GO_API_URL`)

*   **`DASHBOARD_PORT`**: Defines the internal port that the M3TAL dashboard container listens on.
    *   **Default Value**: `8082`
    *   **Example Value**: `8082`
    *   **Used By**: Dashboard

*   **`DASHBOARD_EXPOSE_MODE`**: Controls how the M3TAL dashboard is made accessible.
    *   `local`: The dashboard is exposed via a direct port binding to `${DASHBOARD_PORT:-8082}` on the host. Access is typically via `http://HOST_IP:${DASHBOARD_PORT}` or `http://localhost:${DASHBOARD_PORT}`. This mode does not require Traefik.
    *   `traefik`: The dashboard is exposed through Traefik. Traefik routes traffic from `dash.${DOMAIN}` to the dashboard container. This mode requires Traefik to be running.
    *   **Default Value**: `local`
    *   **Example Value**: `traefik`
    *   **Used By**: CLI, Dashboard

*   **`HTTP_PORT`**: The internal HTTP port used by services. This is generally an internal setting and not intended for direct user modification.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Used By**: API Daemon

*   **`STATE_DIR`**: This variable specifies the directory where the M3TAL state database (`state.db`) and other internal runtime data are stored.
    *   **Default Value**: `./state`
    *   **Example Value**: `/var/lib/m3tal/state`
    *   **Used By**: API Daemon, Dashboard

*   **`LOG_LEVEL`**: Sets the verbosity of logging for M3TAL services. Options typically include `debug`, `info`, `warn`, `error`.
    *   **Default Value**: `info`
    *   **Example Value**: `debug`
    *   **Used By**: API Daemon, Dashboard

*   **`DEBUG_MODE`**: When set to `true`, this variable enables enhanced debugging features within M3TAL services.
    *   **Default Value**: `false`
    *   **Example Value**: `true`
    *   **Used By**: API Daemon, Dashboard

*   **`METRICS_ENABLED`**: Toggles the collection and exposure of operational metrics for M3TAL services.
    *   **Default Value**: `true`
    *   **Example Value**: `false`
    *   **Used By**: API Daemon

### Auth Variables

*   **`DASHBOARD_SECRET`**: A critical secret key used for encrypting session cookies and other sensitive data within the M3TAL dashboard. **This variable is auto-generated on the first `m3tal init` and should not be set manually unless you are rotating it for security purposes.**
    *   **Default Value**: `change_me_immediately`
    *   **Example Value**: `a_very_secure_random_string`
    *   **Used By**: Dashboard

*   **`API_TOKEN`**: An authentication token used by the CLI and potentially other clients to interact with the M3TAL API. **This variable is auto-generated on the first `m3tal init` and should not be set manually unless you are rotating it for security purposes.**
    *   **Default Value**: `change_me_api_token`
    *   **Example Value**: `another_secure_random_token`
    *   **Used By**: CLI, API Daemon (for internal authentication)

*   **`ADMIN_PASSWORD`**: Sets the initial password for the administrator account in the M3TAL dashboard.
    *   **Default Value**: `admin_pass`
    *   **Example Value**: `a_strong_password`
    *   **Used By**: Dashboard

### Network Variables

*   **`NETWORK_NAME`**: Specifies the name of the Docker network that M3TAL services will join. This ensures proper inter-container communication.
    *   **Default Value**: `m3tal`
    *   **Example Value**: `m3tal`
    *   **Used By**: CLI, Docker Compose Stacks

*   **`LOCAL_IP`**: The local IP address that M3TAL services should bind to. This is often used for internal communication between services or for exposing services on the host.
    *   **Default Value**: `127.0.0.1`
    *   **Example Value**: `127.0.0.1`
    *   **Used By**: API Daemon, Dashboard

*   **`DOMAIN`**: This variable is crucial for Traefik routing. When set to a valid domain name (e.g., `m3tal.yourdomain.com`), Traefik will be configured to route traffic to services based on subdomains like `api.DOMAIN` and `dash.DOMAIN`. If set to `localhost`, Traefik will primarily handle local routing.
    *   **Default Value**: `localhost`
    *   **Example Value**: `m3tal.yourdomain.com`
    *   **Used By**: Traefik, API Daemon (for routing configuration)

### Storage Variables

*   **`BASE_STORAGE_PATH`**: This is the primary directory where M3TAL stores all its persistent data, including media files, configuration, and downloads. **In production deployments, this defaults to `/mnt` for better separation from application code. In template or development environments, it might default to `./data`.**
    *   **Default Value**: `./data`
    *   **Example Value**: `/mnt`
    *   **Used By**: CLI, API Daemon, Dashboard

*   **`MEDIA_PATH`**: The specific subdirectory within `BASE_STORAGE_PATH` designated for storing media files (e.g., images, videos).
    *   **Default Value**: `./data/media`
    *   **Example Value**: `/mnt/media`
    *   **Used By**: API Daemon, Dashboard

*   **`CONFIG_PATH`**: The subdirectory within `BASE_STORAGE_PATH` used for storing M3TAL's configuration files.
    *   **Default Value**: `./data/config`
    *   **Example Value**: `/mnt/config`
    *   **Used By**: API Daemon, Dashboard

*   **`DOWNLOADS_PATH`**: The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.
    *   **Default Value**: `./data/downloads`
    *   **Example Value**: `/mnt/downloads`
    *   **Used By**: API Daemon, Dashboard

### User/Group ID Variables

*   **`PUID`**: The User ID (UID) that Docker containers should run as. This is important for file permissions on the host system, ensuring containers can read and write to mounted volumes correctly.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker Compose Stacks

*   **`PGID`**: The Group ID (GID) that Docker containers should run as. Similar to `PUID`, this ensures correct file permissions for mounted volumes.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker Compose Stacks

### System Variables

*   **`TZ`**: Sets the timezone for all M3TAL services. This ensures that timestamps in logs and other time-sensitive data are accurate.
    *   **Default Value**: `America/Denver`
    *   **Example Value**: `UTC`
    *   **Used By**: API Daemon, Dashboard

### Traefik Variables

*   **`TRAEFIK_WEB_PORT`**: The port on the host machine that Traefik will listen on for incoming HTTP traffic. This is typically port 80.
    *   **Default Value**: `80`
    *   **Example Value**: `80`
    *   **Used By**: Traefik

*   **`TRAEFIK_WEBHTTPS_PORT`**: The port on the host machine that Traefik will listen on for incoming HTTPS traffic. This is typically port 443.
    *   **Default Value**: `443`
    *   **Example Value**: `443`
    *   **Used By**: Traefik

*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik itself uses internally to expose its administrative dashboard. This is usually accessed via `http://127.0.0.1:8081` or `http://host.docker.internal:8081` for local access.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Used By**: Traefik

### VPN Variables

*   **`VPN_USER`**: The username required to establish a VPN connection. This is used if you configure M3TAL to route traffic through a VPN.
    *   **Default Value**: `user`
    *   **Example Value**: `myvpnuser`
    *   **Used By**: VPN Client (if configured)

*   **`VPN_PASSWORD`**: The password required to establish a VPN connection. This is used if you configure M3TAL to route traffic through a VPN.
    *   **Default Value**: `password`
    *   **Example Value**: `mysecurevpnpassword`
    *   **Used By**: VPN Client (if configured)