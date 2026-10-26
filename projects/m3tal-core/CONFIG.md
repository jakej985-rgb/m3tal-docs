# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are crucial for configuring M3TAL's behavior, networking, storage, and security.

All M3TAL environment variables are read from the `/etc/m3tal/.env` file. This file is managed by the `m3tal config wizard` and can also be updated using `m3tal config set KEY value`. Both the M3TAL CLI and all Docker Compose stacks utilize this file via the `--env-file` option, ensuring a unified configuration across the system.

**Quick Reference Table**

| Variable Name             | Description                                                                                                    | Default Value      | Example Value                                  | Used By                                                                          |
| :------------------------ | :------------------------------------------------------------------------------------------------------------- | :----------------- | :--------------------------------------------- | :------------------------------------------------------------------------------- |
| **Core**                  |                                                                                                                |                    |                                                |                                                                                  |
| `API_PORT`                | The port the M3TAL API daemon listens on.                                                                      | `5050`             | `5050`                                         | API daemon, Dashboard container                                                  |
| `DASHBOARD_PORT`          | The port the M3TAL Dashboard container listens on.                                                             | `8082`             | `8082`                                         | Dashboard container, Traefik (if applicable)                                     |
| `DASHBOARD_EXPOSE_MODE`   | Controls how the M3TAL Dashboard is exposed: `local` (direct port) or `traefik` (via Traefik).              | `local`            | `traefik`                                      | CLI, Dashboard container                                                         |
| `HTTP_PORT`               | The port for generic HTTP traffic (used by some internal services).                                            | `8080`             | `8080`                                         | API daemon                                                                       |
| `STATE_DIR`               | The directory where M3TAL stores its state database and configuration files.                                   | `./state`          | `/mnt/config/m3tal/state`                      | API daemon, Dashboard container                                                  |
| `LOG_LEVEL`               | Sets the logging verbosity for M3TAL services.                                                                 | `info`             | `debug`                                        | API daemon                                                                       |
| `DEBUG_MODE`              | Enables or disables debug mode for certain M3TAL components.                                                   | `false`            | `true`                                         | CLI, API daemon                                                                  |
| `METRICS_ENABLED`         | Enables or disables the collection and exposition of metrics.                                                  | `true`             | `false`                                        | API daemon                                                                       |
| **Authentication**        |                                                                                                                |                    |                                                |                                                                                  |
| `DASHBOARD_SECRET`        | Secret key for signing dashboard session cookies. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.** | `change_me_immediately` | `s0me_s3cur3_s3cr3t_k3y`                       | Dashboard container                                                              |
| `API_TOKEN`               | API token for authenticating with the M3TAL API. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.** | `change_me_api_token` | `v3ry_s3cur3_api_t0k3n_h3r3`                   | CLI                                                                              |
| `ADMIN_PASSWORD`          | The password for the default administrative user in the M3TAL Dashboard.                                     | `admin_pass`       | `my_strong_admin_password`                     | Dashboard container                                                              |
| **Network**               |                                                                                                                |                    |                                                |                                                                                  |
| `NETWORK_NAME`            | The name of the Docker network used by M3TAL services.                                                         | `m3tal`            | `m3tal_net`                                    | Docker Compose files                                                             |
| `LOCAL_IP`                | The local IP address to bind services to.                                                                      | `127.0.0.1`        | `192.168.1.100`                                | API daemon (for internal communication)                                          |
| `DOMAIN`                  | The primary domain for M3TAL services. Setting this enables `dash.DOMAIN` and `api.DOMAIN` Traefik routing.    | `localhost`        | `my.m3tal.domain`                              | Traefik, Dashboard container (via Traefik labels)                                |
| **Storage**               |                                                                                                                |                    |                                                |                                                                                  |
| `BASE_STORAGE_PATH`       | The base directory for M3TAL data storage. **Defaults to `/mnt` in production, not `./data` as in templates.** | `./data`           | `/mnt/m3tal_storage`                           | Dashboard container, CLI (for volume mounts)                                     |
| `MEDIA_PATH`              | The subdirectory within `BASE_STORAGE_PATH` for media files.                                                   | `./data/media`     | `/mnt/m3tal_storage/media`                     | Dashboard container                                                              |
| `CONFIG_PATH`             | The subdirectory within `BASE_STORAGE_PATH` for configuration files.                                           | `./data/config`    | `/mnt/m3tal_storage/config`                    | Dashboard container, CLI (for volume mounts)                                     |
| `DOWNLOADS_PATH`          | The subdirectory within `BASE_STORAGE_PATH` for downloaded files.                                              | `./data/downloads` | `/mnt/m3tal_storage/downloads`                 | Dashboard container                                                              |
| **User/Group IDs**        |                                                                                                                |                    |                                                |                                                                                  |
| `PUID`                    | The user ID to run Docker containers as.                                                                       | `1000`             | `1001`                                         | Docker Compose files (for container user mapping)                                |
| `PGID`                    | The group ID to run Docker containers as.                                                                      | `1000`             | `1001`                                         | Docker Compose files (for container user mapping)                                |
| **System**                |                                                                                                                |                    |                                                |                                                                                  |
| `TZ`                      | The timezone to use for M3TAL services.                                                                        | `America/Denver`   | `Europe/London`                                | API daemon, Dashboard container                                                  |
| **Traefik**               |                                                                                                                |                    |                                                |                                                                                  |
| `TRAEFIK_WEB_PORT`        | The host port Traefik listens on for HTTP traffic.                                                             | `80`               | `80`                                           | Traefik container                                                                |
| `TRAEFIK_WEBHTTPS_PORT`   | The host port Traefik listens on for HTTPS traffic.                                                            | `443`              | `443`                                          | Traefik container                                                                |
| `TRAEFIK_DASHBOARD_PORT`  | The port Traefik uses internally to access its own dashboard.                                                  | `8080`             | `8080`                                         | Traefik container                                                                |
| **VPN**                   |                                                                                                                |                    |                                                |                                                                                  |
| `VPN_USER`                | The username for VPN authentication.                                                                           | `user`             | `vpnuser123`                                   | M3TAL CLI (if VPN features are implemented)                                      |
| `VPN_PASSWORD`            | The password for VPN authentication.                                                                           | `password`         | `vpnsecretpassword`                            | M3TAL CLI (if VPN features are implemented)                                      |

