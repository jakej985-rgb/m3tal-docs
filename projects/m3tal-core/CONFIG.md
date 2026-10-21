# Environment Variables Reference

All M3TAL configuration, including environment variables, is centrally managed in the `/etc/m3tal/.env` file. This file is the single source of truth for both the M3TAL CLI and all Docker Compose stacks. The CLI reads it directly, and Docker Compose stacks are launched with `--env-file /etc/m3tal/.env` to ensure consistent configuration across the entire ecosystem.

You can manage these variables using the `m3tal config wizard` for an interactive experience or `m3tal config set <KEY> <VALUE>` for direct modifications.

---

## Quick Reference

| Name | Description | Default | Example | Components |
| :---------------------- | :------------------------------------------------------ | :--------------------------- | :--------------------------- | :--------------------------------------- |
| `API_PORT` | Port for Dashboard to connect to API daemon | `5050` | `5050` | M3TAL Dashboard |
| `DASHBOARD_PORT` | M3TAL Dashboard's internal/host port | `8082` | `8083` | M3TAL Dashboard, M3TAL CLI, Traefik |
| `DASHBOARD_EXPOSE_MODE` | How M3TAL Dashboard is exposed (`local` or `traefik`) | `local` | `traefik` | M3TAL CLI, M3TAL Dashboard |
| `HTTP_PORT` | Generic HTTP port (not explicitly used by M3TAL core) | `8080` | `8080` | (User-defined stacks) |
| `STATE_DIR` | Path to M3TAL API's SQLite database | `./state` | `/var/lib/m3tal/state` | M3TAL API daemon, M3TAL Dashboard |
| `LOG_LEVEL` | Logging verbosity for the API daemon | `info` | `debug` | M3TAL API daemon |
| `DASHBOARD_SECRET` | Cryptographic secret for M3TAL Dashboard | `change_me_immediately` | `r4nd0m_s3cr3t_v4lu3` | M3TAL Dashboard |
| `API_TOKEN` | Authentication token for M3TAL API | `change_me_api_token` | `r4nd0m_t0k3n_v4lu3` | M3TAL CLI, M3TAL API daemon |
| `ADMIN_PASSWORD` | Initial password for the default dashboard admin user | `admin_pass` | `MySecureP@ssw0rd` | M3TAL Dashboard |
| `NETWORK_NAME` | Name of the primary Docker network | `m3tal` | `m3tal-network` | All Docker containers |
| `LOCAL_IP` | Host IP for inter-container communication (e.g., `host.docker.internal`) | `127.0.0.1` | `192.168.1.100` | M3TAL Dashboard, Traefik |
| `DOMAIN` | Primary domain for Traefik routing | `localhost` | `m3tal.example.com` | Traefik, M3TAL CLI, M3TAL Dashboard |
| `VPN_USER` | Username for VPN client configurations | `user` | `m3taluser` | VPN client containers |
| `VPN_PASSWORD` | Password for VPN client configurations | `password` | `MyS3cureVPNP@ss` | VPN client containers |
| `BASE_STORAGE_PATH` | Root directory for all M3TAL data storage | `./data` | `/mnt/m3tal_data` | All Docker containers |
| `MEDIA_PATH` | Subdirectory for media files | `./data/media` | `/mnt/m3tal_data/media` | User-defined media services |
| `CONFIG_PATH` | Subdirectory for configuration files | `./data/config` | `/mnt/m3tal_data/config` | M3TAL Dashboard, User-defined services |
| `DOWNLOADS_PATH` | Subdirectory for downloaded files | `./data/downloads` | `/mnt/m3tal_data/downloads` | User-defined download clients |
| `PUID` | User ID for container processes | `1000` | `1000` | All Docker containers |
| `PGID` | Group ID for container processes | `1000` | `1000` | All Docker containers |
| `TZ` | Timezone for containers | `America/Denver` | `America/New_York` | All Docker containers |
| `TRAEFIK_WEB_PORT` | Host port for Traefik's HTTP entrypoint | `80` | `8080` | Traefik |
| `TRAEFIK_WEBHTTPS_PORT` | Host port for Traefik's HTTPS entrypoint | `443` | `8443` | Traefik |
| `TRAEFIK_DASHBOARD_PORT` | Traefik's internal dashboard port | `8080` | `8080` | Traefik |
| `DEBUG_MODE` | Enable debug logging/features | `false` | `true` | M3TAL API daemon, M3TAL Dashboard |
| `METRICS_ENABLED` | Enable internal metrics collection | `true` | `false` | M3TAL API daemon |

---

## Detailed Reference

### Core Configuration

These variables control fundamental aspects of the M3TAL system, including logging, internal ports, and general system behavior.

#### `API_PORT`

