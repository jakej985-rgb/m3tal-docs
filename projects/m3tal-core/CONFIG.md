Hello, I'm DocSmith, your M3TAL Ecosystem Documentation Architect. This document provides a comprehensive reference for all environment variables used within the M3TAL ecosystem.

---

### Environment Variable Configuration

All M3TAL environment variables are centrally managed in the `/etc/m3tal/.env` file. This file is the primary configuration source, read by:
*   The `m3tal` CLI binary for commands and setup operations.
*   All Docker Compose stacks launched by `m3tal up` via the `--env-file` option.
*   The `m3tal-api.service` systemd daemon.

You can manage these variables using the `m3tal config wizard` for guided setup, or `m3tal config set <KEY> <value>` for direct modification.

---

## Quick Reference Table

| Name                   | Description                                                                                                                                                                          | Default                     | Example                          | Components                                           |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------- | :------------------------------- | :--------------------------------------------------- |
| **Core**               |                                                                                                                                                                                      |                             |                                  |                                                      |
| `API_PORT`             | Internal port of the M3TAL API daemon for container communication.                                                                                                                   | `5050`                      | `5050`                           | `m3tal-api`, `m3tal-dashboard`                       |
| `DASHBOARD_PORT`       | Internal port of the M3TAL Dashboard container, and optionally host-exposed port.                                                                                                    | `8082`                      | `8082`                           | `m3tal-dashboard`, `CLI binary`, `Traefik gateway`   |
| `DASHBOARD_EXPOSE_MODE`| Controls dashboard exposure: `local` (direct port) or `traefik` (via `dash.DOMAIN`).                                                                                               | `local`                     | `traefik`                        | `CLI binary`, `m3tal-dashboard`                      |
| **Auth**               |                                                                                                                                                                                      |                             |                                  |                                                      |
| `DASHBOARD_SECRET`     | Secret key for dashboard session management. **Auto-generated on `m3tal init`**.                                                                                                     | `change_me_immediately`     | `a_long_random_string`           | `m3tal-dashboard`                                    |
| `API_TOKEN`            | Authentication token for the M3TAL API. **Auto-generated on `m3tal init`**.                                                                                                        | `change_me_api_token`       | `another_long_random_string`     | `m3tal-api`, `m3tal-dashboard`, `CLI binary`         |
| `ADMIN_PASSWORD`       | Initial password for the `admin` user.                                                                                                                                               | `admin_pass`                | `MySecurePassword123!`           | `m3tal-dashboard`                                    |
| **Network**            |                                                                                                                                                                                      |                             |                                  |                                                      |
| `HTTP_PORT`            | Host port for direct access to the `m3tal-api` daemon.                                                                                                                               | `8080`                      | `8080`                           | `m3tal-api`, `Traefik gateway`                       |
| `NETWORK_NAME`         | Name of the Docker network for inter-container communication.                                                                                                                        | `m3tal`                     | `m3tal-prod`                     | All Docker containers                                |
| `LOCAL_IP`             | Local IP address of the host machine for internal routing.                                                                                                                           | `127.0.0.1`                 | `192.168.1.100`                  | `m3tal-api`, `Traefik gateway`                       |
| **Storage**            |                                                                                                                                                                                      |                             |                                  |                                                      |
| `BASE_STORAGE_PATH`    | Host base directory for all M3TAL data. **Defaults to `/mnt` in production deployments.**                                                                                            | `./data`                    | `/mnt/m3tal`                     | `m3tal-dashboard`, `CLI binary`, User containers     |
| `MEDIA_PATH`           | Subdirectory for media files.                                                                                                                                                        | `./data/media`              | `${BASE_STORAGE_PATH}/media`     | `m3tal-dashboard`, User media containers             |
| `CONFIG_PATH`          | Subdirectory for configuration files (e.g., `users.json`).                                                                                                                           | `./data/config`             | `${BASE_STORAGE_PATH}/config`    | `m3tal-dashboard`, `CLI binary`                      |
| `DOWNLOADS_PATH`       | Subdirectory for downloaded content.                                                                                                                                                 | `./data/downloads`          | `${BASE_STORAGE_PATH}/downloads` | User download containers                             |
| `STATE_DIR`            | Internal container path for state data. (Host path derived from `CONFIG_PATH`).                                                                                                      | `./state`                   | `/docker/state`                  | `m3tal-dashboard` (internal container path)          |
| **Traefik**            |                                                                                                                                                                                      |                             |                                  |                                                      |
| `DOMAIN`               | Primary domain for Traefik routing rules, enabling `dash.DOMAIN` and `api.DOMAIN`.                                                                                                   | `localhost`                 | `m3tal.example.com`              | `Traefik gateway`, `CLI binary`, `m3tal-dashboard`   |
| `TRAEFIK_WEB_PORT`     | Host port Traefik binds for HTTP traffic.                                                                                                                                            | `80`                        | `80`                             | `Traefik gateway`                                    |
| `TRAEFIK_WEBHTTPS_PORT`| Host port Traefik binds for HTTPS traffic.                                                                                                                                           | `443`                       | `443`                            | `Traefik gateway`                                    |
| `TRAEFIK_DASHBOARD_PORT`| Internal port of the Traefik dashboard (mapped to `127.0.0.1:8081` on host).                                                                                                         | `8080`                      | `8080`                           | `Traefik gateway`                                    |
| **VPN**                |                                                                                                                                                                                      |                             |                                  |                                                      |
| `VPN_USER`             | Username for VPN services, if integrated.                                                                                                                                            | `user`                      | `myvpnuser`                      | `VPN containers`                                     |
| `VPN_PASSWORD`         | Password for VPN services.                                                                                                                                                           | `password`                  | `MyStrongVPNPass`                | `VPN containers`                                     |
| **System**             |                                                                                                                                                                                      |                             |                                  |                                                      |
| `PUID`                 | User ID (UID) for containers to match host file permissions.                                                                                                                         | `1000`                      | `1000`                           | `m3tal-dashboard`, `Ollama`, User containers         |
| `PGID`                 | Group ID (GID) for containers to match host file permissions.                                                                                                                        | `1000`                      | `1000`                           | `m3tal-dashboard`, `Ollama`, User containers         |
| `TZ`                   | Timezone for all M3TAL components and containers.                                                                                                                                    | `America/Denver`            | `Europe/London`                  | `m3tal-dashboard`, `Ollama`, `Traefik`, `Cloudflared`|
| `DEBUG_MODE`           | Enables detailed debugging output and developer features.                                                                                                                            | `false`                     | `true`                           | All M3TAL components, User containers                |
| `METRICS_ENABLED`      | Controls whether performance metrics and monitoring endpoints are enabled.                                                                                                             | `true`                      | `false`                          | `m3tal-api`, `m3tal-dashboard`, `Traefik gateway`    |

