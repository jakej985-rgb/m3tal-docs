# Environment Variables

As DocSmith, the M3TAL Ecosystem Documentation Architect, my purpose is to illuminate the intricate workings of your system. All M3TAL environment variables are centrally managed within the `/etc/m3tal/.env` file. This file serves as the definitive source of truth, loaded by both the `m3tal` CLI binary and all Docker Compose stacks via the `--env-file` option, ensuring consistent configuration across your entire deployment.

You can effortlessly interact with and modify these critical variables using the `m3tal config wizard` for a guided setup experience, or through `m3tal config set KEY value` for direct, precise adjustments.

---

## Quick Reference

| Variable Name             | Description (Short)                                                                 | Default Value            | Used By                                            |
| :------------------------ | :---------------------------------------------------------------------------------- | :----------------------- | :------------------------------------------------- |
| `API_PORT`                | Internal port for API daemon (dashboard's connection target).                       | `5050`                   | Dashboard container                                |
| `DASHBOARD_PORT`          | Internal port for Dashboard container; host port in `local` mode.                   | `8082`                   | Dashboard container, CLI                           |
| `DASHBOARD_EXPOSE_MODE`   | How Dashboard is exposed: `local` (direct) or `traefik` (via reverse proxy).        | `local`                  | CLI, Dashboard container                           |
| `HTTP_PORT`               | General purpose HTTP port.                                                          | `8080`                   | (User stacks)                                      |
| `STATE_DIR`               | Directory for API daemon's state DB and Dashboard's config.                         | `./state`                | API daemon, Dashboard container                    |
| `LOG_LEVEL`               | Verbosity of API daemon logs (`debug`, `info`, `warn`, `error`).                    | `info`                   | API daemon                                         |
| `DASHBOARD_SECRET`        | Cryptographic key for Dashboard sessions. **Auto-generated.**                       | `change_me_immediately`  | Dashboard container                                |
| `API_TOKEN`               | Token for API daemon authentication. **Auto-generated.**                            | `change_me_api_token`    | CLI, Dashboard container, API daemon               |
| `ADMIN_PASSWORD`          | Default password for initial Dashboard admin user.                                  | `admin_pass`             | CLI (for `dashpass init`)                          |
| `NETWORK_NAME`            | Name of the Docker network for M3TAL containers.                                    | `m3tal`                  | All M3TAL containers, CLI                          |
| `LOCAL_IP`                | Host machine's local IP address.                                                    | `127.0.0.1`              | API daemon, Traefik gateway                        |
| `DOMAIN`                  | Base domain for Traefik routing (`dash.DOMAIN`, `api.DOMAIN`).                      | `localhost`              | Traefik gateway, CLI, Dashboard container          |
| `VPN_USER`                | Username for VPN client configurations.                                             | `user`                   | Cloudflared, VPN client containers                 |
| `VPN_PASSWORD`            | Password for VPN client configurations.                                             | `password`               | Cloudflared, VPN client containers                 |
| `BASE_STORAGE_PATH`       | Root directory for all application data (media, config, downloads).                 | `./data`                 | Dashboard container, user stacks                   |
| `MEDIA_PATH`              | Subdirectory for media files.                                                       | `./data/media`           | User stacks                                        |
| `CONFIG_PATH`             | Subdirectory for configuration files.                                               | `./data/config`          | Dashboard container, user stacks                   |
| `DOWNLOADS_PATH`          | Subdirectory for downloaded files.                                                  | `./data/downloads`       | User stacks                                        |
| `PUID`                    | User ID (UID) for container processes.                                              | `1000`                   | Dashboard container, user stacks                   |
| `PGID`                    | Group ID (GID) for container processes.                                             | `1000`                   | Dashboard container, user stacks                   |
| `TZ`                      | Timezone for containers.                                                            | `America/Denver`         | Dashboard container, user stacks                   |
| `TRAEFIK_WEB_PORT`        | Host port for Traefik's HTTP entry point.                                           | `80`                     | Traefik gateway                                    |
| `TRAEFIK_WEBHTTPS_PORT`   | Host port for Traefik's HTTPS entry point.                                          | `443`                    | Traefik gateway                                    |
| `TRAEFIK_DASHBOARD_PORT`  | Internal port Traefik uses for its own admin dashboard.                             | `8080`                   | Traefik gateway                                    |
| `DEBUG_MODE`              | Enables debug logging and features.                                                 | `false`                  | API daemon, Dashboard container, CLI               |
| `METRICS_ENABLED`         | Controls if M3TAL components publish metrics.                                       | `true`                   | API daemon, Dashboard container                    |

---

## Detailed Reference

All variables are read from `/etc/m3tal/.env` by both the CLI (`/usr/bin/m3tal`) and all Docker Compose stacks found in `/docker/` via the `--env-file` option. This ensures a unified configuration across the entire M3TAL system.

### Core Variables

These variables define fundamental aspects of the M3TAL system's operation, including logging verbosity and user/group IDs for container processes to ensure proper file permissions.

*   **`LOG_LEVEL`**
    *   **Description:** Controls the verbosity of logging output for the M3TAL API daemon. Set to `debug` for detailed diagnostic information, `info` for general operational messages, `warn` for non-critical issues, or `error` for critical failures.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** `API daemon`

*   **`STATE_DIR`**
    *   **Description:** Defines the directory where the M3TAL API daemon stores its SQLite state database (`state.db`) and other runtime files. For the Dashboard container, this variable specifies the base path where `users.json` is expected.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal` (common for production deployments, ensuring persistent state)
    *   **Used By:** `API daemon`, `Dashboard container`

*   **`PUID`**
    *   **Description:** The User ID (UID) that containers will use to execute processes. Setting this ensures proper file permissions when containers interact with mounted volumes on the host system, preventing "permission denied" errors. It should typically match the UID of the user account owning the M3TAL data directories.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000` (often the UID for the first non-root user on Linux)
    *   **Used By:** `Dashboard container`, `Other user-defined containers`

*   **`PGID`**
    *   **Description:** The Group ID (GID) that containers will use to execute processes. Similar to `PUID`, this ensures proper group-level file permissions for mounted volumes, allowing containers to read and write to shared storage.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** `Dashboard container`, `Other user-defined containers`

*   **`TZ`**
    *   **Description:** Sets the timezone for containers. This affects logging timestamps, scheduled tasks, and any other time-sensitive operations within the containers. Use standard TZ database names (e.g., `America/New_York`, `Europe/London`).
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `America/Los_Angeles`
    *   **Used By:** `Dashboard container`, `Other user-defined containers`

### Authentication Variables

These variables control security aspects for the M3TAL API and Dashboard, including tokens and initial administrator credentials.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A strong, randomly generated cryptographic key used by the M3TAL Dashboard for secure session management and cookie signing. This secret protects user sessions from tampering.
    *   **IMPORTANT:** This variable is **auto-generated** on the first `m3tal init`. Users should **NOT** set this manually unless performing a secret rotation for enhanced security.
    *   **Default Value:** `change_me_immediately` (This value is a placeholder; a strong random string will be generated.)
    *   **Example Value:** `8h2FjP0sY4dLbG7qR1wE5vC9uX3iA6tM_a_long_random_string`
    *   **Used By:** `Dashboard container`

*   **`API_TOKEN`**
    *   **Description:** A unique token used to authenticate requests to the M3TAL API daemon. This token is used by the `m3tal` CLI binary and the Dashboard container to interact with the API securely.
    *   **IMPORTANT:** This variable is **auto-generated** on the first `m3tal init`. Users should **NOT** set this manually unless performing a token rotation for security purposes.
    *   **Default Value:** `change_me_api_token` (This value is a placeholder; a strong random string will be generated.)
    *   **Example Value:** `zK9yX3cLbQ7aV1rJ5oW2eF6dI4hN0pG_another_long_random_string`
    *   **Used By:** `CLI binary`, `Dashboard container` (for API communication), `API daemon` (for authenticating incoming requests)

*   **`ADMIN_PASSWORD`**
    *   **Description:** The default password for the initial `admin` user account for the M3TAL Dashboard. This value is used by `m3tal dashpass init` if no password is provided interactively during the setup of the dashboard's user store (`/docker/users.json`). It is strongly recommended to change this immediately after setup using `m3tal dashpass set admin`.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MyStrongAndComplexAdminPassword123!`
    *   **Used By:** `CLI binary` (for `m3tal dashpass init` command), `Dashboard container` (initial setup when creating `users.json`)

### Network Variables

These variables configure network settings for M3TAL components, including internal communication ports, exposure modes for the dashboard, and the base domain for external routing.

*   **`API_PORT`**
    *   **Description:** The internal port the M3TAL API daemon is expected to be accessible on by other containers, specifically the Dashboard. While the `m3tal-api.service` itself typically runs on port `8080` on the host, the Dashboard container is configured to reach it via `http://host.docker.internal:${API_PORT}` within its `GO_API_URL` environment variable.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** `Dashboard container` (as part of its `GO_API_URL` environment variable)

*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port the M3TAL Dashboard container listens on. If `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host machine, making the dashboard accessible at `http://HOST_IP:${DASHBOARD_PORT}`.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** `Dashboard container`, `CLI binary` (for determining `local` mode port mapping)

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible to users.
        *   `local`: The dashboard's port is directly published on the host, allowing access via `http://HOST_IP:8082` or `http://localhost:8082`. This mode requires no Traefik configuration and is ideal for LAN-only setups or initial testing.
        *   `traefik`: The dashboard is routed through the Traefik gateway, accessible via `http://dash.${DOMAIN}`. This mode requires Traefik to be running via `m3tal up` and is best for domain-based setups.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** `CLI binary` (for selecting the appropriate Docker Compose override file: `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`)

*   **`HTTP_PORT`**
    *   **Description:** A general-purpose HTTP port variable. This is not explicitly used by core M3TAL components in their default configurations but is often included for user-defined Docker Compose stacks that might require a common HTTP entry point.
    *   **Default Value:** `8080`
    *   **Example Value:** `8000`
    *   **Used By:** `None explicitly in core M3TAL components; useful for user stacks.`

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network used by M3TAL containers for inter-service communication. All M3TAL-managed containers and user stacks are typically attached to this network (often named `proxy` in default core compose files) to allow them to communicate securely and efficiently.
    *   **Default Value:** `m3tal` (Note: The core `routing-compose.yml` and `m3tal-compose.yml` explicitly define an external network named `proxy`. This `NETWORK_NAME` variable could be used to override that convention if desired, for example, when setting up custom networks.)
    *   **Example Value:** `m3tal_backend`
    *   **Used By:** `Dashboard container`, `Traefik gateway`, `Cloudflared`, `Other user-defined containers`, `CLI binary` (for network management)

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the host machine. This can be useful for services that need to bind to a specific local interface or for explicit routing configurations, especially when interacting with `host.docker.internal` from containers.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** `API daemon` (potentially for binding), `Traefik gateway` (for `host.docker.internal` references in dynamic configuration)

*   **`DOMAIN`**
    *   **Description:** The base domain name for your M3TAL deployment. This variable is **critical for Traefik**, as it enables dynamic routing rules for core services like the Dashboard (`dash.DOMAIN`) and the API (`api.DOMAIN`). Setting this variable enables domain-based access to your M3TAL services.
    *   **Default Value:** `localhost`
    *   **Example Value:** `yourhomelab.com`
    *   **Used By:** `Traefik gateway`, `CLI binary` (for generating Traefik dynamic configuration files), `Dashboard container` (via Traefik labels when `DASHBOARD_EXPOSE_MODE=traefik`)

### Storage Variables

These variables define the filesystem paths where M3TAL and user applications store their persistent data, ensuring proper volume mapping within containers.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The absolute root directory on the host filesystem for all M3TAL application data, including media, configuration files, and downloads. Containers then mount subdirectories relative to this path.
    *   **IMPORTANT:** While the template value defaults to `./data` for local development, in production M3TAL deployments, this typically defaults to `/mnt` or a dedicated data volume like `/mnt/m3tal` to ensure robust, persistent storage.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal-data`
    *   **Used By:** `Dashboard container`, `Other user-defined containers` (as the base for their mounted volumes)

*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated specifically for media files (e.g., video, audio, images). Applications requiring media storage should mount this path as a volume.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `${BASE_STORAGE_PATH}/media` (e.g., `/mnt/m3tal-data/media`)
    *   **Used By:** `Other user-defined containers` (for media storage)

*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated for configuration files and application-specific persistent data that isn't media or downloads.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `${BASE_STORAGE_PATH}/config` (e.g., `/mnt/m3tal-data/config`)
    *   **Used By:** `Dashboard container` (for its state/config mount), `Other user-defined containers`

*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated for downloaded files. Applications that manage downloads should store them in this location.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `${BASE_STORAGE_PATH}/downloads` (e.g., `/mnt/m3tal-data/downloads`)
    *   **Used By:** `Other user-defined containers` (for downloads)

### Traefik Variables

These variables configure the Traefik reverse proxy gateway, which handles incoming HTTP/HTTPS traffic and routes it to the correct M3TAL services.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port Traefik uses for its standard HTTP entry point. This is typically port `80`, serving unencrypted web traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** `Traefik gateway`

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port Traefik uses for its standard HTTPS entry point. This is typically port `443` and requires proper TLS configuration (e.g., via Let's Encrypt) within Traefik to encrypt web traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** `Traefik gateway`

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik uses for its own administration dashboard. This dashboard is usually exposed on the host via `127.0.0.1:8081` for local access only, providing a view of Traefik's status and configurations. This variable defines Traefik's internal container port for this service.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** `Traefik gateway` (internal configuration for its dashboard)

### VPN Variables

These variables are used for VPN client configurations, often with services like Cloudflared to establish secure tunnels.

*   **`VPN_USER`**
    *   **Description:** A generic username for VPN client configurations. This might be used by services like Cloudflared if they require authentication for establishing tunnels, or for other VPN client containers within your M3TAL ecosystem.
    *   **Default Value:** `user`
    *   **Example Value:** `cloudflared_user`
    *   **Used By:** `Cloudflared` (if used for authentication), `Other VPN client containers`

*   **`VPN_PASSWORD`**
    *   **Description:** A generic password for VPN client configurations. Use strong, unique passwords generated by `m3tal config wizard` or similar secure methods for any production deployments.
    *   **Default Value:** `password`
    *   **Example Value:** `MySecureVpnPass123!`
    *   **Used By:** `Cloudflared` (if used for authentication), `Other VPN client containers`

### System Variables

These variables control system-wide behaviors such as enabling debug modes or metrics collection across M3TAL components.

*   **`DEBUG_MODE`**
    *   **Description:** When set to `true`, this enables verbose debug logging and potentially other debug-specific features across various M3TAL components (CLI, API daemon, Dashboard). It is highly useful for troubleshooting but should typically be `false` in production to minimize log volume and potential performance impact.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** `API daemon`, `Dashboard container`, `CLI binary`

*   **`METRICS_ENABLED`**
    *   **Description:** When set to `true`, M3TAL components will publish internal operational metrics, which can be scraped by monitoring systems (e.g., Prometheus) for performance analysis, health checks, and capacity planning.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** `API daemon`, `Dashboard container`