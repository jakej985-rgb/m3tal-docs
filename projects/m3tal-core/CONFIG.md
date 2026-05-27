# M3TAL Environment Variables Reference

This document details all environment variables used by the M3TAL ecosystem. These variables are loaded from the `/etc/m3tal/.env` file by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` flag.

**Quick Reference Table**

| Variable Name          | Default Value       | Group     | Description                                                                 |
|------------------------|---------------------|-----------|-----------------------------------------------------------------------------|
| `DASHBOARD_PORT`       | `8082`              | Network   | Port for the M3TAL Dashboard.                                               |
| `DASHBOARD_EXPOSE_MODE`| `local`             | Network   | How the dashboard is exposed (local port or via Traefik).                   |
| `HTTP_PORT`            | `8080`              | Network   | Port for the M3TAL API daemon.                                              |
| `STATE_DIR`            | `./state`           | System    | Directory for the M3TAL state database.                                     |
| `LOG_LEVEL`            | `info`              | System    | Logging level for M3TAL services.                                           |
| `DASHBOARD_SECRET`     | `change_me_immediately`| Auth      | Secret for securing the M3TAL Dashboard. Auto-generated.                    |
| `API_TOKEN`            | `change_me_api_token` | Auth      | API token for authenticating with the M3TAL API. Auto-generated.            |
| `ADMIN_PASSWORD`       | `admin_pass`        | Auth      | Password for the M3TAL dashboard administrator account.                     |
| `NETWORK_NAME`         | `m3tal`             | Network   | Name of the Docker network M3TAL services will use.                         |
| `LOCAL_IP`             | `127.0.0.1`         | Network   | Local IP address used for internal communication.                           |
| `DOMAIN`               | `localhost`         | Traefik   | The primary domain for M3TAL services (e.g., `dash.DOMAIN`, `api.DOMAIN`). |
| `VPN_USER`             | `user`              | VPN       | Username for VPN connection.                                                |
| `VPN_PASSWORD`         | `password`          | VPN       | Password for VPN connection.                                                |
| `BASE_STORAGE_PATH`    | `./data`            | Storage   | Base path for all M3TAL data storage. Defaults to `/mnt` in production.     |
| `MEDIA_PATH`           | `./data/media`      | Storage   | Path for storing media files.                                               |
| `CONFIG_PATH`          | `./data/config`     | Storage   | Path for storing M3TAL configuration files.                                 |
| `DOWNLOADS_PATH`       | `./data/downloads`  | Storage   | Path for storing downloaded files.                                          |
| `PUID`                 | `1000`              | System    | User ID for running Docker containers.                                      |
| `PGID`                 | `1000`              | System    | Group ID for running Docker containers.                                     |
| `TZ`                   | `America/Denver`    | System    | Timezone for M3TAL services.                                                |
| `TRAEFIK_WEB_PORT`     | `80`                | Traefik   | Host port for Traefik's HTTP entrypoint.                                    |
| `TRAEFIK_WEBHTTPS_PORT`| `443`               | Traefik   | Host port for Traefik's HTTPS entrypoint.                                   |
| `TRAEFIK_DASHBOARD_PORT`| `8080`              | Traefik   | Internal port for Traefik's dashboard.                                      |
| `DEBUG_MODE`           | `false`             | System    | Enables debug logging and features.                                         |
| `METRICS_ENABLED`      | `true`              | System    | Enables metrics collection for M3TAL services.                              |

---

## Core

### `STATE_DIR`

*   **Description:** Specifies the directory where the M3TAL state database (`state.db`) will be stored. This is where M3TAL keeps track of its internal state and configurations.
*   **Default Value:** `./state`
*   **Example Value:** `/var/lib/m3tal/state`
*   **Used By:**
    *   **CLI binary**: When initializing or managing M3TAL services.
    *   **API daemon**: To locate and manage the state database.

### `LOG_LEVEL`

*   **Description:** Controls the verbosity of logging for M3TAL services. Supported levels typically include `debug`, `info`, `warn`, `error`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:**
    *   **CLI binary**: To set log levels for commands.
    *   **API daemon**: To configure its logging output.
    *   **Dashboard container**: To configure its logging output.

### `PUID`

*   **Description:** The User ID (UID) that Docker containers will run as. This is important for file permissions and ownership within the container and on the host system.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:**
    *   **CLI binary**: To configure container execution.
    *   **API daemon**: To start services with the specified user.
    *   **Dashboard container**: To set the user for the `m3tal-dashboard` service.

### `PGID`

*   **Description:** The Group ID (GID) that Docker containers will run as. Similar to `PUID`, this affects file permissions and ownership.
*   **Default Value:** `1000`
*   **Example Value:** `1001`
*   **Used By:**
    *   **CLI binary**: To configure container execution.
    *   **API daemon**: To start services with the specified group.
    *   **Dashboard container**: To set the group for the `m3tal-dashboard` service.

### `TZ`

*   **Description:** The timezone used by M3TAL services. This ensures correct time logging and scheduling.
*   **Default Value:** `America/Denver`
*   **Example Value:** `UTC`
*   **Used By:**
    *   **API daemon**: For time-related operations.
    *   **Dashboard container**: For time-related operations.

### `DEBUG_MODE`

*   **Description:** Enables debug-specific features and increased logging verbosity across M3TAL services.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:**
    *   **CLI binary**: To enable verbose output during operations.
    *   **API daemon**: To activate debug logging.
    *   **Dashboard container**: To activate debug logging.

### `METRICS_ENABLED`

*   **Description:** Controls whether M3TAL services expose metrics for monitoring.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:**
    *   **CLI binary**: To configure metric exposure.
    *   **API daemon**: To enable or disable metrics endpoint.

---

## Auth

### `DASHBOARD_SECRET`

*   **Description:** A secret key used to sign session cookies for the M3TAL Dashboard. **This variable is auto-generated on first `m3tal init`**. Users should **not** set this manually unless rotating the secret.
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_long_and_random_secret_string_generated_by_m3tal`
*   **Used By:**
    *   **CLI binary**: During initialization to set the secret.
    *   **Dashboard container**: For session security.

