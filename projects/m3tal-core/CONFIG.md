`docs/ENVIRONMENT_VARIABLES.md`

# Environment Variables Reference

Greetings, M3TAL operator. This document serves as your definitive guide to all environment variables configurable within the M3TAL ecosystem. These variables govern the behavior of the M3TAL CLI, the API daemon, and all associated Docker Compose stacks.

All M3TAL environment variables are centrally managed and read from `/etc/m3tal/.env`. This file is the primary configuration source for both the `/usr/bin/m3tal` CLI binary and all Docker Compose stacks, which load these variables via the `--env-file /etc/m3tal/.env` option.

You can manage these variables using the `m3tal config wizard` for an interactive setup experience, or directly with `m3tal config set KEY value` and `m3tal config get KEY`.

---

## Quick Reference

| Name                      | Default                       | Description                                                                                                                                                                                                                                                                                                                                                                                          |
|---------------------------|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `API_PORT`                | `5050`                        | Port the dashboard attempts to connect to the API daemon internally (via `host.docker.internal`). The actual host-bound port for the API daemon is `HTTP_PORT`.                                                                                                                                                                                                                       |
| `DASHBOARD_PORT`          | `8082`                        | Internal port for the M3TAL Dashboard container. Used for direct exposure in `local` mode or Traefik routing in `traefik` mode.                                                                                                                                                                                                                                                        |
| `DASHBOARD_EXPOSE_MODE`   | `local`                       | Controls how the M3TAL Dashboard is exposed: `local` (direct port binding) or `traefik` (via `dash.DOMAIN`).                                                                                                                                                                                                                                                                         |
| `HTTP_PORT`               | `8080`                        | The port on which the M3TAL API daemon (Go systemd service) listens on the host.                                                                                                                                                                                                                                                                                                     |
| `STATE_DIR`               | `./state`                     | (CLI-centric) Default base directory for state files. API daemon uses `/var/lib/m3tal/state.db`, dashboard mounts its state from `CONFIG_PATH`.                                                                                                                                                                                                                                    |
| `LOG_LEVEL`               | `info`                        | Sets the logging verbosity for M3TAL components (`debug`, `info`, `warn`, `error`).                                                                                                                                                                                                                                                                                                  |
| `DASHBOARD_SECRET`        | `change_me_immediately`       | Unique secret for dashboard session management. **Auto-generated on `m3tal init`. Do NOT set manually unless rotating.**                                                                                                                                                                                                                                                             |
| `API_TOKEN`               | `change_me_api_token`         | Authentication token for CLI-to-API daemon communication. **Auto-generated on `m3tal init`. Do NOT set manually unless rotating.**                                                                                                                                                                                                                                                   |
| `ADMIN_PASSWORD`          | `admin_pass`                  | Initial password for the default admin user, used when running `m3tal dashpass`.                                                                                                                                                                                                                                                                                                     |
| `NETWORK_NAME`            | `m3tal`                       | Name of the primary external Docker network. M3TAL's internal proxy network is explicitly named `proxy` in compose files.                                                                                                                                                                                                                                                            |
| `LOCAL_IP`                | `127.0.0.1`                   | Local IP address for internal network configurations, often superseded by `host.docker.internal`.                                                                                                                                                                                                                                                                              |
| `DOMAIN`                  | `localhost`                   | Base domain name for Traefik routing. Setting it enables `api.DOMAIN` and `dash.DOMAIN` routes.                                                                                                                                                                                                                                                                                         |
| `VPN_USER`                | `user`                        | Username for an optional VPN service (not used by `cloudflared` tunnel).                                                                                                                                                                                                                                                                                                             |
| `VPN_PASSWORD`            | `password`                    | Password for an optional VPN service.                                                                                                                                                                                                                                                                                                                                                |
| `BASE_STORAGE_PATH`       | `./data`                      | Root directory for all M3TAL's media and data. **In production deployments, this defaults to `/mnt`.**                                                                                                                                                                                                                                                                               |
| `MEDIA_PATH`              | `./data/media`                | Subdirectory within `BASE_STORAGE_PATH` for media files.                                                                                                                                                                                                                                                                                                                           |
| `CONFIG_PATH`             | `./data/config`               | Base directory for M3TAL configuration and dashboard state (`CONFIG_PATH/m3tal/state`).                                                                                                                                                                                                                                                                                            |
| `DOWNLOADS_PATH`          | `./data/downloads`            | Subdirectory within `BASE_STORAGE_PATH` for downloads.                                                                                                                                                                                                                                                                                                                             |
| `PUID`                    | `1000`                        | User ID (UID) for containers to ensure correct file permissions on mounted volumes.                                                                                                                                                                                                                                                                                                  |
| `PGID`                    | `1000`                        | Group ID (GID) for containers to ensure correct file permissions on mounted volumes.                                                                                                                                                                                                                                                                                                 |
| `TZ`                      | `America/Denver`              | Timezone for M3TAL containers (e.g., `Europe/London`).                                                                                                                                                                                                                                                                                                                             |
| `TRAEFIK_WEB_PORT`        | `80`                          | Host port Traefik listens on for HTTP traffic.                                                                                                                                                                                                                                                                                                                                       |
| `TRAEFIK_WEBHTTPS_PORT`   | `443`                         | Host port Traefik listens on for HTTPS traffic.                                                                                                                                                                                                                                                                                                                                      |
| `TRAEFIK_DASHBOARD_PORT`  | `8080`                        | Traefik's *internal* dashboard port. Host mapping is typically `127.0.0.1:8081`.                                                                                                                                                                                                                                                                                                    |
| `DEBUG_MODE`              | `false`                       | Enables debug functionalities across M3TAL components.                                                                                                                                                                                                                                                                                                                               |
| `METRICS_ENABLED`         | `true`                        | Controls whether components collect and expose metrics.                                                                                                                                                                                                                                                                                                                              |

