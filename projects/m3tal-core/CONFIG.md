```markdown
# Environment Variables Reference

All M3TAL environment variables are read from the primary configuration file, `/etc/m3tal/.env`. This file is loaded by both the `m3tal` CLI binary and all Docker Compose stacks (via `--env-file`) to ensure consistent configuration across the entire ecosystem.

**Important Note:** Do not manually edit `/etc/m3tal/.env` directly after initial setup unless you know exactly what you are doing. Use `m3tal config wizard` or `m3tal config set KEY value` to manage these variables safely and ensure they are validated.

## Quick Reference

| Name                    | Description                                                                 | Default Value            | Example Value              | Component(s)                                                                       |
| :---------------------- | :-------------------------------------------------------------------------- | :----------------------- | :----------------------- | :--------------------------------------------------------------------------------- |
| `DASHBOARD_PORT`        | Port for the M3TAL Dashboard.                                               | `8082`                   | `8082`                     | Dashboard, CLI                                                                     |
| `DASHBOARD_EXPOSE_MODE` | Controls Dashboard access (`local` or `traefik`).                           | `local`                  | `traefik`                  | CLI                                                                                |
| `HTTP_PORT`             | Port for the M3TAL API daemon.                                              | `8080`                   | `8080`                     | API daemon, Dashboard, Traefik                                                     |
| `STATE_DIR`             | Directory for `state.db`.                                                   | `./state`                | `/var/lib/m3tal/state`     | API daemon, Dashboard                                                              |
| `LOG_LEVEL`             | Logging verbosity (`info`, `debug`, etc.).                                  | `info`                   | `debug`                    | API daemon, CLI                                                                    |
| `DASHBOARD_SECRET`      | Secret for Dashboard sessions. **Auto-generated.**                          | `change_me_immediately`  | `a-random-string`          | Dashboard, CLI                                                                     |
| `API_TOKEN`             | Bearer token for API authentication. **Auto-generated.**                    | `change_me_api_token`    | `another-random-string`    | API daemon, CLI                                                                    |
| `ADMIN_PASSWORD`        | Initial password for the Dashboard admin user.                              | `admin_pass`             | `myStrongPassword123!`     | Dashboard, CLI                                                                     |
| `NETWORK_NAME`          | Name of the M3TAL Docker network.                                           | `m3tal`                  | `m3tal`                    | Dashboard, Traefik, Cloudflared, User Stacks                                       |
| `LOCAL_IP`              | IP for `host.docker.internal` mapping.                                      | `127.0.0.1`              | `172.17.0.1`               | Dashboard, Traefik                                                                 |
| `DOMAIN`                | Base domain for Traefik routing (`dash.DOMAIN`, `api.DOMAIN`).              | `localhost`              | `yourserver.com`           | Traefik, Dashboard                                                                 |
| `VPN_USER`              | Username for user-deployed VPNs.                                            | `user`                   | `vpnclient`                | User Stacks                                                                        |
| `VPN_PASSWORD`          | Password for user-deployed VPNs.                                            | `password`               | `myVpnSecret`              | User Stacks                                                                        |
| `BASE_STORAGE_PATH`     | Root directory for all M3TAL data. **Production defaults to `/mnt`.**       | `./data`                 | `/mnt/m3tal-data`          | API daemon, Dashboard, User Stacks                                                 |
| `MEDIA_PATH`            | Subdirectory for media files.                                               | `./data/media`           | `/mnt/m3tal-data/media`    | API daemon, User Stacks                                                            |
| `CONFIG_PATH`           | Subdirectory for M3TAL config files (e.g., `users.json`).                   | `./data/config`          | `/mnt/m3tal-data/config`   | API daemon, Dashboard                                                              |
| `DOWNLOADS_PATH`        | Subdirectory for downloads.                                                 | `./data/downloads`       | `/mnt/m3tal-data/downloads`| API daemon, User Stacks                                                            |
| `PUID`                  | User ID for container file operations.                                      | `1000`                   | `1000`                     | Dashboard, User Stacks                                                             |
| `PGID`                  | Group ID for container file operations.                                     | `1000`                   | `1000`                     | Dashboard, User Stacks                                                             |
| `TZ`                    | Timezone for containers (TZ database format).                               | `America/Denver`         | `America/New_York`         | Dashboard, User Stacks                                                             |
| `TRAEFIK_WEB_PORT`      | Traefik's host port for HTTP (port 80).                                     | `80`                     | `80`                       | Traefik gateway                                                                    |
| `TRAEFIK_WEBHTTPS_PORT` | Traefik's host port for HTTPS (port 443).                                   | `443`                    | `443`                      | Traefik gateway                                                                    |
| `TRAEFIK_DASHBOARD_PORT`| Traefik's internal dashboard port (exposed on host 8081).                   | `8080`                   | `8080`                     | Traefik gateway                                                                    |
| `DEBUG_MODE`            | Enables verbose debugging for M3TAL components.                             | `false`                  | `true`                     | API daemon, CLI, Dashboard                                                         |
| `METRICS_ENABLED`       | Controls if API daemon exposes Prometheus metrics.                          | `true`                   | `false`                    | API daemon                                                                         |

---

## Detailed Reference

### Core Variables

These variables control fundamental aspects of the M3TAL system, including logging, debug features, and general container runtime settings.

*   **`HTTP_PORT`**
    *   **Description:** The internal port on which the M3TAL API daemon (`m3tal-api.service`) listens for requests. The Dashboard and Traefik gateway are configured to communicate with the API on this port.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component(s) Used By:** `m3tal-api.service`, `m3tal-dashboard` container (via `GO_API_URL`), `Traefik gateway` (for routing `api.DOMAIN`).

*   **`STATE_DIR`**
    *   **Description:** The host directory where the M3TAL API daemon stores its SQLite state database (`state.db`). This path is typically mounted as a volume into relevant containers.
    *   **Default Value:** `./state` (This is a template default. In production deployments, it maps to `/var/lib/m3tal/state`.)
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Component(s) Used By:** `m3tal-api.service`, `m3tal-dashboard` container (volume mount).

*   **`LOG_LEVEL`**
    *   **Description:** The minimum severity level for logs generated by the M3TAL API daemon and CLI. Options typically include `debug`, `info`, `warn`, `error`, `fatal`. Setting this to `debug` will provide more verbose output.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Component(s) Used By:** `m3tal-api.service`, `CLI binary`.

*   **`PUID`**
    *   **Description:** The User ID (UID) that M3TAL containers (and user-deployed containers) should use for file operations within mounted volumes, ensuring correct permissions on the host filesystem. Defaults to the first non-root user (usually `1000`).
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Component(s) Used By:** `m3tal-dashboard` container, User-deployed containers.

*   **`PGID`**
    *   **Description:** The Group ID (GID) that M3TAL containers (and user-deployed containers) should use for file operations within mounted volumes, ensuring correct permissions on the host filesystem. Defaults to the primary group of the user with `PUID` (usually `1000`).
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Component(s) Used By:** `m3tal-dashboard` container, User-deployed containers.

*   **`TZ`**
    *   **Description:** The timezone for M3TAL containers, specified in a standard TZ database name format (e.g., `America/Los_Angeles`).
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Component(s) Used By:** `m3tal-dashboard` container, User-deployed containers.

*   **`DEBUG_MODE`**
    *   **Description:** Enables verbose logging and additional debugging features across M3TAL components, including the API daemon and CLI.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Component(s) Used By:** `m3tal-api.service`, `CLI binary`, `m3tal-dashboard` container.

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether the M3TAL API daemon collects and exposes Prometheus-compatible metrics at `/metrics`.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Component(s) Used By:** `m3tal-api.service`.

### Authentication Variables

These variables are crucial for securing access to the M3TAL API and Dashboard.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A unique secret key used by the M3TAL Dashboard for session management, cookie signing, and other security-sensitive operations. **This value is automatically generated on the first `m3tal init` and should NOT be set manually unless you are intentionally rotating the secret.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a-very-long-and-random-string-of-characters`
    *   **Component(s) Used By:** `m3tal-dashboard` container, `CLI binary` (`m3tal init` for generation).

