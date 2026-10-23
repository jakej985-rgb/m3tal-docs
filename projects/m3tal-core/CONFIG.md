```markdown
# M3TAL Environment Variables Reference

Greetings, M3TAL architect. This document, forged by DocSmith himself, serves as the definitive reference for all environment variables within the M3TAL ecosystem.

All environment variables are read from the primary M3TAL configuration file: `/etc/m3tal/.env`. Both the `m3tal` CLI binary and all Docker Compose stacks utilize this file via the `--env-file` option, ensuring a unified configuration.

You can manage these variables using the `m3tal config wizard` for an interactive setup, or `m3tal config set KEY value` for direct manipulation.

---

## Quick Reference

| Variable Name             | Default Value          | Description                                                                                                                                                                                                                                                                          | Component(s)                                   |
| :------------------------ | :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------- |
| `API_PORT`                | `5050`                 | Internal port for the M3TAL API daemon.                                                                                                                                                                                                                                              | M3TAL API, Dashboard                           |
| `DASHBOARD_PORT`          | `8082`                 | Internal port for the M3TAL Dashboard.                                                                                                                                                                                                                                               | M3TAL Dashboard, M3TAL CLI                     |
| `DASHBOARD_EXPOSE_MODE`   | `local`                | Controls how the Dashboard is exposed: `local` (direct port) or `traefik` (via `dash.DOMAIN`).                                                                                                                                                                                         | M3TAL CLI                                      |
| `HTTP_PORT`               | `8080`                 | External port for the M3TAL API daemon, typically routed via Traefik.                                                                                                                                                                                                                  | M3TAL API, Traefik                             |
| `STATE_DIR`               | `./state`              | General-purpose directory for state files.                                                                                                                                                                                                                                           | M3TAL Dashboard                                |
| `LOG_LEVEL`               | `info`                 | Logging verbosity (`debug`, `info`, `warn`, `error`).                                                                                                                                                                                                                               | M3TAL API, CLI, Dashboard                      |
| `DASHBOARD_SECRET`        | `change_me_immediately`| Secret for Dashboard session management. **Auto-generated on `m3tal init`.**                                                                                                                                                                                                         | M3TAL Dashboard                                |
| `API_TOKEN`               | `change_me_api_token`  | Token for M3TAL API authentication. **Auto-generated on `m3tal init`.**                                                                                                                                                                                                              | M3TAL API, CLI, Dashboard                      |
| `ADMIN_PASSWORD`          | `admin_pass`           | Default password for the Dashboard admin user.                                                                                                                                                                                                                                       | M3TAL Dashboard                                |
| `NETWORK_NAME`            | `m3tal`                | Name of the Docker network for M3TAL services.                                                                                                                                                                                                                                       | M3TAL CLI, Docker Compose stacks               |
| `LOCAL_IP`                | `127.0.0.1`            | Local IP for internal routing.                                                                                                                                                                                                                                                       | Traefik, various services                      |
| `DOMAIN`                  | `localhost`            | Primary domain for Traefik routing. Enables `dash.DOMAIN` and `api.DOMAIN`.                                                                                                                                                                                                          | Traefik, M3TAL CLI                             |
| `VPN_USER`                | `user`                 | Username for optional VPN services.                                                                                                                                                                                                                                                  | Cloudflared, external VPNs                     |
| `VPN_PASSWORD`            | `password`             | Password for optional VPN services.                                                                                                                                                                                                                                                  | Cloudflared, external VPNs                     |
| `BASE_STORAGE_PATH`       | `./data`               | Host path for all persistent data. **Defaults to `/mnt` in production.**                                                                                                                                                                                                             | M3TAL API, Dashboard, Docker Compose stacks    |
| `MEDIA_PATH`              | `./data/media`         | Subdirectory for media files.                                                                                                                                                                                                                                                        | M3TAL API, Dashboard, Docker Compose stacks    |
| `CONFIG_PATH`             | `./data/config`        | Subdirectory for application configurations.                                                                                                                                                                                                                                         | M3TAL API, Dashboard, Docker Compose stacks    |
| `DOWNLOADS_PATH`          | `./data/downloads`     | Subdirectory for downloads.                                                                                                                                                                                                                                                          | M3TAL API, Dashboard, Docker Compose stacks    |
| `PUID`                    | `1000`                 | User ID for container file permissions.                                                                                                                                                                                                                                              | All Docker containers                          |
| `PGID`                    | `1000`                 | Group ID for container file permissions.                                                                                                                                                                                                                                             | All Docker containers                          |
| `TZ`                      | `America/Denver`       | Timezone for Docker containers.                                                                                                                                                                                                                                                      | All Docker containers                          |
| `TRAEFIK_WEB_PORT`        | `80`                   | Host port Traefik binds for HTTP.                                                                                                                                                                                                                                                    | Traefik gateway                                |
| `TRAEFIK_WEBHTTPS_PORT`   | `443`                  | Host port Traefik binds for HTTPS.                                                                                                                                                                                                                                                   | Traefik gateway                                |
| `TRAEFIK_DASHBOARD_PORT`  | `8080`                 | Internal port for Traefik's dashboard.                                                                                                                                                                                                                                               | Traefik gateway                                |
| `DEBUG_MODE`              | `false`                | Enables debug logging and features.                                                                                                                                                                                                                                                  | M3TAL API, CLI, Dashboard                      |
| `METRICS_ENABLED`         | `true`                 | Controls exposure of Prometheus metrics endpoints.                                                                                                                                                                                                                                   | M3TAL API, CLI                                 |

---

## Detailed Environment Variable Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system.

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon listens for internal communication, primarily used by the M3TAL Dashboard container (e.g., `GO_API_URL=http://host.docker.internal:${API_PORT}`).
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard
*   **`DASHBOARD_PORT`**
    *   **Description:** The internal port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host machine.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** M3TAL Dashboard container, M3TAL CLI (for `local` mode port mapping)
