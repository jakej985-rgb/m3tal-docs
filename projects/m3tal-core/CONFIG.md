Greetings, fellow architect of the M3TAL Ecosystem! DocSmith here, ready to lay down the definitive schema for our environment variables. These are the cogs and gears that power our magnificent machine, so understanding them is paramount.

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. This file is diligently consumed by both the `m3tal` CLI binary and all Docker Compose stacks through the `--env-file` option, ensuring a consistent configuration across the entire system. You can easily manage these variables using the `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific adjustments.

---

### Quick Reference

| Name                    | Description                                                                                                                               | Default Value            | Component(s)                                   |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- | :----------------------- | :--------------------------------------------- |
| `API_PORT`              | Internal port for the M3TAL API daemon.                                                                                                   | `5050`                   | API daemon, Dashboard container                |
| `DASHBOARD_PORT`        | Internal port for the M3TAL Dashboard container.                                                                                          | `8082`                   | Dashboard container, Traefik gateway           |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL Dashboard is exposed (`local` or `traefik`).                                                                       | `local`                  | Dashboard container, CLI                       |
| `HTTP_PORT`             | Generic HTTP port, typically for API daemon exposure.                                                                                     | `8080`                   | API daemon, Traefik gateway                    |
| `STATE_DIR`             | Directory for the M3TAL API daemon's state database (`state.db`).                                                                         | `./state`                | API daemon, Dashboard container                |
| `LOG_LEVEL`             | Logging verbosity for the API daemon (`info`, `debug`, `warn`, `error`).                                                                  | `info`                   | API daemon                                     |
| `DASHBOARD_SECRET`      | Secret key for Dashboard session management. **Auto-generated.**                                                                          | `change_me_immediately`  | Dashboard container                            |
| `API_TOKEN`             | Authentication token for CLI/external API access. **Auto-generated.**                                                                     | `change_me_api_token`    | CLI, API daemon                                |
| `ADMIN_PASSWORD`        | Default admin password for the Dashboard.                                                                                                 | `admin_pass`             | Dashboard container                            |
| `NETWORK_NAME`          | Name of the internal Docker network used by M3TAL.                                                                                        | `m3tal`                  | All M3TAL Docker containers                    |
| `LOCAL_IP`              | Local IP address, potentially for internal service discovery.                                                                             | `127.0.0.1`              | All M3TAL Docker containers, API daemon        |
| `DOMAIN`                | Base domain for Traefik routing rules (e.g., `dash.DOMAIN`, `api.DOMAIN`).                                                                | `localhost`              | Traefik gateway, Dashboard container, CLI      |
| `VPN_USER`              | Username for VPN services (if enabled).                                                                                                   | `user`                   | VPN container (Cloudflared)                    |
| `VPN_PASSWORD`          | Password for VPN services (if enabled).                                                                                                   | `password`               | VPN container (Cloudflared)                    |
| `BASE_STORAGE_PATH`     | Root directory for all M3TAL persistent data (media, config, downloads).                                                                  | `./data`                 | All M3TAL Docker containers, CLI               |
| `MEDIA_PATH`            | Subdirectory under `BASE_STORAGE_PATH` for media files.                                                                                   | `./data/media`           | All M3TAL Docker containers                    |
| `CONFIG_PATH`           | Subdirectory under `BASE_STORAGE_PATH` for configuration files and user stacks.                                                           | `./data/config`          | All M3TAL Docker containers                    |
| `DOWNLOADS_PATH`        | Subdirectory under `BASE_STORAGE_PATH` for downloads.                                                                                     | `./data/downloads`       | All M3TAL Docker containers                    |
| `PUID`                  | User ID for container processes.                                                                                                          | `1000`                   | All M3TAL Docker containers                    |
| `PGID`                  | Group ID for container processes.                                                                                                         | `1000`                   | All M3TAL Docker containers                    |
| `TZ`                    | Timezone for containers.                                                                                                                  | `America/Denver`         | All M3TAL Docker containers                    |
| `TRAEFIK_WEB_PORT`      | Port Traefik listens on for HTTP traffic.                                                                                                 | `80`                     | Traefik gateway                                |
| `TRAEFIK_WEBHTTPS_PORT` | Port Traefik listens on for HTTPS traffic.                                                                                                | `443`                    | Traefik gateway                                |
| `TRAEFIK_DASHBOARD_PORT`| Internal port for Traefik's own dashboard.                                                                                                | `8080`                   | Traefik gateway                                |
| `DEBUG_MODE`            | Enables verbose debug logging and features for the API daemon.                                                                            | `false`                  | API daemon                                     |
| `METRICS_ENABLED`       | Enables the Prometheus metrics endpoint for the API daemon.                                                                               | `true`                   | API daemon                                     |

---

### Detailed Environment Variable Reference

---

### Core Variables

These variables define the fundamental operational parameters for the M3TAL system's core components.

#### `API_PORT`

*   **Description**: The internal port on which the M3TAL API daemon listens for requests. While the API daemon typically exposes itself on port `8080` to other services (like Traefik) and internally to containers (like the dashboard via `GO_API_URL`), this variable defines its core listening port.
*   **Default Value**: `5050`
*   **Example Value**: `5050`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`), Dashboard container (for `GO_API_URL`).