---

## Detailed Variable Descriptions

### Core

*   **`API_PORT`**
    *   **Description:** The network port on which the M3TAL API daemon (Go application) will listen for incoming requests.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard container (to connect to the API).

*   **`DASHBOARD_PORT`**
    *   **Description:** The network port on which the M3TAL Dashboard (Python/Flask application) will listen internally.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** M3TAL Dashboard container, Traefik (if `DASHBOARD_EXPOSE_MODE` is set to `traefik`).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible.
        *   `local`: The dashboard is exposed directly via its `DASHBOARD_PORT`, accessible via `http://HOST_IP:DASHBOARD_PORT`. This mode does not require Traefik and is suitable for LAN-only access or initial setup.
        *   `traefik`: The dashboard is routed through Traefik. Access is typically via `http://dash.DOMAIN`. This mode requires Traefik to be running and configured.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** M3TAL CLI (to determine which compose override to use), M3TAL Dashboard container (via Traefik labels when in `traefik` mode).

*   **`HTTP_PORT`**
    *   **Description:** A general-purpose HTTP port, often used for internal communication or by specific services.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** M3TAL API daemon.

*   **`STATE_DIR`**
    *   **Description:** The directory where M3TAL stores its persistent state, including the SQLite database (`state.db`) and other configuration files.
    *   **Default Value:** `./state`
    *   **Example Value:** `/mnt/config/m3tal/state`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard container (as a volume mount).

*   **`LOG_LEVEL`**
    *   **Description:** Sets the verbosity of logging for M3TAL services. Common values include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** M3TAL API daemon.

*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug mode for certain M3TAL components. This can provide more verbose output or enable additional debugging features.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** M3TAL CLI, M3TAL API daemon.

*   **`METRICS_ENABLED`**
    *   **Description:** Toggles the collection and exposition of operational metrics for M3TAL services.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** M3TAL API daemon.

### Authentication

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used to sign session cookies for the M3TAL Dashboard. This is a critical security parameter. **This variable is auto-generated on the first run of `m3tal init` and should NOT be set manually unless you are intentionally rotating the secret.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `s0me_s3cur3_s3cr3t_k3y_g3n3rat3d_by_m3tal`
    *   **Used By:** M3TAL Dashboard container.

