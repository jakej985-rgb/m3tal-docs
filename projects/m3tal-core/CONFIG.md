# Environment Variables Reference

All M3TAL environment variables are centrally managed and read from `/etc/m3tal/.env`. This file serves as the primary configuration source for both the `m3tal` CLI binary and all Docker Compose stacks, which load variables via the `--env-file` option. You can manage these variables using `m3tal config wizard` for guided setup or `m3tal config set KEY value` for direct changes.

## Quick Reference

| Name                    | Default Value             | Component(s)                         | Description                                                               |
| :---------------------- | :------------------------ | :----------------------------------- | :------------------------------------------------------------------------ |
| `API_PORT`              | `5050`                    | M3TAL API daemon                     | Alternative or internal API port.                                         |
| `DASHBOARD_PORT`        | `8082`                    | Dashboard, Traefik                   | Port for the M3TAL Dashboard container.                                   |
| `DASHBOARD_EXPOSE_MODE` | `local`                   | CLI, Dashboard                       | Controls Dashboard exposure (`local` or `traefik`).                       |
| `HTTP_PORT`             | `8080`                    | M3TAL API daemon                     | Default HTTP port for core M3TAL API daemon.                              |
| `STATE_DIR`             | `./state`                 | API daemon, Dashboard                | Path for state database and runtime files.                                |
| `LOG_LEVEL`             | `info`                    | M3TAL API daemon                     | Logging severity level (`debug`, `info`, `warn`, `error`).                |
| `DASHBOARD_SECRET`      | `change_me_immediately`   | M3TAL Dashboard                      | Secret key for Dashboard session management.                              |
| `API_TOKEN`             | `change_me_api_token`     | API daemon, CLI                      | Authentication token for the M3TAL API.                                   |
| `ADMIN_PASSWORD`        | `admin_pass`              | M3TAL Dashboard                      | Initial password for the Dashboard admin user.                            |
| `NETWORK_NAME`          | `m3tal`                   | Docker Compose (all stacks)          | Name of the Docker network for inter-container communication.             |
| `LOCAL_IP`              | `127.0.0.1`               | M3TAL API daemon                     | Local IP address of the host machine.                                     |
| `DOMAIN`                | `localhost`               | Traefik, Dashboard                   | Base domain for Traefik routing (`dash.DOMAIN`, `api.DOMAIN`).            |
| `VPN_USER`              | `user`                    | VPN stack                            | Username for VPN client configuration.                                    |
| `VPN_PASSWORD`          | `password`                | VPN stack                            | Password for VPN client configuration.                                    |
| `BASE_STORAGE_PATH`     | `./data`                  | Dashboard, User Stacks               | Root directory for all M3TAL media and configuration.                     |
| `MEDIA_PATH`            | `./data/media`            | User Stacks                          | Subdirectory for media files.                                             |
| `CONFIG_PATH`           | `./data/config`           | Dashboard, User Stacks               | Subdirectory for configuration files.                                     |
| `DOWNLOADS_PATH`        | `./data/downloads`        | User Stacks                          | Subdirectory for downloaded files.                                        |
| `PUID`                  | `1000`                    | All Docker Containers                | Linux User ID (UID) for containers.                                       |
| `PGID`                  | `1000`                    | All Docker Containers                | Linux Group ID (GID) for containers.                                      |
| `TZ`                    | `America/Denver`          | All Docker Containers                | Timezone for containers and services.                                     |
| `TRAEFIK_WEB_PORT`      | `80`                      | Traefik gateway                      | Host port for Traefik's HTTP entry point.                                 |
| `TRAEFIK_WEBHTTPS_PORT` | `443`                     | Traefik gateway                      | Host port for Traefik's HTTPS entry point.                                |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                    | Traefik gateway                      | Internal port for Traefik's own dashboard.                                |
| `DEBUG_MODE`            | `false`                   | API daemon, Dashboard                | Enables debug logging and features.                                       |
| `METRICS_ENABLED`       | `true`                    | M3TAL API daemon                     | Enables Prometheus-compatible metrics endpoints.                          |

---

## Detailed Reference

### Core Variables

These variables control fundamental aspects of the M3TAL system's operation.

#### `API_PORT`

*   **Description**: Specifies an alternative or internal port for the M3TAL API daemon. While the API daemon typically listens on the port defined by `HTTP_PORT`, this variable may be used for specific internal configurations.
*   **Default Value**: `5050`
*   **Example Value**: `5050`
*   **Component(s) Used By**: M3TAL API daemon (potential, configuration dependent)

#### `HTTP_PORT`

*   **Description**: The default HTTP port for core M3TAL services, specifically the M3TAL API daemon. Traefik routes internal requests to this port for the API daemon.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Component(s) Used By**: M3TAL API daemon

