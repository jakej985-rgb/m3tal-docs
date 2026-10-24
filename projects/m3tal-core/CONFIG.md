# Environment Variable Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive guide to configuring your M3TAL system through environment variables. All M3TAL environment variables are read from the primary configuration file, `/etc/m3tal/.env`. This file is sourced by both the `m3tal` CLI binary and all Docker Compose stacks via the `--env-file` option, ensuring a consistent configuration across your entire deployment.

You can manage these variables using the `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific changes.

---

## Quick Reference Table

| Name                  | Default Value           | Description                                                                                                   |
| :-------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------ |
| `API_PORT`            | `5050`                  | Internal port for the M3TAL API daemon.                                                                       |
| `DASHBOARD_PORT`      | `8082`                  | Internal port for the M3TAL Dashboard container.                                                              |
| `DASHBOARD_EXPOSE_MODE` | `local`                 | Controls M3TAL Dashboard exposure: `local` (direct port) or `traefik` (domain-based).                         |
| `HTTP_PORT`           | `8080`                  | General HTTP port for services.                                                                               |
| `STATE_DIR`           | `./state`               | Directory for M3TAL state database and dashboard configuration.                                               |
| `LOG_LEVEL`           | `info`                  | Logging verbosity for the M3TAL API daemon.                                                                   |
| `DASHBOARD_SECRET`    | `change_me_immediately` | Critical secret for M3TAL Dashboard session management. **Auto-generated.**                                   |
| `API_TOKEN`           | `change_me_api_token`   | Authentication token for the M3TAL API daemon. **Auto-generated.**                                            |
| `ADMIN_PASSWORD`      | `admin_pass`            | Initial password for the M3TAL Dashboard `admin` user.                                                        |
| `NETWORK_NAME`        | `m3tal`                 | Name of the Docker network for inter-container communication.                                                 |
| `LOCAL_IP`            | `127.0.0.1`             | Local IP address for host-bound services.                                                                     |
| `DOMAIN`              | `localhost`             | Primary domain for Traefik-routed services (e.g., `dash.DOMAIN`, `api.DOMAIN`).                               |
| `VPN_USER`            | `user`                  | Username for VPN client authentication.                                                                       |
| `VPN_PASSWORD`        | `password`              | Password for VPN client authentication.                                                                       |
| `BASE_STORAGE_PATH`   | `./data`                | Root directory for all M3TAL-managed media and data. **Defaults to `/mnt` in production.**                    |
| `MEDIA_PATH`          | `./data/media`          | Subdirectory for user media files.                                                                            |
| `CONFIG_PATH`         | `./data/config`         | Subdirectory for application configuration files.                                                             |
| `DOWNLOADS_PATH`      | `./data/downloads`      | Subdirectory for downloaded content.                                                                          |
| `PUID`                | `1000`                  | User ID for Docker containers (ensures correct host volume permissions).                                      |
| `PGID`                | `1000`                  | Group ID for Docker containers (ensures correct host volume permissions).                                     |
| `TZ`                  | `America/Denver`        | Timezone for Docker containers.                                                                               |
| `TRAEFIK_WEB_PORT`    | `80`                    | Host port Traefik binds for HTTP traffic.                                                                     |
| `TRAEFIK_WEBHTTPS_PORT` | `443`                   | Host port Traefik binds for HTTPS traffic.                                                                    |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                  | Internal port for the Traefik dashboard.                                                                      |
| `DEBUG_MODE`          | `false`                 | Enables debug logging and features.                                                                           |
| `METRICS_ENABLED`     | `true`                  | Controls whether Prometheus metrics endpoints are exposed by the API daemon.                                  |

---

## Detailed Variable Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system's operation, including internal ports and logging.

#### `API_PORT`
*   **Description**: The default internal port used by clients like the `m3tal-dashboard` to connect to the `m3tal-api` daemon via `host.docker.internal`.
    *   **Note**: The `m3tal-api` systemd service itself listens on port `8080` on the host, and Traefik routes to this host port. The `API_PORT` variable primarily affects internal Docker networking/dashboard connection string if `GO_API_URL` isn't fully specified.
*   **Default Value**: `5050`
*   **Example Value**: `5051`
*   **Used By**: `m3tal-dashboard` (via `GO_API_URL` default), potentially CLI.

#### `DASHBOARD_PORT`
*   **Description**: The internal port on which the `m3tal-dashboard` container listens. This port is also used for direct host port binding when `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **Default Value**: `8082`
*   **Example Value**: `8083`
*   **Used By**: `m3tal-dashboard` (internal, local mode port mapping, Traefik label).

#### `DASHBOARD_EXPOSE_MODE`
*   **Description**: Controls how the `m3tal-dashboard` container is exposed to the network.
    *   Set to `local` to bind the dashboard directly to a host port (`DASHBOARD_PORT`) for direct access (e.g., `http://HOST_IP:8082`). This is ideal for LAN-only setups.
    *   Set to `traefik` to configure Traefik to route traffic to the dashboard via `dash.DOMAIN`. This requires Traefik to be running.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Used By**: `m3tal` CLI (for `m3tal dash up` command to select appropriate compose override), `m3tal-dashboard` (via compose override selection).

