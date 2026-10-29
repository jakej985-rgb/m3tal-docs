# Environment Variable Reference

All M3TAL environment variables are read from the primary configuration file located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` command. Both the `m3tal` CLI binary and all Docker Compose stacks utilize these variables, passed via the `--env-file /etc/m3tal/.env` flag, ensuring a unified and consistent configuration across the entire M3TAL ecosystem.

---

## Quick Reference

| Name                    | Default Value           | Description                                                                                             |
| :---------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------ |
| `API_PORT`              | `5050`                  | Port the dashboard expects the M3TAL API daemon to listen on.                                           |
| `DASHBOARD_PORT`        | `8082`                  | Port the M3TAL Dashboard container listens on internally.                                               |
| `DASHBOARD_EXPOSE_MODE` | `local`                 | Controls how the M3TAL Dashboard is exposed: `local` (direct port) or `traefik` (via domain).           |
| `HTTP_PORT`             | `8080`                  | Port the M3TAL API daemon listens on for incoming requests.                                             |
| `STATE_DIR`             | `./state`               | Internal path within the Dashboard container for state data.                                            |
| `LOG_LEVEL`             | `info`                  | Minimum logging level for the CLI and API daemon.                                                       |
| `DASHBOARD_SECRET`      | `change_me_immediately` | Secret key for securing dashboard sessions. **Auto-generated on `m3tal init`.**                         |
| `API_TOKEN`             | `change_me_api_token`   | Token for authenticating API daemon requests. **Auto-generated on `m3tal init`.**                       |
| `ADMIN_PASSWORD`        | `admin_pass`            | Default password for the initial dashboard admin user.                                                  |
| `NETWORK_NAME`          | `m3tal`                 | Default name for the internal Docker network used by M3TAL services.                                    |
| `LOCAL_IP`              | `127.0.0.1`             | Local IP address for internal network resolution.                                                       |
| `DOMAIN`                | `localhost`             | Primary domain for Traefik routing and service exposure.                                                |
| `VPN_USER`              | `user`                  | Username for a user-defined VPN service.                                                                |
| `VPN_PASSWORD`          | `password`              | Password for a user-defined VPN service.                                                                |
| `BASE_STORAGE_PATH`     | `./data`                | Base directory for all persistent M3TAL data volumes. **Defaults to `/mnt` in production.**             |
| `MEDIA_PATH`            | `./data/media`          | Path for media files, relative to `BASE_STORAGE_PATH`.                                                  |
| `CONFIG_PATH`           | `./data/config`         | Path for configuration files, relative to `BASE_STORAGE_PATH`.                                          |
| `DOWNLOADS_PATH`        | `./data/downloads`      | Path for downloaded content, relative to `BASE_STORAGE_PATH`.                                           |
| `PUID`                  | `1000`                  | User ID to use for running container processes.                                                         |
| `PGID`                  | `1000`                  | Group ID to use for running container processes.                                                        |
| `TZ`                    | `America/Denver`        | Timezone setting for all M3TAL containers.                                                              |
| `TRAEFIK_WEB_PORT`      | `80`                    | Port Traefik listens on for HTTP traffic.                                                               |
| `TRAEFIK_WEBHTTPS_PORT` | `443`                   | Port Traefik listens on for HTTPS traffic.                                                              |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                  | Internal port for Traefik's own dashboard.                                                              |
| `DEBUG_MODE`            | `false`                 | Enables verbose debug logging and features.                                                             |
| `METRICS_ENABLED`       | `true`                  | Enables collection and exposure of system metrics.                                                      |

---

## Detailed Reference

### Core Variables

These variables control fundamental aspects of M3TAL's operation, logging, and debugging.

#### `STATE_DIR`

*   **Description:** Specifies the internal path within the M3TAL Dashboard container where its state database and related files are stored. For the M3TAL API daemon, the state database is typically located at `/var/lib/m3tal/state.db` on the host. This variable primarily affects the dashboard's internal state management.
*   **Default Value:** `./state`
*   **Example Value:** `/docker/state`
*   **Used By:** M3TAL CLI (for internal operations and local testing), M3TAL Dashboard container (as the internal path for its state data).

#### `LOG_LEVEL`

*   **Description:** Sets the minimum logging severity level for the M3TAL CLI and API daemon. Valid values typically include `debug`, `info`, `warn`, `error`, `fatal`. Setting to `debug` provides the most verbose output.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:** M3TAL CLI, M3TAL API daemon.

#### `DEBUG_MODE`

*   **Description:** When set to `true`, this variable enables verbose logging, additional debug endpoints, and potentially development-specific features across M3TAL components. This is useful for troubleshooting.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:** M3TAL CLI, M3TAL API daemon, M3TAL Dashboard container.

#### `METRICS_ENABLED`

*   **Description:** Controls whether system and application metrics are collected and exposed by the M3TAL API daemon. Set to `false` to disable metrics reporting.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:** M3TAL CLI, M3TAL API daemon.

### Authentication Variables

These variables are crucial for securing access to the M3TAL API and Dashboard.

#### `DASHBOARD_SECRET`

*   **Description:** A critical secret key used by the M3TAL Dashboard for session management, cookie signing, and other cryptographic operations. **This value is automatically generated on the first `m3tal init` and should NOT be set manually unless you are intentionally rotating the key for security purposes.**
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_highly_random_long_string_of_characters_for_security`
*   **Used By:** M3TAL Dashboard container.