#### `STATE_DIR`

*   **Description**: Defines the path to the directory where M3TAL stores its SQLite state database (`state.db`) and other critical runtime files. In a production environment, this typically maps to `/var/lib/m3tal/state`.
*   **Default Value**: `./state`
*   **Example Value**: `/var/lib/m3tal/state`
*   **Component(s) Used By**: M3TAL API daemon, M3TAL Dashboard container (via volume mount)

#### `LOG_LEVEL`

*   **Description**: Sets the minimum severity level for log messages generated by the M3TAL API daemon. Valid values, in order of increasing severity, are `debug`, `info`, `warn`, and `error`.
*   **Default Value**: `info`
*   **Example Value**: `debug`
*   **Component(s) Used By**: M3TAL API daemon

#### `DEBUG_MODE`

*   **Description**: Enables or disables debug logging and additional debugging features for core M3TAL components, such as the API daemon and Dashboard.
*   **Default Value**: `false`
*   **Example Value**: `true`
*   **Component(s) Used By**: M3TAL API daemon, M3TAL Dashboard (potentially)

#### `METRICS_ENABLED`

*   **Description**: Controls whether Prometheus-compatible metrics endpoints are exposed by the M3TAL API daemon.
*   **Default Value**: `true`
*   **Example Value**: `false`
*   **Component(s) Used By**: M3TAL API daemon

### Authentication Variables

These variables are critical for securing access to M3TAL services.

#### `DASHBOARD_SECRET`

*   **Description**: A secret key used by the M3TAL Dashboard for session management, encryption, and other security-sensitive operations. **This variable is automatically generated by `m3tal init` on first setup and should NOT be set manually unless you are intentionally rotating it.**
*   **Default Value**: `change_me_immediately`
*   **Example Value**: `aRandomLongAndSecureStringGeneratedByM3TAL`
*   **Component(s) Used By**: M3TAL Dashboard container

#### `API_TOKEN`

*   **Description**: An authentication token used to secure access to the M3TAL API. This token is required by the CLI and any other clients interacting with the API daemon. **This variable is automatically generated by `m3tal init` on first setup and should NOT be set manually unless you are intentionally rotating it.**
*   **Default Value**: `change_me_api_token`
*   **Example Value**: `anotherRandomLongAndSecureTokenForApiAccess`
*   **Component(s) Used By**: M3TAL API daemon, CLI

#### `ADMIN_PASSWORD`

*   **Description**: The initial default password for the `admin` user account within the M3TAL Dashboard, stored in `/docker/users.json`. This password can be managed using the `m3tal dashpass` command.
*   **Default Value**: `admin_pass`
*   **Example Value**: `MySecureDashPass123!`
*   **Component(s) Used By**: M3TAL Dashboard container

### Network Variables

These variables configure network-related settings for M3TAL services.

#### `DASHBOARD_PORT`

*   **Description**: The internal port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly exposed on the host. When `DASHBOARD_EXPOSE_MODE` is `traefik`, Traefik routes to this internal port.
*   **Default Value**: `8082`
*   **Example Value**: `8082`
*   **Component(s) Used By**: M3TAL Dashboard container, Traefik gateway (when `DASHBOARD_EXPOSE_MODE=traefik`)

#### `DASHBOARD_EXPOSE_MODE`

*   **Description**: Controls how the M3TAL Dashboard is made accessible.
    *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly bound to the host, accessible via `http://HOST_IP:8082`. Ideal for local network access.
    *   `traefik`: The dashboard is exposed via the Traefik reverse proxy, accessible at `http://dash.DOMAIN`. Requires Traefik to be running.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Component(s) Used By**: CLI (for `m3tal dash up` command to select appropriate compose override), M3TAL Dashboard container (indirectly via compose overrides)

#### `NETWORK_NAME`

*   **Description**: Specifies the name of the Docker network created and used for inter-container communication across M3TAL's Docker Compose stacks. The `proxy` network is often used for services requiring Traefik integration.
*   **Default Value**: `m3tal`
*   **Example Value**: `proxy`
*   **Component(s) Used By**: Docker Compose (all stacks)

#### `LOCAL_IP`

*   **Description**: The local IP address of the host machine. This variable can be used by services that need to reference the host's network interface directly for internal routing or specific configurations.
*   **Default Value**: `127.0.0.1`
*   **Example Value**: `192.168.1.100`
*   **Component(s) Used By**: M3TAL API daemon (potentially for binding/referencing)

### Storage Variables

These variables define the filesystem paths for various types of M3TAL data.

#### `BASE_STORAGE_PATH`