#### `DASHBOARD_PORT`

*   **Description**: The internal port on which the `m3tal-dashboard` container listens. This port is either directly exposed to the host (in `local` mode) or routed by Traefik (in `traefik` mode).
*   **Default Value**: `8082`
*   **Example Value**: `8082`
*   **Component(s) Used By**: Dashboard container, Traefik gateway.

#### `DASHBOARD_EXPOSE_MODE`

*   **Description**: Controls how the M3TAL Dashboard is made accessible.
    *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly bound to the host, allowing access via `http://HOST_IP:8082`. This is ideal for LAN-only or local testing setups and does not require Traefik.
    *   `traefik`: The dashboard container is configured with Traefik labels, enabling access via `http://dash.DOMAIN` through the Traefik gateway.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Component(s) Used By**: Dashboard container (`m3tal-compose.yml` overrides), CLI (`m3tal dash up`).

#### `HTTP_PORT`

*   **Description**: A generic HTTP port, often used to refer to the port on which the M3TAL API daemon is publicly exposed to other services on the host machine. This value typically aligns with the port Traefik routes to for the API.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`), Traefik gateway.

#### `STATE_DIR`

*   **Description**: Specifies the directory where the M3TAL API daemon stores its essential runtime data, including the SQLite state database (`state.db`). For production deployments, this path is canonically `/var/lib/m3tal/state`.
*   **Default Value**: `./state`
*   **Example Value**: `/var/lib/m3tal/state`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`), Dashboard container.

#### `LOG_LEVEL`

*   **Description**: Sets the logging verbosity for the M3TAL API daemon. Options include `info`, `debug`, `warn`, and `error`.
*   **Default Value**: `info`
*   **Example Value**: `debug`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`).

#### `DEBUG_MODE`

*   **Description**: When set to `true`, enables verbose debug logging and potentially other debug-specific features within the M3TAL API daemon.
*   **Default Value**: `false`
*   **Example Value**: `true`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`).

#### `METRICS_ENABLED`

*   **Description**: When set to `true`, the M3TAL API daemon exposes a Prometheus-compatible metrics endpoint for monitoring purposes.
*   **Default Value**: `true`
*   **Example Value**: `false`
*   **Component(s) Used By**: API daemon (`m3tal-api.service`).

---

### Authentication Variables

These variables secure access to M3TAL's components and services.

#### `DASHBOARD_SECRET`

*   **Description**: A critical secret key used by the M3TAL Dashboard for securely managing user sessions and protecting sensitive operations. **This variable is automatically generated by `m3tal init` during the first-time setup.** Users should typically *not* set this manually unless performing a secret rotation.
*   **Default Value**: `change_me_immediately`
*   **Example Value**: `a_long_random_string_of_characters_and_symbols_123ABC`
*   **Component(s) Used By**: Dashboard container.

#### `API_TOKEN`

*   **Description**: An authentication token essential for the M3TAL CLI and other external clients to securely interact with the M3TAL API daemon. **This variable is automatically generated by `m3tal init` during the first-time setup.** Users should typically *not* set this manually unless performing a token rotation.
*   **Default Value**: `change_me_api_token`
*   **Example Value**: `another_long_random_alphanumeric_token_XYZ789`
*   **Component(s) Used By**: CLI, API daemon (`m3tal-api.service`).

#### `ADMIN_PASSWORD`

*   **Description**: The default administrator password for accessing the M3TAL Dashboard. It is highly recommended to change this immediately after initial setup using `m3tal dashpass` or via the dashboard interface.
*   **Default Value**: `admin_pass`
*   **Example Value**: `my_secure_admin_password`
*   **Component(s) Used By**: Dashboard container.

---

### Network Variables

These variables define the network configuration for M3TAL services and Docker containers.

#### `NETWORK_NAME`

*   **Description**: Specifies the name of the internal Docker network created and used by all M3TAL-managed Docker containers to communicate with each other.
*   **Default Value**: `m3tal`
*   **Example Value**: `my-m3tal-network`
*   **Component(s) Used By**: All M3TAL Docker containers.

#### `LOCAL_IP`

*   **Description**: Defines the local IP address for the M3TAL system. This can be used for internal service discovery or binding specific services.
*   **Default Value**: `127.0.0.1`
*   **Example Value**: `192.168.1.100`
*   **Component(s) Used By**: All M3TAL Docker containers, API daemon.

---

### Storage Variables

These variables dictate where M3TAL stores its persistent data on the filesystem.

#### `BASE_STORAGE_PATH`