---

## Detailed Variable Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system, including API communication, logging, and debugging.

#### `HTTP_PORT`
- **Description:** The port on which the M3TAL API daemon (a Go binary running as a systemd service) listens for incoming requests on the host machine. Traefik is configured to proxy `api.DOMAIN` to this port.
- **Default Value:** `8080`
- **Example Value:** `8080`
- **Used By:** M3TAL API daemon, Traefik (via `dynamic/api.yml`), CLI (to communicate with API).

#### `API_PORT`
- **Description:** This variable influences the internal `GO_API_URL` environment variable within the `m3tal-dashboard` container. While the M3TAL API daemon typically runs on `HTTP_PORT` (default `8080`) on the host, the dashboard is configured to attempt connection to `http://host.docker.internal:${API_PORT}`.
- **Default Value:** `5050`
- **Example Value:** `5050`
- **Used By:** m3tal-dashboard (via `GO_API_URL`).

#### `STATE_DIR`
- **Description:** (Primarily CLI-centric) Specifies the base directory for M3TAL's state and data files if not explicitly overridden by `CONFIG_PATH` or other specific paths. For the API daemon, the primary state database is located at `/var/lib/m3tal/state.db`. For the dashboard, its internal `STATE_DIR` environment variable is fixed at `/docker/state`, which is mounted from `${CONFIG_PATH}/m3tal/state`.
- **Default Value:** `./state`
- **Example Value:** `/var/lib/m3tal/state`
- **Used By:** CLI (as a default or for local operations).

#### `LOG_LEVEL`
- **Description:** Sets the logging verbosity for M3TAL components. Common values include `debug`, `info`, `warn`, `error`. Setting to `debug` provides more detailed output for troubleshooting.
- **Default Value:** `info`
- **Example Value:** `debug`
- **Used By:** CLI, M3TAL API daemon, m3tal-dashboard.

#### `DEBUG_MODE`
- **Description:** Enables or disables debug-specific functionalities, such as more detailed logging or extra API endpoints, across M3TAL components.
- **Default Value:** `false`
- **Example Value:** `true`
- **Used By:** CLI, M3TAL API daemon, m3tal-dashboard.

#### `METRICS_ENABLED`
- **Description:** Controls whether M3TAL components collect and expose operational metrics, useful for monitoring system health and performance.
- **Default Value:** `true`
- **Example Value:** `false`
- **Used By:** CLI, M3TAL API daemon, m3tal-dashboard.

### Authentication

These variables manage access and security for the M3TAL API and Dashboard.

#### `DASHBOARD_SECRET`
- **Description:** A unique secret key used by the `m3tal-dashboard` for secure session management and cookie signing. **This variable is automatically generated on first `m3tal init` and users should NOT set it manually unless rotating a compromised key.**
- **Default Value:** `change_me_immediately`
- **Example Value:** `aB3dE-fGhIjK-LmNoPq-RsTuVwX-yZ1234` (Auto-generated strong secret)
- **Used By:** m3tal-dashboard.

