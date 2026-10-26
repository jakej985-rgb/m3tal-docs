# M3TAL Environment Variables Reference

All M3TAL environment variables are managed via the `/etc/m3tal/.env` file. This file is read by both the M3TAL CLI and all Docker Compose stacks, ensuring a unified configuration experience. Variables are grouped logically below for clarity.

## Quick Reference

| Variable Name           | Description                                                                                                                                                                                                                                                                                                        | Default Value      |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------- |
| **Core**                |                                                                                                                                                                                                                                                                                                                    |                    |
| `STATE_DIR`             | Directory where the M3TAL state database and other core data are stored.                                                                                                                                                                                                                                           | `./state`          |
| `LOG_LEVEL`             | Sets the logging level for M3TAL services.                                                                                                                                                                                                                                                                         | `info`             |
| `DEBUG_MODE`            | Enables debug mode for M3TAL services.                                                                                                                                                                                                                                                                             | `false`            |
| `METRICS_ENABLED`       | Enables or disables metrics collection for M3TAL services.                                                                                                                                                                                                                                                         | `true`             |
| `PUID`                  | The User ID to run containers as.                                                                                                                                                                                                                                                                                  | `1000`             |
| `PGID`                  | The Group ID to run containers as.                                                                                                                                                                                                                                                                                 | `1000`             |
| `TZ`                    | The timezone to use for all services.                                                                                                                                                                                                                                                                              | `America/Denver`   |
| **Auth**                |                                                                                                                                                                                                                                                                                                                    |                    |
| `DASHBOARD_SECRET`      | A secret key used for securing the M3TAL dashboard. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.**                                                                                                                                                                                  | `change_me_immediately` |
| `API_TOKEN`             | An API token for authenticating with the M3TAL API. **Auto-generated on first `m3tal init`. Do not set manually unless rotating.**                                                                                                                                                                                  | `change_me_api_token` |
| `ADMIN_PASSWORD`        | The password for the M3TAL administrator account.                                                                                                                                                                                                                                                                  | `admin_pass`       |
| **Network**             |                                                                                                                                                                                                                                                                                                                    |                    |
| `API_PORT`              | The port on which the M3TAL API daemon listens.                                                                                                                                                                                                                                                                    | `5050`             |
| `DASHBOARD_PORT`        | The port on which the M3TAL dashboard listens internally.                                                                                                                                                                                                                                                          | `8082`             |
| `HTTP_PORT`             | The port used internally by services for HTTP communication.                                                                                                                                                                                                                                                         | `8080`             |
| `NETWORK_NAME`          | The name of the Docker network used by M3TAL services.                                                                                                                                                                                                                                                             | `m3tal`            |
| `LOCAL_IP`              | The local IP address to bind services to.                                                                                                                                                                                                                                                                          | `127.0.0.1`        |
| **Storage**             |                                                                                                                                                                                                                                                                                                                    |                    |
| `BASE_STORAGE_PATH`     | The base directory where M3TAL stores all its persistent data, including configuration and media. Defaults to `/mnt` in production deployments.                                                                                                                                                                     | `./data`           |
| `MEDIA_PATH`            | The subdirectory within `BASE_STORAGE_PATH` for storing media files.                                                                                                                                                                                                                                               | `./data/media`     |
| `CONFIG_PATH`           | The subdirectory within `BASE_STORAGE_PATH` for storing configuration files.                                                                                                                                                                                                                                         | `./data/config`    |
| `DOWNLOADS_PATH`        | The subdirectory within `BASE_STORAGE_PATH` for storing downloaded files.                                                                                                                                                                                                                                          | `./data/downloads` |
| **Traefik**             |                                                                                                                                                                                                                                                                                                                    |                    |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL dashboard is exposed. `local` exposes it via direct port mapping, `traefik` exposes it via Traefik routing.                                                                                                                                                                                  | `local`            |
| `DOMAIN`                | The domain name used for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes.                                                                                                                                                                                                              | `localhost`        |
| `TRAEFIK_WEB_PORT`      | The host port that Traefik uses for HTTP traffic.                                                                                                                                                                                                                                                                  | `80`               |
| `TRAEFIK_WEBHTTPS_PORT` | The host port that Traefik uses for HTTPS traffic.                                                                                                                                                                                                                                                                 | `443`              |
| `TRAEFIK_DASHBOARD_PORT`| The port Traefik uses to access its own dashboard.                                                                                                                                                                                                                                                                 | `8080`             |
| **VPN**                 |                                                                                                                                                                                                                                                                                                                    |                    |
| `VPN_USER`              | The username for VPN authentication.                                                                                                                                                                                                                                                                               | `user`             |
| `VPN_PASSWORD`          | The password for VPN authentication.                                                                                                                                                                                                                                                                               | `password`         |