*   **Description:** Specifies the internal port that the M3TAL Dashboard uses to communicate with the M3TAL API daemon. While the M3TAL API daemon systemd service runs on port `8080` (which Traefik routes to), this variable sets the `GO_API_URL` environment variable within the dashboard container, allowing it to connect to the API.
*   **Default Value:** `5050`
*   **Example Value:** `5050`
*   **Components:** M3TAL Dashboard (via `GO_API_URL`)

#### `HTTP_PORT`

*   **Description:** A generic HTTP port variable. This variable is not explicitly used by the core M3TAL components (API daemon, Dashboard, Traefik) for primary functionality but can be utilized by user-defined Docker Compose stacks for exposing custom HTTP services.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Components:** (Primarily user-defined Docker services)

#### `STATE_DIR`

*   **Description:** Defines the directory path where the M3TAL API daemon stores its SQLite state database (`state.db`). This path is mounted into the dashboard container as `/docker/state`. In a typical production setup, this path resolves to `/var/lib/m3tal/state`.
*   **Default Value:** `./state`
*   **Example Value:** `/var/lib/m3tal/state`
*   **Components:** M3TAL API daemon, M3TAL Dashboard

#### `LOG_LEVEL`

*   **Description:** Sets the verbosity level for logs generated by the M3TAL API daemon. Valid values typically include `debug`, `info`, `warn`, `error`, `fatal`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Components:** M3TAL API daemon

#### `PUID`