#### `API_TOKEN`
- **Description:** An authentication token used by the M3TAL CLI to authenticate with the M3TAL API daemon. This token secures communication between the CLI and the backend. **This variable is automatically generated on first `m3tal init` and users should NOT set it manually unless rotating a compromised token.**
- **Default Value:** `change_me_api_token`
- **Example Value:** `ABCDEFG1234567890HIJKLMNOP` (Auto-generated strong token)
- **Used By:** CLI, M3TAL API daemon (for token validation).

#### `ADMIN_PASSWORD`
- **Description:** The default password used to initialize the first admin user for the M3TAL Dashboard. After initialization via `m3tal dashpass`, user credentials (including this password, which can be changed) are stored in `/docker/users.json`.
- **Default Value:** `admin_pass`
- **Example Value:** `SecureP@ssw0rd!`
- **Used By:** CLI (`m3tal dashpass` for initial setup).

### Network Configuration

Variables related to how M3TAL components communicate over the network, including dashboard exposure and domain routing.

#### `DASHBOARD_PORT`
- **Description:** The internal port on which the `m3tal-dashboard` container listens.
    - When `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly exposed on the host, allowing access via `http://HOST_IP:${DASHBOARD_PORT}`.
    - When `DASHBOARD_EXPOSE_MODE` is `traefik`, Traefik routes traffic from `dash.DOMAIN` to this internal port.
- **Default Value:** `8082`
- **Example Value:** `8082`
- **Used By:** m3tal-dashboard, Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`).

#### `DASHBOARD_EXPOSE_MODE`
- **Description:** Controls how the M3TAL Dashboard is made accessible.
    - `local`: Uses the `m3tal-compose.local.yml` override to add a direct port binding (`${DASHBOARD_PORT:-8082}:8082`). Access via `http://HOST_IP:8082`. Ideal for LAN-only setups or initial testing.
    - `traefik`: Uses the `m3tal-compose.traefik.yml` override to add Traefik labels, routing `dash.${DOMAIN}` to the dashboard. Access via `http://dash.DOMAIN`. Requires Traefik to be running. Best for domain-based setups behind a reverse proxy.
- **Default Value:** `local`
- **Example Value:** `traefik`
- **Used By:** CLI (`m3tal dash up` to select compose overrides), Docker Compose (to apply specific network configurations).

#### `NETWORK_NAME`
- **Description:** The name of the primary external Docker network used by M3TAL for inter-container communication and proxying. M3TAL's internal proxy network is explicitly defined as `proxy` in its compose files; this variable provides a potential alternative or internal reference for other user-defined networks.
- **Default Value:** `m3tal`
- **Example Value:** `my_m3tal_network`
- **Used By:** Docker Compose (potentially for user-defined networks), Traefik.

#### `LOCAL_IP`
- **Description:** Specifies the local IP address for internal network configurations. While `host.docker.internal` is commonly used for container-to-host communication and is configured for the dashboard (`extra_hosts`), this variable can provide a specific IP if needed for custom setups or older Docker environments.
- **Default Value:** `127.0.0.1`
- **Example Value:** `192.168.1.100`
- **Used By:** (Potentially) Custom Docker network configurations.

#### `DOMAIN`
- **Description:** The base domain name used for Traefik routing rules. **Setting this variable enables and controls domain-based access patterns, such as `api.DOMAIN` (for the M3TAL API daemon) and `dash.DOMAIN` (for the M3TAL Dashboard when `DASHBOARD_EXPOSE_MODE=traefik`). If Traefik is enabled, this is a critical setting for external access.**
- **Default Value:** `localhost`
- **Example Value:** `myhome.arpa` or `example.com`
- **Used By:** Traefik (for generating routing rules), CLI (when configuring Traefik dynamic routes).

### Storage Configuration

These variables define the filesystem paths where M3TAL stores its data, media, and configurations.

#### `BASE_STORAGE_PATH`
- **Description:** The root directory for all M3TAL-managed media, application data, and user files. **It is crucial to note that while the template default is `./data`, in production M3TAL deployments, this variable typically defaults to `/mnt` to leverage dedicated storage.**
- **Default Value:** `./data`
- **Example Value:** `/mnt` or `/var/lib/m3tal/data`
- **Used By:** m3tal-dashboard (as a Docker volume mount point `:/mnt`), other user stacks (for persistent data storage).