*   **Description**: The root directory for all M3TAL-related persistent data, including media files, configuration, and downloads. **Important:** While the template defaults to `./data` for local testing, in production M3TAL deployments, this variable typically defaults to `/mnt` (for dedicated mounted volumes) or `/var/lib/m3tal/data` to ensure data persistence across reboots and upgrades.
*   **Default Value**: `./data`
*   **Example Value**: `/mnt/m3tal-data`
*   **Component(s) Used By**: All M3TAL Docker containers (via bind mounts), CLI.

#### `MEDIA_PATH`

*   **Description**: The subdirectory under `BASE_STORAGE_PATH` where media files and user-uploaded content are stored.
*   **Default Value**: `./data/media`
*   **Example Value**: `${BASE_STORAGE_PATH}/media` (e.g., `/mnt/media`)
*   **Component(s) Used By**: All M3TAL Docker containers (via bind mounts).

#### `CONFIG_PATH`

*   **Description**: The subdirectory under `BASE_STORAGE_PATH` designated for various M3TAL configuration files and user-managed Docker Compose stacks. This path is crucial as it contains files like `m3tal-compose.yml` and `/docker/users.json`.
*   **Default Value**: `./data/config`
*   **Example Value**: `${BASE_STORAGE_PATH}/config` (e.g., `/mnt/config`)
*   **Component(s) Used By**: All M3TAL Docker containers (via bind mounts), CLI.

#### `DOWNLOADS_PATH`

*   **Description**: The subdirectory under `BASE_STORAGE_PATH` where any M3TAL-managed downloads or temporary files requiring persistence are stored.
*   **Default Value**: `./data/downloads`
*   **Example Value**: `${BASE_STORAGE_PATH}/downloads` (e.g., `/mnt/downloads`)
*   **Component(s) Used By**: All M3TAL Docker containers (via bind mounts).

---

### Traefik Variables

These variables configure the Traefik reverse proxy for external access to M3TAL services.

#### `DOMAIN`

*   **Description**: The base domain name that Traefik uses to construct routing rules. Setting this variable enables domain-based routes such as `dash.DOMAIN` for the M3TAL Dashboard and `api.DOMAIN` for the M3TAL API daemon. If left at its default, Traefik routes will use `dash.localhost` and `api.localhost`.
*   **Default Value**: `localhost`
*   **Example Value**: `m3tal.example.com`
*   **Component(s) Used By**: Traefik gateway, Dashboard container (for Traefik labels), CLI.

#### `TRAEFIK_WEB_PORT`

*   **Description**: The host port on which Traefik listens for incoming HTTP traffic. This is typically port 80.
*   **Default Value**: `80`
*   **Example Value**: `80`
*   **Component(s) Used By**: Traefik gateway.

#### `TRAEFIK_WEBHTTPS_PORT`

*   **Description**: The host port on which Traefik listens for incoming HTTPS traffic. This is typically port 443.
*   **Default Value**: `443`
*   **Example Value**: `443`
*   **Component(s) Used By**: Traefik gateway.

#### `TRAEFIK_DASHBOARD_PORT`

*   **Description**: The internal port on which the Traefik dashboard listens within its container. While the internal container port is `8080`, Traefik's dashboard is typically exposed on the host via `127.0.0.1:8081` for local diagnostic access.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Component(s) Used By**: Traefik gateway.

---

### VPN Variables

These variables are used to configure optional VPN client services, such as Cloudflared tunnels.

#### `VPN_USER`

*   **Description**: The username used for authentication with VPN services, such as Cloudflared tunnels. This is used by the `cloudflared` container if configured.
*   **Default Value**: `user`
*   **Example Value**: `m3taluser`
*   **Component(s) Used By**: VPN container (Cloudflared).

#### `VPN_PASSWORD`

*   **Description**: The password used for authentication with VPN services, such as Cloudflared tunnels. This is used by the `cloudflared` container if configured.
*   **Default Value**: `password`
*   **Example Value**: `m3talpassword`
*   **Component(s) Used By**: VPN container (Cloudflared).

---

### System Variables

These variables affect the runtime environment of Docker containers, particularly relating to user permissions and locale.

#### `PUID`

*   **Description**: The User ID (UID) that container processes will run as. Setting this ensures that files created by containers have the correct ownership on the host filesystem, preventing permission issues.
*   **Default Value**: `1000`
*   **Example Value**: `1000`
*   **Component(s) Used By**: All M3TAL Docker containers.

#### `PGID`

*   **Description**: The Group ID (GID) that container processes will run as. Similar to `PUID`, this ensures correct group ownership for container-generated files on the host.
*   **Default Value**: `1000`
*   **Example Value**: `1000`
*   **Component(s) Used By**: All M3TAL Docker containers.

#### `TZ`

*   **Description**: Sets the timezone for all M3TAL Docker containers, ensuring consistent time reporting and scheduling for services.
*   **Default Value**: `America/Denver`
*   **Example Value**: `Europe/London`
*   **Component(s) Used By**: All M3TAL Docker containers.