*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is exposed to users.
        *   `local`: The dashboard is exposed directly on `HOST_IP:DASHBOARD_PORT`. This mode requires no Traefik configuration and is ideal for LAN-only setups or initial testing.
        *   `traefik`: The dashboard is routed via the Traefik gateway at `http://dash.DOMAIN`. This mode requires Traefik to be running via `m3tal up` and `DOMAIN` to be configured.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** M3TAL CLI (determines which Docker Compose override file, `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`, is used for the dashboard)
*   **`HTTP_PORT`**
    *   **Description:** The port on which the M3TAL API daemon listens for external communication. This port is typically used by the Traefik gateway to route `api.DOMAIN` traffic to the API daemon (e.g., `http://host.docker.internal:${HTTP_PORT}`).
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** M3TAL API daemon, Traefik gateway
*   **`STATE_DIR`**
    *   **Description:** A general-purpose environment variable that specifies a directory for state or configuration files. Note that the M3TAL API daemon stores its primary SQLite database at `/var/lib/m3tal/state.db` by default, regardless of this variable. For the M3TAL Dashboard, its internal state directory `/docker/state` is mapped from the host path `${CONFIG_PATH}/m3tal/state`.
    *   **Default Value:** `./state`
    *   **Example Value:** `/opt/m3tal/state_data`
    *   **Used By:** M3TAL Dashboard (influences host volume mapping for internal `/docker/state`), potentially other user-defined stacks.
*   **`LOG_LEVEL`**
    *   **Description:** Sets the logging verbosity for M3TAL components. Available levels typically include `debug`, `info`, `warn`, and `error`. Setting to `debug` provides the most detailed output for troubleshooting.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** M3TAL API daemon, M3TAL CLI, M3TAL Dashboard
*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug logging and features across M3TAL components. Set to `true` for detailed diagnostics and development.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** M3TAL API daemon, M3TAL CLI, M3TAL Dashboard
*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether Prometheus-style metrics endpoints are exposed by M3TAL components, allowing for external monitoring.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** M3TAL API daemon, M3TAL CLI

### Authentication

Variables related to securing access to M3TAL services.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A critical secret key used by the M3TAL Dashboard for session management, encryption, and securing sensitive data. **This variable is automatically generated on the first run of `m3tal init`. Users should generally NOT set this manually unless performing a secret rotation.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_very_long_and_complex_random_string_for_security`
    *   **Used By:** M3TAL Dashboard container
*   **`API_TOKEN`**
    *   **Description:** An authentication token used to secure communication with the M3TAL API daemon. This token is required for the `m3tal` CLI and the M3TAL Dashboard to interact with the API. **This variable is automatically generated on the first run of `m3tal init`. Users should generally NOT set this manually unless performing a token rotation.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `a_unique_alphanumeric_api_key_string`
    *   **Used By:** M3TAL API daemon, M3TAL CLI, M3TAL Dashboard
*   **`ADMIN_PASSWORD`**
    *   **Description:** The default password for the initial administrator user of the M3TAL Dashboard. It is highly recommended to change this password immediately upon first login.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `StrongPa$$w0rd`
    *   **Used By:** M3TAL Dashboard (for initial `users.json` credential store setup)

### Network Configuration

Settings for Docker networking and host IP addresses.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the custom Docker network created and used by M3TAL to allow its core components and all user-defined Docker Compose stacks to communicate with each other.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal-proxy-net`
    *   **Used By:** M3TAL CLI (for creating the Docker network), all Docker Compose stacks
