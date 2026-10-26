# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables serve as the primary configuration mechanism for both the M3TAL CLI and all associated Docker Compose stacks.

All variables are read from the central configuration file, `/etc/m3tal/.env`. The `m3tal config wizard` and `m3tal config set` commands are the recommended way to manage these values. Both the `m3tal` CLI and all Docker Compose stacks leverage this file via the `--env-file /etc/m3tal/.env` argument, ensuring a consistent configuration across the entire system.

## Quick Reference Table

| Name                  | Default               | Description                                                                                                   | Used By                                              |
|-----------------------|-----------------------|---------------------------------------------------------------------------------------------------------------|------------------------------------------------------|
| `API_PORT`            | `5050`                | Internal port for the M3TAL API daemon (used by dashboard for `host.docker.internal` connection).             | `m3tal-dashboard`                                    |
| `DASHBOARD_PORT`      | `8082`                | Port the M3TAL Dashboard container listens on.                                                                | `m3tal-dashboard`, `CLI` (local mode)                |
| `DASHBOARD_EXPOSE_MODE`| `local`               | Determines how the M3TAL Dashboard is exposed: `local` (direct port binding) or `traefik` (via Traefik).    | `CLI`, `m3tal-dashboard`                             |
| `HTTP_PORT`           | `8080`                | Host-facing port for the M3TAL API daemon (`m3tal-api.service`).                                              | `m3tal-api.service`, `traefik`                       |
| `STATE_DIR`           | `./state`             | Path to the M3TAL API daemon's SQLite state database (`/var/lib/m3tal/state.db` in production).               | `m3tal-api.service`, `m3tal-dashboard`               |
| `LOG_LEVEL`           | `info`                | Logging verbosity for the M3TAL API daemon.                                                                   | `m3tal-api.service`                                  |
| `DASHBOARD_SECRET`    | `change_me_immediately`| Secret key for M3TAL Dashboard session management.                                                            | `m3tal-dashboard`                                    |
| `API_TOKEN`           | `change_me_api_token` | Bearer token for authenticating CLI and external API requests.                                                | `CLI`, `m3tal-api.service`                           |
| `ADMIN_PASSWORD`      | `admin_pass`          | Default administrator password for the M3TAL Dashboard.                                                       | `m3tal-dashboard`                                    |
| `NETWORK_NAME`        | `m3tal`               | Name of the Docker network used by M3TAL services.                                                            | `m3tal-dashboard`, `traefik`, `cloudflared`, User Stacks |
| `LOCAL_IP`            | `127.0.0.1`           | Local IP address of the host, often used for internal binds or routing within Compose.                        | `CLI`, `traefik`                                     |
| `DOMAIN`              | `localhost`           | Base domain for Traefik routing, enabling `dash.DOMAIN` and `api.DOMAIN` routes.                              | `traefik`, `m3tal-dashboard` (traefik mode), `cloudflared` |
| `VPN_USER`            | `user`                | Username for VPN services, if integrated (e.g., WireGuard client).                                            | `cloudflared` (potential), User VPN Stacks           |
| `VPN_PASSWORD`        | `password`            | Password for VPN services.                                                                                    | `cloudflared` (potential), User VPN Stacks           |
| `BASE_STORAGE_PATH`   | `./data`              | Base directory for all M3TAL-related media and configuration data.                                            | `m3tal-dashboard`, `CLI`, User Stacks                |
| `MEDIA_PATH`          | `./data/media`        | Path for media files, relative to `BASE_STORAGE_PATH`.                                                        | `m3tal-dashboard`, User Stacks                       |
| `CONFIG_PATH`         | `./data/config`       | Path for configuration files, relative to `BASE_STORAGE_PATH`.                                                | `m3tal-dashboard`, `CLI`, User Stacks                |
| `DOWNLOADS_PATH`      | `./data/downloads`    | Path for downloaded files, relative to `BASE_STORAGE_PATH`.                                                   | `m3tal-dashboard`, User Stacks                       |
| `PUID`                | `1000`                | User ID to run Docker containers as, for file permissions.                                                    | `m3tal-dashboard`, User Stacks                       |
| `PGID`                | `1000`                | Group ID to run Docker containers as, for file permissions.                                                   | `m3tal-dashboard`, User Stacks                       |
| `TZ`                  | `America/Denver`      | Timezone for containers and services to ensure correct time synchronization.                                  | `m3tal-dashboard`, `traefik`, `cloudflared`, User Stacks |
| `TRAEFIK_WEB_PORT`    | `80`                  | Port Traefik listens on for HTTP traffic.                                                                     | `traefik`, `CLI`                                     |
| `TRAEFIK_WEBHTTPS_PORT`| `443`                 | Port Traefik listens on for HTTPS traffic.                                                                    | `traefik`, `CLI`                                     |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                | Internal port for the Traefik dashboard.                                                                      | `traefik`                                            |
| `DEBUG_MODE`          | `false`               | Enables debug logging and features for the API daemon and Dashboard.                                          | `m3tal-api.service`, `m3tal-dashboard`               |
| `METRICS_ENABLED`     | `true`                | Enables collection and exposure of application metrics by the API daemon.                                     | `m3tal-api.service`                                  |