---

## Detailed Environment Variable Reference

### Core Variables

These variables are fundamental to the operation and accessibility of the M3TAL control plane.

*   **`API_PORT`**
    *   **Description**: Specifies the internal port on which the `m3tal-api` daemon listens for HTTP requests from other containers (e.g., the M3TAL Dashboard) via `host.docker.internal`.
    *   **Default**: `5050`
    *   **Example**: `5050`
    *   **Used By**: `m3tal-api` (daemon), `m3tal-dashboard` (communicates with `http://host.docker.internal:${API_PORT}`).

*   **`DASHBOARD_PORT`**
    *   **Description**: Defines the internal port on which the `m3tal-dashboard` container listens. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host machine.
    *   **Default**: `8082`
    *   **Example**: `8082`
    *   **Used By**: `m3tal-dashboard` (container), `CLI binary` (for `m3tal dash up` when using local mode), `Traefik gateway` (when `DASHBOARD_EXPOSE_MODE=traefik`).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description**: Controls how the M3TAL Dashboard is made accessible.
        *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly published to the host. Access via `http://HOST_IP:DASHBOARD_PORT`. This uses the `m3tal-compose.local.yml` override.
        *   `traefik`: The dashboard is exposed via Traefik at `http://dash.DOMAIN`. This requires Traefik to be running and uses the `m3tal-compose.traefik.yml` override.
    *   **Default**: `local`
    *   **Example**: `traefik`
    *   **Used By**: `CLI binary` (specifically `m3tal dash up` to select the correct compose override file).

### Authentication Variables

These variables secure access to the M3TAL Dashboard and API.

*   **`DASHBOARD_SECRET`**
    *   **Description**: A secret key used by the M3TAL Dashboard for secure session management (e.g., signing cookies). **This value is automatically generated during the initial `m3tal init` setup.** Users should only manually set this if rotating the secret.
    *   **Default**: `change_me_immediately`
    *   **Example**: `my_very_long_and_random_dashboard_secret_key_0123456789`
    *   **Used By**: `m3tal-dashboard` (container).

