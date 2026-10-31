## M3TAL Environment Variables Reference

All M3TAL components, including the CLI (`/usr/bin/m3tal`), the API daemon (`m3tal-api.service`), and all Docker Compose stacks, read their configuration from a central `.env` file located at `/etc/m3tal/.env`. This file is paramount for M3TAL's operation and is managed via the `m3tal config wizard` or `m3tal config set` commands. Docker Compose stacks automatically load these variables via the `--env-file` option.

This document provides a comprehensive reference for all M3TAL environment variables.

---

### Quick Reference Table

| Name                   | Default Value           | Group                 | Components                                                                |
| :--------------------- | :---------------------- | :-------------------- | :------------------------------------------------------------------------ |
| `API_PORT`             | `5050`                  | Core                  | `m3tal-dashboard`                                                         |
| `LOG_LEVEL`            | `info`                  | Core                  | `m3tal-api.service`, CLI                                                  |
| `DEBUG_MODE`           | `false`                 | Core                  | CLI, `m3tal-api.service`, `m3tal-dashboard`                               |
| `METRICS_ENABLED`      | `true`                  | Core                  | `m3tal-api.service`                                                       |
| `STATE_DIR`            | `./state`               | Core                  | `m3tal-dashboard` (volume mount)                                          |
| `DASHBOARD_SECRET`     | `change_me_immediately` | Authentication        | `m3tal-dashboard`                                                         |
| `API_TOKEN`            | `change_me_api_token`   | Authentication        | `m3tal-api.service`, CLI                                                  |
| `ADMIN_PASSWORD`       | `admin_pass`            | Authentication        | `m3tal-dashboard`                                                         |
| `DASHBOARD_PORT`       | `8082`                  | Network               | `m3tal-dashboard`                                                         |
| `HTTP_PORT`            | `8080`                  | Network               | User stacks (generic HTTP)                                                |
| `NETWORK_NAME`         | `m3tal`                 | Network               | Docker networks                                                           |
| `LOCAL_IP`             | `127.0.0.1`             | Network               | User stacks, internal routing                                             |
| `BASE_STORAGE_PATH`    | `./data`                | Storage & Paths       | `m3tal-dashboard`, user stacks                                            |
| `MEDIA_PATH`           | `./data/media`          | Storage & Paths       | User stacks                                                               |
| `CONFIG_PATH`          | `./data/config`         | Storage & Paths       | `m3tal-dashboard`, user stacks                                            |
| `DOWNLOADS_PATH`       | `./data/downloads`      | Storage & Paths       | User stacks                                                               |
| `DOMAIN`               | `localhost`             | Traefik & Routing     | `traefik`, `m3tal-dashboard`                                              |
| `DASHBOARD_EXPOSE_MODE`| `local`                 | Traefik & Routing     | CLI (`m3tal dash up`), `m3tal-dashboard`                                  |
| `TRAEFIK_WEB_PORT`     | `80`                    | Traefik & Routing     | `traefik`                                                                 |
| `TRAEFIK_WEBHTTPS_PORT`| `443`                   | Traefik & Routing     | `traefik`                                                                 |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                  | Traefik & Routing     | `traefik`                                                                 |
| `VPN_USER`             | `user`                  | VPN                   | VPN stacks                                                                |
| `VPN_PASSWORD`         | `password`              | VPN                   | VPN stacks                                                                |
| `PUID`                 | `1000`                  | System & Runtime      | `m3tal-dashboard`, user stacks                                            |
| `PGID`                 | `1000`                  | System & Runtime      | `m3tal-dashboard`, user stacks                                            |
| `TZ`                   | `America/Denver`        | System & Runtime      | `m3tal-dashboard`, user stacks                                            |

---

### Detailed Environment Variable Reference

#### Core Configuration

These variables control fundamental M3TAL behaviors and internal communication.

*   **`API_PORT`**
    *   **Description:** The port M3TAL Dashboard uses to communicate with the M3TAL API daemon via `host.docker.internal`.
    *   **Default Value:** `5050`
    *   **Example Value:** `8080`
    *   **Component Usage:** `m3tal-dashboard` (specifically via its `GO_API_URL` environment variable).
    *   **Note:** The `m3tal-api.service` daemon itself runs on port `8080` on the host, and Traefik routes to `8080`. This `API_PORT` variable primarily affects the dashboard's internal connection.