#### `MEDIA_PATH`
- **Description:** The subdirectory within `BASE_STORAGE_PATH` designated for user-uploaded media files or other media-related data.
- **Default Value:** `./data/media`
- **Example Value:** `/mnt/media`
- **Used By:** User stacks (mounted volumes), potential future m3tal-dashboard features.

#### `CONFIG_PATH`
- **Description:** The base directory for M3TAL's general configuration files, including runtime state files for the dashboard. The dashboard mounts `${CONFIG_PATH}/m3tal/state` to its internal `/docker/state` path. This path should be persisted.
- **Default Value:** `./data/config`
- **Example Value:** `/etc/m3tal/config` or `/mnt/config`
- **Used By:** m3tal-dashboard (for state volume mounts), CLI (for managing `/etc/m3tal/.env` and other configuration).

#### `DOWNLOADS_PATH`
- **Description:** The subdirectory within `BASE_STORAGE_PATH` specifically for managing application-related downloads or user download directories.
- **Default Value:** `./data/downloads`
- **Example Value:** `/mnt/downloads`
- **Used By:** User stacks (mounted volumes).

### Traefik Gateway Configuration

Variables specific to the Traefik reverse proxy, controlling its exposed ports.

#### `TRAEFIK_WEB_PORT`
- **Description:** The host port on which the Traefik gateway container listens for standard HTTP (unencrypted web) traffic. This is the entry point for `http://DOMAIN` requests.
- **Default Value:** `80`
- **Example Value:** `80`
- **Used By:** Traefik gateway container.

#### `TRAEFIK_WEBHTTPS_PORT`
- **Description:** The host port on which the Traefik gateway container listens for HTTPS (encrypted websecure) traffic. Utilizing this requires additional Traefik configuration for SSL/TLS certificates (e.g., Let's Encrypt).
- **Default Value:** `443`
- **Example Value:** `443`
- **Used By:** Traefik gateway container.

#### `TRAEFIK_DASHBOARD_PORT`
- **Description:** The *internal* port on which the Traefik dashboard listens within its container. The host-bound port for the Traefik dashboard is typically hardcoded to `8081` in the `routing-compose.yml` (`127.0.0.1:8081:8080`), ensuring it's only accessible locally on the host. This variable refers to the container's internal port.
- **Default Value:** `8080`
- **Example Value:** `8080`
- **Used By:** Traefik gateway container (internal dashboard exposure).

### VPN Configuration

These variables are placeholders for potential integration with VPN services. Note that Cloudflared provides a tunnel, not a traditional VPN, and typically does not utilize these credentials.

#### `VPN_USER`
- **Description:** Specifies the username for configuring a VPN service, if a VPN container is integrated into the M3TAL ecosystem (e.g., OpenVPN or WireGuard clients).
- **Default Value:** `user`
- **Example Value:** `my_vpn_user`
- **Used By:** (Optional) Third-party VPN container configurations.

#### `VPN_PASSWORD`
- **Description:** Specifies the password for configuring a VPN service, if a VPN container is integrated into the M3TAL ecosystem.
- **Default Value:** `password`
- **Example Value:** `MyS3cur3Vpnp@ss`
- **Used By:** (Optional) Third-party VPN container configurations.

### System User & Localization

Variables controlling user/group IDs for container permissions and timezone settings.

#### `PUID`
- **Description:** The Process User ID (UID) that Docker containers should run as to ensure proper file permissions when interacting with mounted volumes on the host filesystem. This should ideally match the UID of the user who owns the `BASE_STORAGE_PATH` and other persistent data directories.
- **Default Value:** `1000`
- **Example Value:** `1001` (e.g., for a dedicated `m3tal` user)
- **Used By:** m3tal-dashboard, other user containers (to set container user).

#### `PGID`
- **Description:** The Process Group ID (GID) that Docker containers should run as to ensure proper file permissions when interacting with mounted volumes on the host filesystem. This should ideally match the GID of the group that owns the `BASE_STORAGE_PATH` and other persistent data directories.
- **Default Value:** `1000`
- **Example Value:** `1001` (e.g., for a dedicated `m3tal` group)
- **Used By:** m3tal-dashboard, other user containers (to set container group).

#### `TZ`
- **Description:** Sets the timezone for all M3TAL Docker containers. This ensures consistent timekeeping, correct timestamps in logs, and accurate scheduling across services. Value should be a [valid tz database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
- **Default Value:** `America/Denver`
- **Example Value:** `Europe/London` or `Asia/Tokyo`
- **Used By:** m3tal-dashboard, other user containers.