---

## Core

These variables control the fundamental operation and data management of M3TAL.

### `STATE_DIR`

*   **Description:** The directory where the M3TAL state database and other core data are stored. This is where `state.db` will reside.
*   **Default Value:** `./state`
*   **Example Value:** `/var/lib/m3tal/state`
*   **Used By:** M3TAL API daemon, M3TAL CLI

### `LOG_LEVEL`

*   **Description:** Sets the logging level for M3TAL services. Accepted values typically include `debug`, `info`, `warn`, `error`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:** M3TAL API daemon, M3TAL Dashboard

### `DEBUG_MODE`

*   **Description:** Enables debug mode for M3TAL services, which often results in more verbose logging and potentially additional debugging features.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:** M3TAL API daemon, M3TAL Dashboard

### `METRICS_ENABLED`

*   **Description:** Enables or disables metrics collection for M3TAL services. When enabled, services may expose metrics endpoints for monitoring.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:** M3TAL API daemon

### `PUID`

*   **Description:** The User ID (UID) that Docker containers will run as. This is important for file permissions when containers write to host volumes.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:** M3TAL Dashboard container

### `PGID`

*   **Description:** The Group ID (GID) that Docker containers will run as. This is important for file permissions when containers write to host volumes.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:** M3TAL Dashboard container

### `TZ`

*   **Description:** The timezone to use for all services. This ensures consistent timestamping in logs and for any time-dependent operations.
*   **Default Value:** `America/Denver`
*   **Example Value:** `UTC`
*   **Used By:** M3TAL Dashboard container

## Auth

These variables are critical for securing your M3TAL instance.

### `DASHBOARD_SECRET`

*   **Description:** A secret key used for signing session cookies and securing communication within the M3TAL dashboard. **This variable is auto-generated on the first `m3tal init` and should not be set manually unless you are intentionally rotating it.**
*   **Default Value:** `change_me_immediately`
*   **Example Value:** `a_very_long_and_random_secret_key_generated_by_m3tal`
*   **Used By:** M3TAL Dashboard container

### `API_TOKEN`

*   **Description:** An API token used for authenticating requests to the M3TAL API. **This variable is auto-generated on the first `m3tal init` and should not be set manually unless you are intentionally rotating it.**
*   **Default Value:** `change_me_api_token`
*   **Example Value:** `a_secure_api_token_here_12345`
*   **Used By:** M3TAL API daemon (for internal communication and potential external access)

### `ADMIN_PASSWORD`

*   **Description:** The password for the default administrator account used to log into the M3TAL dashboard.
*   **Default Value:** `admin_pass`
*   **Example Value:** `my_strong_admin_password`
*   **Used By:** M3TAL Dashboard container

## Network

These variables configure network-related aspects of M3TAL services.

### `API_PORT`

*   **Description:** The port on which the M3TAL API daemon (Go binary) listens for incoming requests.
*   **Default Value:** `5050`
*   **Example Value:** `8080`
*   **Used By:** M3TAL API daemon, M3TAL Dashboard container (for `GO_API_URL`)

### `DASHBOARD_PORT`