*   **`LOG_LEVEL`**
    *   **Description:** The minimum logging level for the M3TAL API daemon and CLI.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`, `warn`, `error`, `fatal`
    *   **Component Usage:** CLI, `m3tal-api.service`.

*   **`DEBUG_MODE`**
    *   **Description:** Enables verbose debugging output and potentially additional debugging features in M3TAL components.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Component Usage:** CLI, `m3tal-api.service`, `m3tal-dashboard`.

*   **`METRICS_ENABLED`**
    *   **Description:** Controls whether performance metrics are collected and exposed by M3TAL components.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Component Usage:** `m3tal-api.service`.

*   **`STATE_DIR`**
    *   **Description:** Defines a base directory for M3TAL's internal state within containers.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal`
    *   **Component Usage:** `m3tal-dashboard` (sets its internal `STATE_DIR` env var to `/docker/state`, which is volume-mounted from `${CONFIG_PATH}/m3tal/state`).
    *   **Note:** The API daemon's primary SQLite database is fixed at `/var/lib/m3tal/state.db` and is *not* controlled by this environment variable.

#### Authentication

These variables manage access and security for M3TAL components.

*   **`DASHBOARD_SECRET`**
    *   **Description:** Secret key used by the M3TAL Dashboard for session management and other security-sensitive operations.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `aVeryLongAndRandomlyGeneratedSecretKey12345`
    *   **Component Usage:** `m3tal-dashboard`.
    *   **Note:** This variable is **auto-generated on first `m3tal init`** and users should **NOT set it manually unless rotating**.

*   **`API_TOKEN`**
    *   **Description:** Authorization token for accessing the M3TAL API daemon programmatically.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJpc3MiOiJtM3RhbCJ9.aSignatureGoesHere`
    *   **Component Usage:** `m3tal-api.service` (for authentication), CLI (for API calls).
    *   **Note:** This variable is **auto-generated on first `m3tal init`** and users should **NOT set it manually unless rotating**.

*   **`ADMIN_PASSWORD`**
    *   **Description:** The default password for the initial `admin` user in the M3TAL Dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `MySecureAdminPass123!`
    *   **Component Usage:** `m3tal-dashboard` (when initializing the `users.json` credential store). This password can be changed later using `m3tal dashpass`.

#### Network Configuration

Variables related to ports, network names, and host IP addresses.

*   **`DASHBOARD_PORT`**
    *   **Description:** The port on which the `m3tal-dashboard` container listens internally. In `local` expose mode, this port is directly bound to the host. In `traefik` mode, Traefik routes to this internal port.
    *   **Default Value:** `8082`
    *   **Example Value:** `80` (if using Traefik to route to the root domain)
    *   **Component Usage:** `m3tal-dashboard` (internal container port), `m3tal-compose.local.yml` (host port binding), `m3tal-compose.traefik.yml` (Traefik service port).

*   **`HTTP_PORT`**
    *   **Description:** A generic HTTP port that might be used by user-defined stacks.
    *   **Default Value:** `8080`
    *   **Example Value:** `80`
    *   **Component Usage:** Not explicitly used by core M3TAL components in the provided architecture, but available for custom service definitions.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the common Docker network used by M3TAL for inter-container communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal-prod-network`
    *   **Component Usage:** All Docker Compose stacks (e.g., `m3tal-compose.yml`, `routing-compose.yml`, and user-defined stacks) that connect to the shared `proxy` external network are influenced by the project name, which defaults to `m3tal` and often results in a network like `m3tal_proxy`.

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the M3TAL host.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Component Usage:** Currently not explicitly used by core M3TAL components for routing or bindings, but can be referenced by user-defined services or scripts. `host.docker.internal` is primarily used for host-gateway access.

#### Storage & Paths