*   **`API_TOKEN`**
    *   **Description**: An authentication token required to access the M3TAL API daemon. **This value is automatically generated during the initial `m3tal init` setup.** Users should only manually set this if rotating the token.
    *   **Default**: `change_me_api_token`
    *   **Example**: `api_token_generated_by_m3tal_init_randomstringabc123`
    *   **Used By**: `m3tal-api` (daemon), `m3tal-dashboard` (to authenticate with the API), `CLI binary` (to authenticate with the API).

*   **`ADMIN_PASSWORD`**
    *   **Description**: The default password for the initial `admin` user account within the M3TAL Dashboard. It is highly recommended to change this immediately after the first login using the `m3tal dashpass` command.
    *   **Default**: `admin_pass`
    *   **Example**: `MyStrongAdminPassword123!`
    *   **Used By**: `m3tal-dashboard` (container, for initial user creation if `users.json` is empty).

### Network Variables

These variables configure network settings for M3TAL components and container communication.

*   **`HTTP_PORT`**
    *   **Description**: The host port on which the `m3tal-api` daemon is directly accessible, typically for `localhost` connections or specific Traefik routing rules that target the host directly (e.g., `http://host.docker.internal:8080`).
    *   **Default**: `8080`
    *   **Example**: `8080`
    *   **Used By**: `m3tal-api` (daemon), `Traefik gateway` (for routing to the `m3tal-api` via `host.docker.internal`).

*   **`NETWORK_NAME`**
    *   **Description**: The name of the Docker bridge network used by all M3TAL-managed containers for internal communication. All services within this network can communicate by their container names.
    *   **Default**: `m3tal`
    *   **Example**: `m3tal-prod`
    *   **Used By**: `m3tal-dashboard`, `Traefik gateway`, `Cloudflared`, `Ollama`, and any other user-defined Docker Compose stacks.

*   **`LOCAL_IP`**
    *   **Description**: Specifies the local IP address of the host machine. This can be used for explicit network bindings or routing rules.
    *   **Default**: `127.0.0.1`
    *   **Example**: `192.168.1.100`
    *   **Used By**: `m3tal-api` (daemon, for potential binding), `Traefik gateway` (e.g., for exposing its own dashboard on `127.0.0.1:8081`).

### Storage Variables

These variables define the filesystem paths for M3TAL's data and configurations.

*   **`BASE_STORAGE_PATH`**
    *   **Description**: The top-level directory on the host machine where all M3TAL-related persistent data (media, configurations, downloads, etc.) will be stored. **In production deployments, this variable defaults to `/mnt` to ensure data persists on mounted storage.** The template default of `./data` is primarily for local development.
    *   **Default**: `./data`
    *   **Example**: `/mnt/m3tal`
    *   **Used By**: `m3tal-dashboard` (for volume mounts), `CLI binary` (during initial setup), other user-defined containers requiring persistent storage.

*   **`MEDIA_PATH`**
    *   **Description**: A subdirectory within `BASE_STORAGE_PATH` specifically designated for media files (e.g., movies, music, photos).
    *   **Default**: `./data/media`
    *   **Example**: `${BASE_STORAGE_PATH}/media` (resolves to `/mnt/m3tal/media` if `BASE_STORAGE_PATH=/mnt/m3tal`).
    *   **Used By**: `m3tal-dashboard` (for volume mounts), user-defined media server containers.

*   **`CONFIG_PATH`**
    *   **Description**: A subdirectory within `BASE_STORAGE_PATH` used for storing configuration files and state unique to M3TAL's control plane, such as the dashboard's `users.json` and internal API state.
    *   **Default**: `./data/config`
    *   **Example**: `${BASE_STORAGE_PATH}/config` (resolves to `/mnt/m3tal/config`).
    *   **Used By**: `m3tal-dashboard` (for volume mounts), `CLI binary` (for config management).

*   **`DOWNLOADS_PATH`**
    *   **Description**: A subdirectory within `BASE_STORAGE_PATH` intended for downloaded content.
    *   **Default**: `./data/downloads`
    *   **Example**: `${BASE_STORAGE_PATH}/downloads` (resolves to `/mnt/m3tal/downloads`).
    *   **Used By**: User-defined download client containers.