*   **`API_TOKEN`**
    *   **Description:** A bearer token required for authenticated API requests to the `m3tal-api.service`. This secures communication with the core API. **This value is automatically generated on the first `m3tal init` and should NOT be set manually unless you are intentionally rotating the token.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `another-long-and-random-api-key-for-m3tal`
    *   **Component(s) Used By:** `m3tal-api.service`, `CLI binary` (`m3tal init` for generation).

*   **`ADMIN_PASSWORD`**
    *   **Description:** The initial default password for the `admin` user on the M3TAL Dashboard. This is used for first-time setup or when resetting credentials via `m3tal dashpass`. It is highly recommended to change this after initial setup.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `myStrongPassword123!`
    *   **Component(s) Used By:** `m3tal-dashboard` container (for `users.json`), `CLI binary` (`m3tal dashpass`).

### Network Variables

These variables configure network settings, including how the M3TAL Dashboard is exposed and how services are routed.

*   **`DASHBOARD_PORT`**
    *   **Description:** The port on which the M3TAL Dashboard container listens internally (8082). When `DASHBOARD_EXPOSE_MODE` is set to `local`, this port is directly exposed on the host.
    *   **Default Value:** `8082`
    *   **Example Value:** `9000` (if you want to run the dashboard on a different host port)
    *   **Component(s) Used By:** `m3tal-dashboard` container, `CLI binary` (`m3tal dash up` for port mapping).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is made accessible.
        *   `local`: The dashboard port (`DASHBOARD_PORT`) is directly bound to the host, allowing access via `http://HOST_IP:DASHBOARD_PORT`.
        *   `traefik`: The dashboard is routed via the Traefik gateway under `dash.DOMAIN`. Requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Component(s) Used By:** `CLI binary` (determines which Docker Compose override file to use: `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).

*   **`NETWORK_NAME`**
    *   **Description:** The name of the custom Docker network used by M3TAL for inter-container communication. This network must be created (usually by `m3tal init`) for services to communicate.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal`
    *   **Component(s) Used By:** `m3tal-dashboard` container, `Traefik gateway`, `Cloudflared` container, and all other user-defined M3TAL-managed Docker stacks.

*   **`LOCAL_IP`**
    *   **Description:** The IP address used to resolve `host.docker.internal` from within containers. This allows containers to reach services running directly on the host machine (like the M3TAL API daemon).
    *   **Default Value:** `127.0.0.1` (may need to be `host-gateway` for some Docker environments)
    *   **Example Value:** `172.17.0.1` (a common Docker bridge IP)
    *   **Component(s) Used By:** `m3tal-dashboard` container (`extra_hosts` mapping), `Traefik gateway` (`dynamic/api.yml` for API routing).

*   **`DOMAIN`**
    *   **Description:** The base domain name used for Traefik routing rules. Setting this enables URL-based access to services like the Dashboard (`dash.DOMAIN`) and API (`api.DOMAIN`) when Traefik is active.
    *   **Default Value:** `localhost`
    *   **Example Value:** `yourserver.com`
    *   **Component(s) Used By:** `Traefik gateway` (`routing-compose.yml`, `dynamic/api.yml`), `m3tal-dashboard` container (`m3tal-compose.traefik.yml`).

### Storage Variables

These variables define the filesystem paths where M3TAL stores its persistent data.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host where all M3TAL-related persistent data (media, configuration, state, downloads) is stored. **In production deployments, this defaults to `/mnt` to leverage commonly mounted storage.**
    *   **Default Value:** `./data` (This is a template default. In production deployments, it's `/mnt`.)
    *   **Example Value:** `/mnt/m3tal-data`
    *   **Component(s) Used By:** `m3tal-api.service`, `m3tal-dashboard` container (volume mounts), and implicitly by all other data-storing user-deployed containers.

*   **`MEDIA_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` designated for user media files.
    *   **Default Value:** `./data/media` (derived from `./data`)
    *   **Example Value:** `/mnt/m3tal-data/media`
    *   **Component(s) Used By:** `m3tal-api.service`, User-deployed media containers.

*   **`CONFIG_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL stores its internal configuration files, such as `users.json`.
    *   **Default Value:** `./data/config` (derived from `./data`)
    *   **Example Value:** `/mnt/m3tal-data/config`
    *   **Component(s) Used By:** `m3tal-api.service`, `m3tal-dashboard` container (volume mount).

*   **`DOWNLOADS_PATH`**
    *   **Description:** The subdirectory within `BASE_STORAGE_PATH` used for managing downloads by M3TAL components or user-deployed download clients.
    *   **Default Value:** `./data/downloads` (derived from `./data`)
    *   **Example Value:** `/mnt/m3tal-data/downloads`
    *   **Component(s) Used By:** `m3tal-api.service`, User-deployed download client containers.

### Traefik Variables

These variables specifically configure the Traefik gateway for routing and its own dashboard access.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port that Traefik uses for its main HTTP (unencrypted) entry point. Services configured with Traefik using HTTP will be accessible via this port.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Component(s) Used By:** `Traefik gateway` container.

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port that Traefik uses for its main HTTPS (encrypted) entry point. Services configured with Traefik using HTTPS will be accessible via this port.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Component(s) Used By:** `Traefik gateway` container.

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own administration dashboard listens. This is typically mapped to `127.0.0.1:8081` on the host, making it accessible only locally.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component(s) Used By:** `Traefik gateway` container.

### VPN Variables

These variables are provided for configuring user-deployed VPN services, though M3TAL includes `cloudflared` for zero-config tunnel access.

*   **`VPN_USER`**
    *   **Description:** A generic username variable intended for use with external VPN services if configured within specific user-deployed containers. It is not directly used by core M3TAL components, but available for custom stacks.
    *   **Default Value:** `user`
    *   **Example Value:** `vpnclient`
    *   **Component(s) Used By:** User-deployed VPN containers (e.g., WireGuard, OpenVPN clients).

*   **`VPN_PASSWORD`**
    *   **Description:** A generic password variable intended for use with external VPN services if configured within specific user-deployed containers. It is not directly used by core M3TAL components, but available for custom stacks.
    *   **Default Value:** `password`
    *   **Example Value:** `myVpnSecret`
    *   **Component(s) Used By:** User-deployed VPN containers.
```