*   **Description:** Specifies the User ID (UID) that Docker container processes will run as. Setting this helps prevent file permission issues when containers interact with host-mounted volumes. It should ideally match a user ID on your host system (e.g., your primary user's UID, often `1000`).
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Components:** M3TAL Dashboard, all user-defined Docker containers

#### `PGID`

*   **Description:** Specifies the Group ID (GID) that Docker container processes will run as. Similar to `PUID`, this ensures correct file permissions for host-mounted volumes. It should ideally match a group ID on your host system (e.g., your primary group's GID, often `1000`).
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Components:** M3TAL Dashboard, all user-defined Docker containers

#### `TZ`

*   **Description:** Sets the timezone for Docker containers. This is crucial for accurate timestamps in logs and applications. Refer to the [TZ database name list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for valid values.
*   **Default Value:** `America/Denver`
*   **Example Value:** `America/New_York`
*   **Components:** M3TAL Dashboard, all user-defined Docker containers

#### `DEBUG_MODE`

*   **Description:** A boolean flag to enable debug logging and potentially other debug-specific features within M3TAL components. Setting this to `true` will provide more verbose output, useful for troubleshooting.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Components:** M3TAL API daemon, M3TAL Dashboard

#### `METRICS_ENABLED`

*   **Description:** Controls whether the M3TAL API daemon collects and exposes internal operational metrics. Set to `false` to disable metrics collection if not needed.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Components:** M3TAL API daemon

### Authentication

These variables are critical for securing access to the M3TAL API and Dashboard. Some are auto-generated for security.

#### `DASHBOARD_SECRET`

*   **Description:** A unique, randomly generated cryptographic secret used by the M3TAL Dashboard for session management, cookie signing, and other security operations.
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_very_long_and_random_string_of_characters_for_security`
*   **Components:** M3TAL Dashboard
*   **Note:** This variable is **auto-generated** on the first `m3tal init` command. Users should **NOT** set this manually unless performing a rotation of the secret or during a disaster recovery scenario.

#### `API_TOKEN`

*   **Description:** An authentication token required for the M3TAL CLI to interact securely with the M3TAL API daemon. It serves as a bearer token for API requests.
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `another_extremely_long_and_random_token_for_api_access_security`
*   **Components:** M3TAL CLI, M3TAL API daemon
*   **Note:** This variable is **auto-generated** on the first `m3tal init` command. Users should **NOT** set this manually unless performing a rotation of the token or during a disaster recovery scenario.

#### `ADMIN_PASSWORD`

*   **Description:** Sets the initial password for the default 'admin' user account in the M3TAL Dashboard upon its first startup. It is **highly recommended** to change this immediately after the initial setup for security reasons.
*   **Default Value:** `admin_pass`
*   **Example Value:** `MySuperSecureP@ssw0rd123`
*   **Components:** M3TAL Dashboard (initial setup)

### Network Configuration

These variables define how M3TAL components communicate with each other and are exposed on the network.

#### `NETWORK_NAME`

*   **Description:** Specifies the base name for the primary Docker network(s) created and utilized by the M3TAL ecosystem for inter-container communication. By default, M3TAL creates an external network named `m3tal_proxy` (or similar, depending on the stack prefix) that all other services connect to.
*   **Default Value:** `m3tal`
*   **Example Value:** `my-custom-m3tal-network`
*   **Components:** All Docker containers (implicitly defines external network names)

#### `LOCAL_IP`

*   **Description:** Represents the IP address of the host machine from within Docker containers. This is particularly useful for services to communicate back to the host system or to provide a stable IP for scenarios where `host.docker.internal` might not be universally supported or desired.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `192.168.1.100` (Your host's local network IP address)
*   **Components:** M3TAL Dashboard (via `extra_hosts` for `host.docker.internal`), Traefik (for routing to host-local services like the M3TAL API daemon)

### Storage Paths

These variables define the filesystem structure for M3TAL's data, ensuring persistence and organized storage on the host.

#### `BASE_STORAGE_PATH`

*   **Description:** The root directory on the host machine where all M3TAL-related persistent data (media, configurations, downloads, etc.) will be stored. Docker containers will mount subdirectories relative to this path.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal_data`
*   **Components:** All Docker containers (as a base for volume mounts)
*   **Note:** In production M3TAL deployments, this variable typically defaults to `/mnt` to leverage dedicated storage mounts, rather than `./data` (which is more common for local testing or development setups relative to the stack directory).

#### `MEDIA_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` specifically designated for media files (e.g., movies, TV shows, music). This path is typically mounted into media server containers (e.g., Plex, Jellyfin).
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal_data/media`
*   **Components:** User-defined media services

#### `CONFIG_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` used to store application-specific configuration files. The M3TAL Dashboard's `/docker/state/config` volume maps to this host path.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal_data/config`
*   **Components:** M3TAL Dashboard, user-defined Docker services

#### `DOWNLOADS_PATH`

*   **Description:** A subdirectory within `BASE_STORAGE_PATH` intended for downloaded files. This path is commonly mounted into download client containers (e.g., qBittorrent, Transmission).
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal_data/downloads`
*   **Components:** User-defined download clients

### Traefik & Routing

These variables configure how M3TAL services are exposed via the Traefik reverse proxy and how the Dashboard itself is accessed.

#### `DOMAIN`

*   **Description:** Sets the primary domain name for services exposed through Traefik. When this variable is configured and Traefik is running, it will automatically enable routing for `dash.${DOMAIN}` to the M3TAL Dashboard and `api.${DOMAIN}` to the M3TAL API daemon.
*   **Default Value:** `localhost`
*   **Example Value:** `m3tal.example.com`
*   **Components:** Traefik, M3TAL CLI (for configuration generation), M3TAL Dashboard (when `DASHBOARD_EXPOSE_MODE` is `traefik`)
*   **Note:** Setting this variable is essential for domain-based routing for M3TAL services.

#### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Determines how the M3TAL Dashboard container is exposed to the network.
    *   **`local` (default):** The dashboard is directly exposed on the host's `DASHBOARD_PORT`. Access is typically via `http://HOST_IP:8082` or `http://localhost:8082`. This mode does not require Traefik.
    *   **`traefik`:** The dashboard is exposed via Traefik, accessible at `http://dash.${DOMAIN}`. This mode requires Traefik to be running via `m3tal up` and adds Traefik labels to the dashboard container through compose overrides.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Components:** M3TAL CLI (`m3tal dash up` command), M3TAL Dashboard (via compose overrides)

#### `DASHBOARD_PORT`

*   **Description:** The internal port on which the M3TAL Dashboard container listens. If `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly bound to the host. If `DASHBOARD_EXPOSE_MODE` is `traefik`, Traefik routes to this internal container port.
*   **Default Value:** `8082`
*   **Example Value:** `8083`
*   **Components:** M3TAL Dashboard, M3TAL CLI (for `local` mode port binding), Traefik (for routing in `traefik` mode)

#### `TRAEFIK_WEB_PORT`

*   **Description:** The host port that Traefik binds to for handling incoming HTTP (non-SSL) requests. This is Traefik's primary entrypoint for HTTP traffic.
*   **Default Value:** `80`
*   **Example Value:** `8080`
*   **Components:** Traefik

#### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The host port that Traefik binds to for handling incoming HTTPS (SSL/TLS) requests. This is Traefik's primary entrypoint for HTTPS traffic, often paired with certificate management.
*   **Default Value:** `443`
*   **Example Value:** `8443`
*   **Components:** Traefik

#### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The internal port on which Traefik's own administrative dashboard listens. This port is typically mapped to `127.0.0.1:8081` on the host, allowing local access to monitor Traefik's status and configurations.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Components:** Traefik

### VPN Configuration

These variables are placeholders for configuring VPN client containers within the M3TAL ecosystem, allowing user-defined stacks to integrate with VPN services.

#### `VPN_USER`

*   **Description:** A generic username variable intended for use by user-defined VPN client containers (e.g., OpenVPN, WireGuard). This is commonly used to pass credentials into VPN configuration files or environment variables.
*   **Default Value:** `user`
*   **Example Value:** `m3taluser`
*   **Components:** User-defined VPN client containers

#### `VPN_PASSWORD`

*   **Description:** A generic password variable intended for use by user-defined VPN client containers. This is commonly used to pass credentials into VPN configuration files or environment variables.
*   **Default Value:** `password`
*   **Example Value:** `MyS3cureVPNP@ssphrase`
*   **Components:** User-defined VPN client containers