*   **`STATE_DIR`**
    *   **Description**: Specifies an *internal* path within certain containers (e.g., `m3tal-dashboard`) where they expect to find their runtime state. The actual host path mounted to this internal path is typically managed via `CONFIG_PATH` and the Docker Compose volumes.
    *   **Default**: `./state` (This is typically an internal container path; the host path is determined by volume mounts, e.g., `${CONFIG_PATH}/m3tal/state` mounted to `/docker/state`).
    *   **Example**: `/docker/state` (internal to dashboard container).
    *   **Used By**: `m3tal-dashboard` (as an internal path for its state files).

### Traefik Variables

These variables configure the Traefik reverse proxy for routing services by domain.

*   **`DOMAIN`**
    *   **Description**: The primary domain name used for dynamic routing rules by Traefik. Setting this enables accessible routes like `dash.DOMAIN` for the dashboard and `api.DOMAIN` for the API daemon.
    *   **Default**: `localhost`
    *   **Example**: `m3tal.example.com`
    *   **Used By**: `Traefik gateway` (for creating host rules), `CLI binary` (for configuration generation), `m3tal-dashboard` (via Traefik labels when `DASHBOARD_EXPOSE_MODE=traefik`).

*   **`TRAEFIK_WEB_PORT`**
    *   **Description**: The host port that Traefik binds for HTTP traffic. This is the entrypoint named `web` in Traefik's configuration.
    *   **Default**: `80`
    *   **Example**: `80`
    *   **Used By**: `Traefik gateway`.

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description**: The host port that Traefik binds for HTTPS traffic. This typically corresponds to an entrypoint named `websecure` in Traefik's configuration, though not explicitly shown in the provided context for routing.
    *   **Default**: `443`
    *   **Example**: `443`
    *   **Used By**: `Traefik gateway`.

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description**: Defines the *internal* port of the Traefik dashboard within its container. This internal port is then typically mapped to `127.0.0.1:8081` on the host, making the Traefik dashboard accessible locally.
    *   **Default**: `8080`
    *   **Example**: `8080`
    *   **Used By**: `Traefik gateway`.

### VPN Variables

These variables are for configuring VPN services if they are integrated into your M3TAL setup.

*   **`VPN_USER`**
    *   **Description**: The username for authenticating with VPN services.
    *   **Default**: `user`
    *   **Example**: `myvpnuser`
    *   **Used By**: `VPN containers` (if deployed as part of your stack).

*   **`VPN_PASSWORD`**
    *   **Description**: The password for authenticating with VPN services.
    *   **Default**: `password`
    *   **Example**: `MyStrongVPNPass123`
    *   **Used By**: `VPN containers` (if deployed as part of your stack).

### System Variables

These variables control general system-level behavior, permissions, and logging.

*   **`PUID`**
    *   **Description**: The User ID (UID) that containers will run as, crucial for ensuring proper file ownership and permissions on host-mounted volumes. It should typically match the UID of the user running Docker.
    *   **Default**: `1000` (Commonly the UID of the first non-root user).
    *   **Example**: `1000`
    *   **Used By**: `m3tal-dashboard`, `Ollama`, and most other user-defined containers.

*   **`PGID`**
    *   **Description**: The Group ID (GID) that containers will run as, ensuring proper file group ownership and permissions on host-mounted volumes. It should typically match the GID of the user running Docker.
    *   **Default**: `1000` (Commonly the GID of the first non-root user).
    *   **Example**: `1000`
    *   **Used By**: `m3tal-dashboard`, `Ollama`, and most other user-defined containers.

*   **`TZ`**
    *   **Description**: Sets the timezone for all M3TAL components and containers. This ensures consistent timestamps in logs and internal operations. Uses standard `Area/Location` format.
    *   **Default**: `America/Denver`
    *   **Example**: `Europe/London`, `Asia/Tokyo`
    *   **Used By**: `m3tal-dashboard`, `Ollama`, `Traefik gateway`, `Cloudflared`, `m3tal-api` (daemon, implicitly if systemd service inherits host TZ).

*   **`DEBUG_MODE`**
    *   **Description**: When set to `true`, enables more verbose logging and potentially activates development-specific features or debugging endpoints across M3TAL components.
    *   **Default**: `false`
    *   **Example**: `true`
    *   **Used By**: `m3tal-api` (daemon), `m3tal-dashboard` (container), `CLI binary`, other containers.

*   **`METRICS_ENABLED`**
    *   **Description**: Controls whether performance metrics and monitoring endpoints are exposed by M3TAL components, allowing for integration with monitoring tools.
    *   **Default**: `true`
    *   **Example**: `false`
    *   **Used By**: `m3tal-api` (daemon), `m3tal-dashboard` (container), `Traefik gateway`.