*   **Description:** The port on which the M3TAL dashboard container listens internally. This is the port Traefik will be configured to route to, or the port directly exposed in `local` mode.
*   **Default Value:** `8082`
*   **Example Value:** `8082`
*   **Used By:** M3TAL Dashboard container, Traefik configuration

### `HTTP_PORT`

*   **Description:** A general-purpose HTTP port used internally by services for communication.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** M3TAL API daemon (for internal listening)

### `NETWORK_NAME`

*   **Description:** The name of the Docker network that M3TAL services will use for inter-container communication.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal_network`
*   **Used By:** M3TAL Docker Compose stacks

### `LOCAL_IP`

*   **Description:** The local IP address to which services should bind. `host.docker.internal` is often used for host-to-container communication, but this can define a specific IP for binding.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `127.0.0.1`
*   **Used By:** M3TAL API daemon, M3TAL Dashboard container (as `GO_API_URL`)

## Storage

These variables define the locations for M3TAL's persistent data.

### `BASE_STORAGE_PATH`

*   **Description:** The base directory where M3TAL stores all its persistent data, including configuration, media, and downloads. **In production deployments, this defaults to `/mnt`. In template/development setups, it may default to `./data`.**
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal_data`
*   **Used By:** M3TAL Docker Compose stacks (for volume mounts)

### `MEDIA_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal_data/media`
*   **Used By:** M3TAL Docker Compose stacks (for volume mounts)

### `CONFIG_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where configuration files are stored. This is typically where `/etc/m3tal/.env` would be managed and where persistent config for services might live.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal_data/config`
*   **Used By:** M3TAL Docker Compose stacks (for volume mounts)

### `DOWNLOADS_PATH`

*   **Description:** The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal_data/downloads`
*   **Used By:** M3TAL Docker Compose stacks (for volume mounts)

## Traefik

These variables control how M3TAL integrates with Traefik for routing and dashboard access.

### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Controls how the M3TAL dashboard is exposed to the network.
    *   `local`: The dashboard is exposed via a direct port mapping (`DASHBOARD_PORT`) on the host. Access via `http://HOST_IP:DASHBOARD_PORT`.
    *   `traefik`: The dashboard is exposed via Traefik routing. Access via `http://dash.DOMAIN`.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:** M3TAL Dashboard container (via compose overrides), Traefik configuration

### `DOMAIN`

*   **Description:** The domain name used for Traefik routing rules. Setting this variable enables routes such as `dash.DOMAIN` for the dashboard and `api.DOMAIN` for the API. If not set, Traefik may default to `localhost`.
*   **Default Value:** `localhost`
*   **Example Value:** `m3tal.example.com`
*   **Used By:** Traefik configuration (for router rules), M3TAL Dashboard container (via compose overrides)

### `TRAEFIK_WEB_PORT`

*   **Description:** The host port that Traefik uses as its HTTP entry point. This is the port that external HTTP traffic will hit on your host and be routed by Traefik.
*   **Default Value:** `80`
*   **Example Value:** `80`
*   **Used By:** Traefik container

### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The host port that Traefik uses as its HTTPS entry point. This is the port that external HTTPS traffic will hit on your host and be routed by Traefik.
*   **Default Value:** `443`
*   **Example Value:** `443`
*   **Used By:** Traefik container

### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The internal port on which Traefik itself is accessible for its administrative dashboard. This is typically mapped to `127.0.0.1:8081` on the host.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** Traefik container

## VPN

These variables are used for configuring VPN connections.

### `VPN_USER`

*   **Description:** The username used for authenticating with a VPN service.
*   **Default Value:** `user`
*   **Example Value:** `myvpnuser`
*   **Used By:** VPN client configuration (if a VPN client is integrated into the M3TAL stack)

### `VPN_PASSWORD`

*   **Description:** The password used for authenticating with a VPN service.
*   **Default Value:** `password`
*   **Example Value:** `mysecurevpnpassword`
*   **Used By:** VPN client configuration (if a VPN client is integrated into the M3TAL stack)