*   **`API_TOKEN`**
    *   **Description:** An authentication token used by the M3TAL CLI to interact with the M3TAL API. **This variable is auto-generated on the first run of `m3tal init` and should NOT be set manually unless you are intentionally rotating the token.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `v3ry_s3cur3_api_t0k3n_h3r3_g3n3rat3d_by_m3tal`
    *   **Used By:** M3TAL CLI.

*   **`ADMIN_PASSWORD`**
    *   **Description:** The password for the default administrative user account within the M3TAL Dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `my_strong_and_unique_admin_password`
    *   **Used By:** M3TAL Dashboard container.

### Network

*   **`NETWORK_NAME`**
    *   **Description:** Specifies the name of the Docker network that M3TAL services will be connected to. This ensures proper inter-container communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_network`
    *   **Used By:** Docker Compose files defining M3TAL services.

*   **`LOCAL_IP`**
    *   **Description:** The local IP address that M3TAL services will bind to. This is typically `127.0.0.1` for internal services.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `127.0.0.1`
    *   **Used By:** M3TAL API daemon (for internal listening addresses).

*   **`DOMAIN`**
    *   **Description:** The primary domain name associated with your M3TAL deployment. Setting this variable is crucial for Traefik routing. When set, M3TAL will automatically configure Traefik to route traffic to `api.DOMAIN` (for the API) and `dash.DOMAIN` (for the dashboard, when `DASHBOARD_EXPOSE_MODE=traefik`).
    *   **Default Value:** `localhost`
    *   **Example Value:** `my.m3tal.domain`
    *   **Used By:** Traefik (to define routing rules), M3TAL Dashboard container (via Traefik labels when in `traefik` mode).

### Storage

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory where M3TAL will store all its persistent data, including configuration, media, downloads, and state. **Note: In production deployments, this defaults to `/mnt` to ensure data is stored on a potentially separate, persistent volume. In template or development setups, it might default to `./data`.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal_storage`
    *   **Used By:** M3TAL Dashboard container, M3TAL CLI (for defining Docker volume mounts).

*   **`MEDIA_PATH`**
    *   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` where media files (e.g., uploaded images, videos) will be stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal_storage/media`
    *   **Used By:** M3TAL Dashboard container.

*   **`CONFIG_PATH`**
    *   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` where M3TAL's configuration files (beyond the `.env` file itself) will be stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal_storage/config`
    *   **Used By:** M3TAL Dashboard container, M3TAL CLI (for volume mounts).

*   **`DOWNLOADS_PATH`**
    *   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` where downloaded files from M3TAL services will be saved.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal_storage/downloads`
    *   **Used By:** M3TAL Dashboard container.

### User/Group IDs

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers will run as. This is important for ensuring correct file permissions when containers access host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used By:** Docker Compose files (typically used in the `user` directive for services).

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers will run as. This works in conjunction with `PUID` to manage file permissions.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Used By:** Docker Compose files (typically used in the `user` directive for services).

### System

*   **`TZ`**
    *   **Description:** Specifies the timezone to be used by M3TAL services. This ensures that timestamps in logs and other time-sensitive operations are correct.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard container.

### Traefik

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik will listen on for incoming HTTP traffic. This is the primary entry point for external HTTP requests.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik container.

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik will listen on for incoming HTTPS traffic. This is the primary entry point for external HTTPS requests (requires SSL certificate configuration).
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik container.

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik uses to expose its own administrative dashboard. This is typically accessible on `127.0.0.1:8081`.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik container.

### VPN

*   **`VPN_USER`**
    *   **Description:** The username required for VPN authentication. This variable is relevant if M3TAL integrates with or manages VPN connections.
    *   **Default Value:** `user`
    *   **Example Value:** `vpnuser123`
    *   **Used By:** M3TAL CLI (if VPN-related features are implemented).

*   **`VPN_PASSWORD`**
    *   **Description:** The password required for VPN authentication. This variable is relevant if M3TAL integrates with or manages VPN connections.
    *   **Default Value:** `password`
    *   **Example Value:** `vpnsecretpassword`
    *   **Used By:** M3TAL CLI (if VPN-related features are implemented).