#### `API_TOKEN`

*   **Description:** An authentication token used to secure communications with the M3TAL API daemon. This token must be present in API requests for successful authentication. **This value is automatically generated on the first `m3tal init` and should NOT be set manually unless you are intentionally rotating the token for security purposes.**
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `another_super_long_random_token_string_for_api_auth`
*   **Used By:** M3TAL API daemon (for validating incoming requests), M3TAL CLI (for sending authenticated requests to the daemon).

#### `ADMIN_PASSWORD`

*   **Description:** Sets the initial password for the default administrator user of the M3TAL Dashboard. After initial setup, it can be updated via the `m3tal dashpass` command.
*   **Default Value:** `admin_pass`
*   **Example Value:** `MySecurePassword123`
*   **Used By:** M3TAL CLI (`m3tal dashpass` command), M3TAL Dashboard container (during initial user setup via `users.json`).

### Network Variables

These variables define how M3TAL components communicate and are exposed over the network.

#### `API_PORT`

*   **Description:** Specifies the port the M3TAL Dashboard expects the M3TAL API daemon to be listening on. This is the port the dashboard container uses to communicate with the host API daemon via `host.docker.internal`. While the M3TAL API daemon typically listens on port 8080, this variable allows the dashboard to connect to a different port if the API daemon's listening port is changed.
*   **Default Value:** `5050`
*   **Example Value:** `8080`
*   **Used By:** M3TAL Dashboard container (as part of its `GO_API_URL` environment variable).

#### `DASHBOARD_PORT`

*   **Description:** Defines the internal port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is set to `local`, this internal port is directly mapped to the host's `DASHBOARD_PORT`.
*   **Default Value:** `8082`
*   **Example Value:** `8080`
*   **Used By:** M3TAL Dashboard container, M3TAL CLI (`m3tal dash up` when in `local` mode).

#### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Controls how the M3TAL Dashboard is made accessible.
    *   `local`: Exposes the dashboard directly on `http://HOST_IP:DASHBOARD_PORT` via a direct port binding. This is best for LAN-only setups, first-time users, or local testing.
    *   `traefik`: Configures the dashboard to be routed via the Traefik gateway at `http://dash.DOMAIN`. This requires Traefik to be running via `m3tal up`. Best for domain-based setups.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:** M3TAL CLI (`m3tal dash up` command to select the appropriate Docker Compose override file).

#### `HTTP_PORT`

*   **Description:** The port on which the M3TAL API daemon (the Go binary service) listens for incoming HTTP requests from the host or other authorized clients. By default, the API daemon runs on 8080.
*   **Default Value:** `8080`
*   **Example Value:** `8081`
*   **Used By:** M3TAL API daemon.

#### `NETWORK_NAME`

*   **Description:** Specifies the name of the external Docker network that M3TAL services use for inter-service communication. This typically refers to the `proxy` network defined in `routing-compose.yml` and `m3tal-compose.yml`, which is named `m3tal_proxy` by default based on the project folder `m3tal` and the network `proxy`. If set directly as an external network, it might just be `m3tal`.
*   **Default Value:** `m3tal`
*   **Example Value:** `my_custom_m3tal_network`
*   **Used By:** All Docker Compose stacks (e.g., `m3tal-compose.yml`, `routing-compose.yml`, user-defined stacks) to connect services to a shared network.

#### `LOCAL_IP`

*   **Description:** An IP address used for internal network resolution within the M3TAL ecosystem, particularly for services needing to address the host machine from within Docker containers. This is less critical when `host.docker.internal:host-gateway` is utilized, but can be helpful in specific networking configurations.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `192.168.1.100`
*   **Used By:** M3TAL CLI, M3TAL API daemon (for internal network binding/resolution logic).

### Storage Variables

These variables define the filesystem paths where M3TAL stores its persistent data.