### `API_TOKEN`

*   **Description:** A token used for authenticating API requests to the M3TAL API daemon. **This variable is auto-generated on first `m3tal init`**. Users should **not** set this manually unless rotating the token.
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `a_super_secret_api_authentication_token_12345`
*   **Used By:**
    *   **CLI binary**: During initialization to set the token.
    *   **API daemon**: To validate incoming API requests.
    *   **Dashboard container**: To authenticate with the API daemon.

### `ADMIN_PASSWORD`

*   **Description:** The password for the default administrator account in the M3TAL Dashboard.
*   **Default Value:** `admin_pass`
*   **Example Value:** `a_strong_and_unique_password`
*   **Used By:**
    *   **CLI binary**: During initial setup or when setting the dashboard password.
    *   **Dashboard container**: For user authentication.

---

## Network

### `DASHBOARD_PORT`

*   **Description:** The port on the host machine where the M3TAL Dashboard is exposed. This port is directly accessible when `DASHBOARD_EXPOSE_MODE` is `local`.
*   **Default Value:** `8082`
*   **Example Value:** `8083`
*   **Used By:**
    *   **CLI binary**: To configure the dashboard container's port mapping.
    *   **Dashboard container**: To set its listening port.

### `HTTP_PORT`

*   **Description:** The port on the host machine where the M3TAL API daemon is exposed.
*   **Default Value:** `8080`
*   **Example Value:** `5050`
*   **Used By:**
    *   **CLI binary**: To configure the API daemon's access.
    *   **API daemon**: To set its listening port.

### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Determines how the M3TAL Dashboard is made accessible.
    *   `local`: Exposes the dashboard on `DASHBOARD_PORT` directly on the host. No reverse proxy is required. Access via `http://HOST_IP:DASHBOARD_PORT`.
    *   `traefik`: Configures the dashboard to be routed by Traefik. Access via `http://dash.DOMAIN`.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:**
    *   **CLI binary**: To apply the correct Docker Compose override for the dashboard.
    *   **Dashboard container**: While not directly used by the container itself, it influences its configuration via compose files.

### `NETWORK_NAME`

*   **Description:** The name of the Docker network that M3TAL services will be attached to. This facilitates communication between containers.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal_network`
*   **Used By:**
    *   **CLI binary**: To create or reference the Docker network.
    *   **API daemon**: To ensure services are connected to the correct network.
    *   **Traefik container**: To discover services on this network.

### `LOCAL_IP`

*   **Description:** The local IP address used for internal communication between M3TAL services, particularly for the dashboard communicating with the API.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `172.17.0.1` (common Docker gateway IP)
*   **Used By:**
    *   **API daemon**: For its internal service discovery.
    *   **Dashboard container**: For the `GO_API_URL` environment variable.

---

## Storage

### `BASE_STORAGE_PATH`

*   **Description:** The root directory where M3TAL stores all persistent data, including configuration, media, and downloads. **Note:** In production deployments, this defaults to `/mnt` rather than `./data` as seen in templates.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal_data`
*   **Used By:**
    *   **CLI binary**: To determine storage locations for various M3TAL components.
    *   **API daemon**: For volumes in its service definition.
    *   **Dashboard container**: For volumes in its service definition.

### `MEDIA_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal_data/media`
*   **Used By:**
    *   **CLI binary**: To reference media storage.
    *   **API daemon**: For media-related volumes.

### `CONFIG_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where M3TAL configuration files are stored.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal_data/config`
*   **Used By:**
    *   **CLI binary**: To reference configuration storage.
    *   **API daemon**: For configuration-related volumes.

### `DOWNLOADS_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal_data/downloads`
*   **Used By:**
    *   **CLI binary**: To reference download storage.
    *   **API daemon**: For download-related volumes.

---

## Traefik

### `DOMAIN`

*   **Description:** The base domain name used for routing services through Traefik. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes, allowing access to services via custom domain names instead of host IPs and ports.
*   **Default Value:** `localhost`
*   **Example Value:** `m3tal.example.com`
*   **Used By:**
    *   **CLI binary**: To configure Traefik routing rules.
    *   **Traefik container**: To define router rules (`Host()` matchers).
    *   **API daemon**: For its internal configuration which can be influenced by domain settings.

### `TRAEFIK_WEB_PORT`

*   **Description:** The host port that Traefik will listen on for HTTP traffic. This is the primary public-facing HTTP port when Traefik is used.
*   **Default Value:** `80`
*   **Example Value:** `80`
*   **Used By:**
    *   **Traefik container**: To expose its HTTP entrypoint.

### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The host port that Traefik will listen on for HTTPS traffic. This is the primary public-facing HTTPS port when Traefik is used.
*   **Default Value:** `443`
*   **Example Value:** `443`
*   **Used By:**
    *   **Traefik container**: To expose its HTTPS entrypoint.

### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The internal port on which the Traefik dashboard is accessible. Note that this is typically exposed via `127.0.0.1` only for local access to the Traefik dashboard itself.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:**
    *   **Traefik container**: For its internal dashboard service.

---

## VPN

### `VPN_USER`

*   **Description:** The username required to establish a VPN connection.
*   **Default Value:** `user`
*   **Example Value:** `myvpnuser`
*   **Used By:**
    *   **CLI binary**: Potentially for configuring VPN client setup.
    *   **API daemon**: If managing VPN connections directly.

### `VPN_PASSWORD`

*   **Description:** The password required to establish a VPN connection.
*   **Default Value:** `password`
*   **Example Value:** `mysecurevpnpassword`
*   **Used By:**
    *   **CLI binary**: Potentially for configuring VPN client setup.
    *   **API daemon**: If managing VPN connections directly.