---

## Detailed Variable Reference

### Core Variables

These variables control the fundamental operation and connectivity of the M3TAL API daemon and Dashboard.

### `API_PORT`

- **Description:** Specifies the internal port on which the M3TAL API daemon is expected to be reachable by other Docker containers, particularly the `m3tal-dashboard`. The dashboard connects to `http://host.docker.internal:${API_PORT}`.
- **Default Value:** `5050`
- **Example Value:** `5050`
- **Used By:** `m3tal-dashboard`

### `DASHBOARD_PORT`

- **Description:** Defines the port on which the `m3tal-dashboard` container listens. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly mapped to the host.
- **Default Value:** `8082`
- **Example Value:** `8000`
- **Used By:** `m3tal-dashboard`, `CLI` (for local mode port binding)

### `DASHBOARD_EXPOSE_MODE`

- **Description:** Controls how the M3TAL Dashboard is made accessible.
    - `local`: The dashboard port (`DASHBOARD_PORT`) is directly published to the host, making it accessible via `http://HOST_IP:DASHBOARD_PORT`. This is ideal for LAN-only setups or first-time use, requiring no Traefik configuration.
    - `traefik`: The dashboard container is configured with Traefik labels, allowing Traefik to route `dash.${DOMAIN}` to the dashboard on its internal port 8082. This requires Traefik to be running.
- **Default Value:** `local`
- **Example Value:** `traefik`
- **Used By:** `CLI` (to select appropriate compose override), `m3tal-dashboard` (via Traefik labels)

### `HTTP_PORT`

- **Description:** This is the primary host-facing port on which the `m3tal-api.service` daemon listens for incoming requests. It's the port that Traefik routes `api.${DOMAIN}` to via `http://host.docker.internal:${HTTP_PORT}`.
- **Default Value:** `8080`
- **Example Value:** `9000`
- **Used By:** `m3tal-api.service`, `traefik`

### `STATE_DIR`

- **Description:** Specifies the directory where the M3TAL API daemon stores its SQLite state database (`state.db`). In production deployments, this path is typically `/var/lib/m3tal/state.db`.
- **Default Value:** `./state`
- **Example Value:** `/var/lib/m3tal`
- **Used By:** `m3tal-api.service`, `m3tal-dashboard` (to mount the state directory)

### `LOG_LEVEL`

- **Description:** Sets the verbosity of logging for the M3TAL API daemon. Possible values typically include `debug`, `info`, `warn`, `error`, `fatal`.
- **Default Value:** `info`
- **Example Value:** `debug`
- **Used By:** `m3tal-api.service`

### `DEBUG_MODE`

- **Description:** When set to `true`, enables debug logging and additional development/troubleshooting features within the M3TAL API daemon and Dashboard.
- **Default Value:** `false`
- **Example Value:** `true`
- **Used By:** `m3tal-api.service`, `m3tal-dashboard`

### `METRICS_ENABLED`

- **Description:** When set to `true`, enables the collection and exposure of application metrics by the M3TAL API daemon, typically for monitoring purposes.
- **Default Value:** `true`
- **Example Value:** `false`
- **Used By:** `m3tal-api.service`

### Authentication Variables

These variables control access and security for the M3TAL Dashboard and API.

### `DASHBOARD_SECRET`

- **Description:** A unique secret key used by the `m3tal-dashboard` for session management, token signing, and other cryptographic operations. **This value is auto-generated on first `m3tal init` and should NOT be set manually unless you are intentionally rotating it.**
- **Default Value:** `change_me_immediately`
- **Example Value:** `super_secret_dashboard_key_12345`
- **Used By:** `m3tal-dashboard`

### `API_TOKEN`

- **Description:** A Bearer token used for authenticating requests to the M3TAL API daemon from the CLI or other external clients. **This value is auto-generated on first `m3tal init` and should NOT be set manually unless you are intentionally rotating it.**
- **Default Value:** `change_me_api_token`
- **Example Value:** `Bearer_abc123def456ghi789`
- **Used By:** `CLI`, `m3tal-api.service`

### `ADMIN_PASSWORD`

- **Description:** The default administrator password for accessing the M3TAL Dashboard. It's highly recommended to change this immediately after initial setup using `m3tal dashpass`.
- **Default Value:** `admin_pass`
- **Example Value:** `MyStrongPassword123!`
- **Used By:** `m3tal-dashboard` (for initial user creation/default login)

### Network Variables

These variables configure network settings and routing for M3TAL services.

### `NETWORK_NAME`

- **Description:** The name of the Docker network that M3TAL uses for internal communication between its containers and other user-defined stacks. This network must be `external: true`.
- **Default Value:** `m3tal`
- **Example Value:** `my-m3tal-network`
- **Used By:** `m3tal-dashboard`, `traefik`, `cloudflared`, User Stacks