*   **`LOCAL_IP`**
    *   **Description:** Defines the local IP address of the host machine. This can be used for internal routing by services like Traefik to access host-bound services, or for specific network configurations within Docker containers.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** Traefik (for internal service routing, e.g., to host services like the API daemon), various Docker services.

### Storage Configuration

Defines paths for persistent data storage on the host filesystem.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The absolute path on the host filesystem that serves as the root directory for all M3TAL persistent media and application data. All other storage-related paths (`MEDIA_PATH`, `CONFIG_PATH`, `DOWNLOADS_PATH`) are typically subdirectories of this path. **In production deployments, this variable typically defaults to `/mnt` (or `/mnt/m3tal-data`) for robust storage, not `./data` as often seen in development templates.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard, all user Docker Compose stacks (for data volumes)
*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` designated for storing user media files.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal/media`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard, user Docker Compose stacks
*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` for application configuration files and mounted volumes for container configuration. This path is crucial as the M3TAL Dashboard's internal state directory (`/docker/state`) is mapped from `${CONFIG_PATH}/m3tal/state` on the host.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal/config`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard, user Docker Compose stacks
*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` reserved for user downloads or temporary files generated by M3TAL services.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal/downloads`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard, user Docker Compose stacks
*   **`PUID`**
    *   **Description:** The numeric User ID (UID) that Docker containers will use for file permissions when accessing mounted host volumes. Setting this to your host user's UID ensures proper read/write access and prevents permission issues.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** All Docker containers managed by M3TAL (via `user: "${PUID}:${PGID}"` in compose files)
*   **`PGID`**
    *   **Description:** The numeric Group ID (GID) that Docker containers will use for file permissions when accessing mounted host volumes. Setting this to your host user's GID ensures proper read/write access.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** All Docker containers managed by M3TAL (via `user: "${PUID}:${PGID}"` in compose files)

### Traefik Gateway

Variables specific to the Traefik reverse proxy and routing.

*   **`DOMAIN`**
    *   **Description:** The primary domain name under which your M3TAL services will be accessible via Traefik. Setting this variable enables Traefik routing rules, allowing access to the M3TAL Dashboard at `http://dash.DOMAIN` and the M3TAL API daemon at `http://api.DOMAIN`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `my-m3tal-server.com`
    *   **Used By:** Traefik gateway (for routing rules), M3TAL CLI (for generating Traefik dynamic configuration)
*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that the Traefik gateway binds to for handling incoming HTTP traffic. This is typically the standard HTTP port 80.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik gateway
*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that the Traefik gateway binds to for handling incoming HTTPS (secured HTTP) traffic. This is typically the standard HTTPS port 443.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik gateway
*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own administrative dashboard listens. This port is typically mapped to `127.0.0.1:8081` on the host, making the Traefik dashboard accessible locally.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik gateway

### VPN Configuration

Variables for optional VPN integration, such as Cloudflare Tunnels.

*   **`VPN_USER`**
    *   **Description:** A username for an optional VPN service. This could be used for configuring authenticated Cloudflare Tunnels or other external VPN client configurations managed by M3TAL.
    *   **Default Value:** `user`
    *   **Example Value:** `my_cloudflare_tunnel_user`
    *   **Used By:** Cloudflared (if configured for authenticated tunnels), or external VPN services.
*   **`VPN_PASSWORD`**
    *   **Description:** The password associated with `VPN_USER` for an optional VPN service.
    *   **Default Value:** `password`
    *   **Example Value:** `secure_tunnel_pass`
    *   **Used By:** Cloudflared (if configured for authenticated tunnels), or external VPN services.

### System Settings

General system-wide configurations for M3TAL components.

*   **`TZ`**
    *   **Description:** Sets the timezone for all Docker containers managed by M3TAL. This ensures consistent timestamps in logs and for applications running within containers.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `America/Los_Angeles`
    *   **Used By:** All Docker containers managed by M3TAL