#### `HTTP_PORT`
*   **Description**: A generic HTTP port variable, often used as a fallback or for specific internal services if not explicitly overridden by other configurations or Traefik.
*   **Default Value**: `8080`
*   **Example Value**: `8000`
*   **Used By**: General, no direct component linkage shown in context.

#### `STATE_DIR`
*   **Description**: Specifies the root directory where the `m3tal-api` daemon stores its SQLite state database (`state.db`) and where the `m3tal-dashboard` expects to find its configuration (e.g., `users.json`). For the dashboard, this directory is mapped to `/docker/state` inside the container.
*   **Default Value**: `./state`
*   **Example Value**: `/var/lib/m3tal/state`, `/opt/m3tal/stack/state`
*   **Used By**: `m3tal-api` daemon, `m3tal-dashboard` (volume mount).

#### `LOG_LEVEL`
*   **Description**: Sets the logging verbosity for the `m3tal-api` daemon and potentially other core components. Useful for debugging or reducing log noise.
*   **Default Value**: `info`
*   **Example Value**: `debug`
*   **Used By**: `m3tal-api` daemon.

#### `DEBUG_MODE`
*   **Description**: When set to `true`, enables verbose debug logging and potentially other debug-specific features across various M3TAL components.
*   **Default Value**: `false`
*   **Example Value**: `true`
*   **Used By**: `m3tal-api`, `m3tal-dashboard`, `m3tal` CLI.

#### `METRICS_ENABLED`
*   **Description**: Controls whether the `m3tal-api` daemon exposes a Prometheus-compatible metrics endpoint for monitoring purposes.
*   **Default Value**: `true`
*   **Example Value**: `false`
*   **Used By**: `m3tal-api`.

### Authentication & Security

These variables are critical for securing your M3TAL dashboard and API. Handle them with care.

#### `DASHBOARD_SECRET`
*   **Description**: A crucial secret key used by the `m3tal-dashboard` for session management, cookie signing, and other security-sensitive cryptographic operations. **This variable is automatically generated on first `m3tal init`. Users should NOT set it manually unless performing a careful rotation process.**
*   **Default Value**: `change_me_immediately`
*   **Example Value**: `a_very_long_and_complex_random_string_of_characters_for_security`
*   **Used By**: `m3tal-dashboard`, `m3tal` CLI (for generation).