### `LOCAL_IP`

- **Description:** Specifies the local IP address of the host machine. This can be used for internal service binds or when services need to reference the host's IP within the Docker Compose network.
- **Default Value:** `127.0.0.1`
- **Example Value:** `192.168.1.100`
- **Used By:** `CLI`, internal service configurations (e.g., Traefik dynamic routing to host services)

### `DOMAIN`

- **Description:** The base domain name used for Traefik routing rules. Setting this variable enables routes like `dash.${DOMAIN}` for the dashboard and `api.${DOMAIN}` for the API daemon (when `DASHBOARD_EXPOSE_MODE` is `traefik`).
- **Default Value:** `localhost`
- **Example Value:** `example.com`
- **Used By:** `traefik`, `m3tal-dashboard` (via Traefik labels), `cloudflared` (for tunnel configurations)

### Storage Variables

These variables define the filesystem paths for various types of data managed by M3TAL.

### `BASE_STORAGE_PATH`

- **Description:** The root directory for all M3TAL-related media and configuration data. All other storage paths (`MEDIA_PATH`, `CONFIG_PATH`, `DOWNLOADS_PATH`) are typically relative to this base path. **In production deployments, this defaults to `/mnt` (e.g., for persistent storage on a dedicated volume), not `./data` as in the template.**
- **Default Value:** `./data`
- **Example Value:** `/mnt/m3tal-data`
- **Used By:** `m3tal-dashboard`, `CLI`, User Stacks (for volume mounts)

### `MEDIA_PATH`

- **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
- **Default Value:** `./data/media`
- **Example Value:** `${BASE_STORAGE_PATH}/media`
- **Used By:** `m3tal-dashboard`, User Stacks

### `CONFIG_PATH`

- **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL configuration files (e.g., `users.json`) are stored.
- **Default Value:** `./data/config`
- **Example Value:** `${BASE_STORAGE_PATH}/config`
- **Used By:** `m3tal-dashboard`, `CLI`, User Stacks

### `DOWNLOADS_PATH`

- **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are typically saved.
- **Default Value:** `./data/downloads`
- **Example Value:** `${BASE_STORAGE_PATH}/downloads`
- **Used By:** `m3tal-dashboard`, User Stacks

### Traefik Variables

These variables configure the Traefik reverse proxy.

### `TRAEFIK_WEB_PORT`

- **Description:** The port Traefik binds on the host for handling incoming HTTP (`web`) traffic.
- **Default Value:** `80`
- **Example Value:** `8080`
- **Used By:** `traefik`, `CLI`

### `TRAEFIK_WEBHTTPS_PORT`

- **Description:** The port Traefik binds on the host for handling incoming HTTPS (`websecure`) traffic.
- **Default Value:** `443`
- **Example Value:** `8443`
- **Used By:** `traefik`, `CLI`

### `TRAEFIK_DASHBOARD_PORT`

- **Description:** The internal port on which the Traefik dashboard listens. This is typically exposed only on `127.0.0.1` on the host, making it accessible via `http://127.0.0.1:8081` by default.
- **Default Value:** `8080`
- **Example Value:** `8080`
- **Used By:** `traefik`

### VPN Variables

These variables are used for configuring VPN services, especially with the optional `cloudflared` tunnel.

### `VPN_USER`

- **Description:** Username for VPN services, such as when configuring a WireGuard client or a similar VPN connection.
- **Default Value:** `user`
- **Example Value:** `m3taluser`
- **Used By:** `cloudflared` (potential), User VPN Stacks

### `VPN_PASSWORD`

- **Description:** Password associated with the `VPN_USER` for VPN services.
- **Default Value:** `password`
- **Example Value:** `StrongVPnPa$$word!`
- **Used By:** `cloudflared` (potential), User VPN Stacks

### System Variables

These variables control general system-level configurations for containers.

### `PUID`

- **Description:** Specifies the User ID (UID) under which Docker containers should run. This is crucial for ensuring correct file permissions when containers interact with host-mounted volumes.
- **Default Value:** `1000`
- **Example Value:** `1001`
- **Used By:** `m3tal-dashboard`, User Stacks

### `PGID`

- **Description:** Specifies the Group ID (GID) under which Docker containers should run. This, along with `PUID`, helps manage file ownership and permissions for host-mounted volumes.
- **Default Value:** `1000`
- **Example Value:** `1001`
- **Used By:** `m3tal-dashboard`, User Stacks

### `TZ`

- **Description:** Sets the timezone for containers and services, ensuring accurate timestamps and time-based operations within the M3TAL ecosystem. Follows standard timezone database names (e.g., `America/New_York`, `Europe/London`).
- **Default Value:** `America/Denver`
- **Example Value:** `America/New_York`
- **Used By:** `m3tal-dashboard`, `traefik`, `cloudflared`, User Stacks