*   **Description**: The root directory for all M3TAL media, configuration, and state data. All other storage-related paths (`MEDIA_PATH`, `CONFIG_PATH`, `DOWNLOADS_PATH`) are typically subdirectories relative to this path. **In production deployments, this variable defaults to `/mnt` to align with common Linux server conventions for mounting external storage, rather than `./data` as in the template.**
*   **Default Value**: `./data`
*   **Example Value**: `/mnt/m3tal_data`
*   **Component(s) Used By**: M3TAL Dashboard container, other user stacks (for volume mounts)

#### `MEDIA_PATH`

*   **Description**: Defines the subdirectory within `BASE_STORAGE_PATH` where media files (e.g., photos, videos, audio) are stored.
*   **Default Value**: `./data/media`
*   **Example Value**: `/mnt/m3tal_data/media`
*   **Component(s) Used By**: User stacks (e.g., media servers), M3TAL Dashboard (for media access)

#### `CONFIG_PATH`

*   **Description**: Defines the subdirectory within `BASE_STORAGE_PATH` where configuration files specific to M3TAL services and user stacks are stored. This includes the Dashboard's `users.json`.
*   **Default Value**: `./data/config`
*   **Example Value**: `/mnt/m3tal_data/config`
*   **Component(s) Used By**: M3TAL Dashboard container (for `users.json`), user stacks

#### `DOWNLOADS_PATH`

*   **Description**: Defines the subdirectory within `BASE_STORAGE_PATH` designated for downloaded files.
*   **Default Value**: `./data/downloads`
*   **Example Value**: `/mnt/m3tal_data/downloads`
*   **Component(s) Used By**: User stacks (e.g., download clients)

### Traefik Variables

These variables configure the Traefik reverse proxy gateway for routing M3TAL services.

#### `DOMAIN`

*   **Description**: The base domain name used for M3TAL services when Traefik is enabled. Setting this variable correctly allows Traefik to create routing rules such as `dash.DOMAIN` for the Dashboard and `api.DOMAIN` for the M3TAL API.
*   **Default Value**: `localhost`
*   **Example Value**: `mydomain.com`
*   **Component(s) Used By**: Traefik gateway, M3TAL Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)

#### `TRAEFIK_WEB_PORT`

*   **Description**: The host machine's port that Traefik binds to for its HTTP entry point, typically used for non-HTTPS traffic.
*   **Default Value**: `80`
*   **Example Value**: `80`
*   **Component(s) Used By**: Traefik gateway

#### `TRAEFIK_WEBHTTPS_PORT`

*   **Description**: The host machine's port that Traefik binds to for its HTTPS entry point, typically used for secure web traffic.
*   **Default Value**: `443`
*   **Example Value**: `443`
*   **Component(s) Used By**: Traefik gateway

#### `TRAEFIK_DASHBOARD_PORT`

*   **Description**: The internal port on which the Traefik dashboard service listens within its container. This port is typically mapped to `127.0.0.1:8081` on the host for local access.
*   **Default Value**: `8080`
*   **Example Value**: `8080`
*   **Component(s) Used By**: Traefik gateway

### VPN Variables

These variables are used for configuring VPN-related services, such as Cloudflared tunnels.

#### `VPN_USER`

*   **Description**: Specifies the username to be used for VPN client configurations (e.g., within a WireGuard or OpenVPN client container).
*   **Default Value**: `user`
*   **Example Value**: `m3talvpn`
*   **Component(s) Used By**: VPN stack (if deployed)

#### `VPN_PASSWORD`

*   **Description**: Specifies the password to be used in conjunction with `VPN_USER` for VPN client configurations.
*   **Default Value**: `password`
*   **Example Value**: `vPnS3cur3P@ss`
*   **Component(s) Used By**: VPN stack (if deployed)

### System Variables

These variables control system-level settings for Docker containers and the host environment.

#### `PUID`

*   **Description**: Sets the Linux User ID (UID) for containers. This ensures that files created or modified by containers have the correct permissions on the host filesystem, matching a specific user.
*   **Default Value**: `1000`
*   **Example Value**: `1000`
*   **Component(s) Used By**: All Docker containers (via `user` directive in compose files)

#### `PGID`

*   **Description**: Sets the Linux Group ID (GID) for containers. Similar to `PUID`, this ensures correct group ownership and permissions for files accessed by containers on the host filesystem.
*   **Default Value**: `1000`
*   **Example Value**: `1000`
*   **Component(s) Used By**: All Docker containers (via `user` directive in compose files)

#### `TZ`

*   **Description**: Sets the timezone for Docker containers and M3TAL services. Use standard IANA timezone database names (e.g., `America/New_York`, `Europe/London`, `Asia/Tokyo`).
*   **Default Value**: `America/Denver`
*   **Example Value**: `Europe/Berlin`
*   **Component(s) Used By**: All Docker containers