#### `BASE_STORAGE_PATH`

*   **Description:** The foundational directory on the host filesystem where all persistent M3TAL data (media, configuration files, downloads, etc.) is stored. Docker volumes are typically mounted relative to this path. **In production deployments, this path defaults to `/mnt`**. In local development or template environments, it might default to `./data`.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal_data`
*   **Used By:** M3TAL Dashboard container, all user-defined Docker Compose stacks (as the base for volume mounts).

#### `MEDIA_PATH`

*   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated for media content storage.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/media`
*   **Used By:** User-defined Docker Compose stacks (for mounting media libraries).

#### `CONFIG_PATH`

*   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated for M3TAL-related configuration files and state data. The dashboard uses this path to store its `state.db` and `users.json`.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/config`
*   **Used By:** M3TAL Dashboard container, user-defined Docker Compose stacks.

#### `DOWNLOADS_PATH`

*   **Description:** The subdirectory, relative to `BASE_STORAGE_PATH`, designated for downloaded files and temporary content.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/downloads`
*   **Used By:** User-defined Docker Compose stacks.

### Traefik Variables

These variables configure the Traefik reverse proxy gateway for service exposure.

#### `DOMAIN`

*   **Description:** The base domain name used for Traefik routing rules. Setting this enables specific routing for `dash.DOMAIN` (M3TAL Dashboard) and `api.DOMAIN` (M3TAL API daemon), allowing access to services via friendly URLs over HTTP (and HTTPS if configured).
*   **Default Value:** `localhost`
*   **Example Value:** `m3tal.example.com`
*   **Used By:** Traefik gateway (`routing-compose.yml` and dynamic configuration files like `dynamic/api.yml`), M3TAL Dashboard container (when `DASHBOARD_EXPOSE_MODE` is `traefik`), M3TAL CLI (for generating relevant URLs).

#### `TRAEFIK_WEB_PORT`

*   **Description:** The host port on which Traefik listens for incoming HTTP (non-SSL) traffic. This is the primary entry point for web services.
*   **Default Value:** `80`
*   **Example Value:** `8080`
*   **Used By:** Traefik gateway (`routing-compose.yml`).

#### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The host port on which Traefik listens for incoming HTTPS (SSL/TLS) traffic. This port should be used if you configure Traefik with SSL certificates.
*   **Default Value:** `443`
*   **Example Value:** `8443`
*   **Used By:** Traefik gateway (`routing-compose.yml`).

#### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The internal port for Traefik's own management dashboard, which is typically exposed locally on the host via `127.0.0.1:8081`. This variable refers to Traefik's *internal* listening port for its dashboard.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** Traefik gateway (`routing-compose.yml` for internal port mapping).

### VPN Variables

These variables are placeholders for configuring user-defined VPN services.

#### `VPN_USER`

*   **Description:** A username variable intended for use with user-defined VPN services or tunnel configurations that require authentication. (Note: The `cloudflared` container, while part of the routing stack, typically uses tokens, not username/password, for its tunnel functionality).
*   **Default Value:** `user`
*   **Example Value:** `m3talvpn`
*   **Used By:** User-defined VPN/tunnel Docker Compose stacks that might require a username.

#### `VPN_PASSWORD`

*   **Description:** A password variable intended for use with user-defined VPN services or tunnel configurations that require authentication.
*   **Default Value:** `password`
*   **Example Value:** `SecureVpnPass123`
*   **Used By:** User-defined VPN/tunnel Docker Compose stacks that might require a password.

### System Variables

These variables control system-level settings for Docker containers.

#### `PUID`

*   **Description:** The User ID (UID) that Docker containers will use to run their processes. This is crucial for ensuring correct file permissions on mounted volumes, preventing permission denied errors. It should typically match the UID of the user owning the `BASE_STORAGE_PATH` on the host system.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:** M3TAL Dashboard container, all user-defined Docker Compose stacks.

#### `PGID`

*   **Description:** The Group ID (GID) that Docker containers will use for their processes. Similar to `PUID`, this is vital for ensuring correct file permissions on mounted volumes. It should typically match the GID of the group owning the `BASE_STORAGE_PATH` on the host system.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:** M3TAL Dashboard container, all user-defined Docker Compose stacks.

#### `TZ`

*   **Description:** Sets the timezone inside M3TAL Docker containers. This ensures logs, timestamps, and scheduled tasks within containers are consistent with your local time. Use standard timezone names (e.g., `America/New_York`, `Europe/London`).
*   **Default Value:** `America/Denver`
*   **Example Value:** `Europe/Berlin`
*   **Used By:** M3TAL Dashboard container, all user-defined Docker Compose stacks.