These variables define the filesystem structure for M3TAL data.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory on the host filesystem where all M3TAL application data, media, and configurations are stored.
    *   **Default Value:** `./data` (for development/local setup)
    *   **Example Value:** `/srv/m3tal` or `/mnt/data`
    *   **Component Usage:** `m3tal-dashboard` (as a volume mount, e.g., `/mnt`), user-defined stacks.
    *   **Note:** For production deployments, this variable typically defaults to `/mnt`.

*   **`MEDIA_PATH`**
    *   **Description:** A subdirectory for user media files, typically located within `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/media`
    *   **Component Usage:** User-defined media services, `m3tal-dashboard` (if it processes media).

*   **`CONFIG_PATH`**
    *   **Description:** A subdirectory for application configuration files, typically located within `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Component Usage:** `m3tal-dashboard` (as a volume mount for `/docker/state/config`), user-defined stacks.

*   **`DOWNLOADS_PATH`**
    *   **Description:** A subdirectory for downloaded content, typically located within `BASE_STORAGE_PATH`.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/downloads`
    *   **Component Usage:** User-defined download services.

#### Traefik & Routing

Variables related to the Traefik reverse proxy and domain-based routing.

*   **`DOMAIN`**
    *   **Description:** The primary domain name for M3TAL services. Setting this variable enables Traefik routing rules for `dash.DOMAIN` and `api.DOMAIN`.
    *   **Default Value:** `localhost`
    *   **Example Value:** `mycoolserver.com`
    *   **Component Usage:** `traefik` (via dynamic config for `api.DOMAIN`), `m3tal-dashboard` (via Traefik labels for `dash.DOMAIN` when `DASHBOARD_EXPOSE_MODE=traefik`).

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is exposed.
        *   `local`: Binds directly to `DASHBOARD_PORT` on the host. Accessible via `http://HOST_IP:8082`. No Traefik required.
        *   `traefik`: Enables Traefik routing via `dash.DOMAIN`. Accessible via `http://dash.DOMAIN`. Traefik must be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Component Usage:** CLI (`m3tal dash up` selects the appropriate Docker Compose override file), `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port Traefik uses for its main HTTP entry point.
    *   **Default Value:** `80`
    *   **Example Value:** `8080` (if port 80 is already in use)
    *   **Component Usage:** `traefik` container in `routing-compose.yml`.

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port Traefik uses for its HTTPS entry point.
    *   **Default Value:** `443`
    *   **Example Value:** `8443`
    *   **Component Usage:** `traefik` container in `routing-compose.yml`.

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port Traefik exposes its own dashboard on. This is typically mapped to `127.0.0.1:8081` on the host for local access.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Component Usage:** `traefik` container in `routing-compose.yml`.

#### VPN Configuration

Variables for configuring a potential VPN service integrated with M3TAL.

*   **`VPN_USER`**
    *   **Description:** Username for the M3TAL VPN service (e.g., WireGuard).
    *   **Default Value:** `user`
    *   **Example Value:** `m3tal_vpn_user`
    *   **Component Usage:** VPN stacks (if deployed).

*   **`VPN_PASSWORD`**
    *   **Description:** Password for the M3TAL VPN service.
    *   **Default Value:** `password`
    *   **Example Value:** `StrongPass123!`
    *   **Component Usage:** VPN stacks (if deployed).

#### System & Runtime

General system-level variables for container runtime and localization.

*   **`PUID`**
    *   **Description:** The User ID (UID) that Docker containers should run as. This ensures correct file permissions for mounted volumes and often matches the host user's UID.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component Usage:** `m3tal-dashboard`, user-defined containers (via `user: "${PUID:-1000}:${PGID:-1000}"` in compose files).

*   **`PGID`**
    *   **Description:** The Group ID (GID) that Docker containers should run as. This ensures correct file permissions for mounted volumes and often matches the host user's GID.
    *   **Default Value:** `1000`
    *   **Example Value:** `1001`
    *   **Component Usage:** `m3tal-dashboard`, user-defined containers.

*   **`TZ`**
    *   **Description:** The timezone for M3TAL services, in IANA timezone database format (e.g., `America/New_York`, `Europe/London`).
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Component Usage:** `m3tal-dashboard`, user-defined containers.