#### `API_TOKEN`
*   **Description**: An authentication token (typically a JWT) used by the `m3tal` CLI and potentially other clients to authenticate with the `m3tal-api` daemon. **This variable is automatically generated on first `m3tal init`. Users should NOT set it manually unless performing a careful rotation process.**
*   **Default Value**: `change_me_api_token`
*   **Example Value**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE2NzgwODk2MDB9.EXAMPLE_JWT_TOKEN_WITH_AN_ACTUAL_SIGNATURE`
*   **Used By**: `m3tal` CLI, `m3tal-api` daemon.

#### `ADMIN_PASSWORD`
*   **Description**: The initial password for the default `admin` user account created for the `m3tal-dashboard`. It is highly recommended to change this immediately after first login using the `m3tal dashpass` command.
*   **Default Value**: `admin_pass`
*   **Example Value**: `MySuperSecureM3talAdminPassword!2023`
*   **Used By**: `m3tal-dashboard` (for initial `users.json` creation), `m3tal` CLI (for `dashpass` command).

### Network & Routing

Variables controlling network connectivity, Docker networking, and domain-based routing.

#### `NETWORK_NAME`
*   **Description**: The name of the Docker network (`proxy` in compose files) used for inter-container communication, particularly for services exposed via Traefik. All services that need to communicate with each other or be exposed by Traefik should join this network.
*   **Default Value**: `m3tal`
*   **Example Value**: `m3tal_internal_network`
*   **Used By**: All M3TAL Docker Compose stacks (e.g., `m3tal-dashboard`, `traefik`, user stacks).

#### `LOCAL_IP`
*   **Description**: Specifies a local IP address for host-bound services or for `host.docker.internal` resolution if a specific IP is required instead of the default `host-gateway`.
*   **Default Value**: `127.0.0.1`
*   **Example Value**: `192.168.1.100`
*   **Used By**: Potentially Docker containers using `host.docker.internal` for specific host interactions.

#### `DOMAIN`
*   **Description**: The primary domain name configured for services exposed via Traefik. Setting this variable enables Traefik to create routing rules such as `dash.DOMAIN` for the dashboard and `api.DOMAIN` for the API daemon. **This variable is crucial for enabling domain-based access via Traefik.**
*   **Default Value**: `localhost`
*   **Example Value**: `myhomeserver.arpa`, `m3tal.example.com`
*   **Used By**: `traefik`, `m3tal-dashboard` (Traefik labels in `traefik` expose mode), `m3tal` CLI (for dynamic Traefik configuration generation).

### Storage Paths

These variables define the filesystem locations for M3TAL's data, media, and configurations.

#### `BASE_STORAGE_PATH`
*   **Description**: The root directory where all M3TAL-managed media, configuration, and other data are stored on the host filesystem. This path is volume-mounted into relevant containers.
    *   **Note**: While the template defaults to `./data`, in production M3TAL deployments, this variable typically defaults to `/mnt` (e.g., for mounting an entire dedicated storage device) and **NOT** `./data`.
*   **Default Value**: `./data`
*   **Example Value**: `/mnt/m3tal_data`, `/opt/m3tal/data`
*   **Used By**: `m3tal-dashboard` (volume mount), other data-intensive containers.

#### `MEDIA_PATH`
*   **Description**: The subdirectory located under `BASE_STORAGE_PATH` specifically designated for user media files and content.
*   **Default Value**: `./data/media`
*   **Example Value**: `/mnt/media`, `${BASE_STORAGE_PATH}/media`
*   **Used By**: Any media-serving or media-processing containers.

#### `CONFIG_PATH`
*   **Description**: The subdirectory located under `BASE_STORAGE_PATH` used for storing application-specific configuration files (e.g., `users.json`).
*   **Default Value**: `./data/config`
*   **Example Value**: `/mnt/config`, `${BASE_STORAGE_PATH}/config/m3tal`
*   **Used By**: `m3tal-dashboard` (volume mount), other configuration-dependent containers.

#### `DOWNLOADS_PATH`
*   **Description**: The subdirectory located under `BASE_STORAGE_PATH` intended for storing downloaded content from various services.
*   **Default Value**: `./data/downloads`
*   **Example Value**: `/mnt/downloads`, `${BASE_STORAGE_PATH}/downloads`
*   **Used By**: Download client containers or services that manage content acquisition.

### Traefik Gateway

Variables specific to configuring the Traefik reverse proxy, which handles incoming HTTP/HTTPS traffic.

#### `TRAEFIK_WEB_PORT`
*   **Description**: The host port that Traefik binds to listen for incoming unencrypted HTTP traffic. This is the primary entry point for web services.
*   **Default Value**: `80`
*   **Example Value**: `8080` (if port 80 is occupied by another service)
*   **Used By**: `traefik` container.

#### `TRAEFIK_WEBHTTPS_PORT`
*   **Description**: The host port that Traefik binds to listen for incoming encrypted HTTPS traffic. This is typically used in conjunction with a certificate resolver.
*   **Default Value**: `443`
*   **Example Value**: `8443`
*   **Used By**: `traefik` container.

#### `TRAEFIK_DASHBOARD_PORT`
*   **Description**: The internal port on which the Traefik management dashboard (not the M3TAL dashboard) listens within its container. This is mapped to `127.0.0.1:8081` on the host for local access.
*   **Default Value**: `8080`
*   **Example Value**: `8080` (internal)
*   **Used By**: `traefik` container.

### VPN Configuration

Variables for setting up and configuring an optional VPN service within the M3TAL ecosystem.

#### `VPN_USER`
*   **Description**: The username to be used for VPN client authentication (e.g., for WireGuard or OpenVPN configurations).
*   **Default Value**: `user`
*   **Example Value**: `m3tal_client_user`
*   **Used By**: VPN service/container.

#### `VPN_PASSWORD`
*   **Description**: The password to be used in conjunction with `VPN_USER` for VPN client authentication.
*   **Default Value**: `password`
*   **Example Value**: `Sup3rSecur3VpNP@ssw0rd!`
*   **Used By**: VPN service/container.

### System & User Settings

General system-level settings, primarily for managing permissions within Docker containers.

#### `PUID`
*   **Description**: The User ID (UID) that Docker containers should run as. Setting this to match a user on your host machine (e.g., your own user ID) helps ensure correct file ownership and permissions when mapping host volumes into containers.
*   **Default Value**: `1000`
*   **Example Value**: `1001`
*   **Used By**: All user-facing Docker containers (e.g., `m3tal-dashboard`, user-deployed stacks).

#### `PGID`
*   **Description**: The Group ID (GID) that Docker containers should run as. Similar to `PUID`, this ensures correct group ownership and permissions for files on host volumes.
*   **Default Value**: `1000`
*   **Example Value**: `1001`
*   **Used By**: All user-facing Docker containers (e.g., `m3tal-dashboard`, user-deployed stacks).

#### `TZ`
*   **Description**: Sets the timezone for Docker containers. This ensures that logs, scheduled tasks, and timestamps within the containers reflect the correct local time. Use standard timezone formats (e.g., `America/New_York`).
*   **Default Value**: `America/Denver`
*   **Example Value**: `Europe/London`, `Asia/Tokyo`
*   **Used By**: All Docker containers (e.g., `m3tal-dashboard`, user-deployed stacks).