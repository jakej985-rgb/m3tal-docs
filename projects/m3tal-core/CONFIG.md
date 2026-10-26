# M3TAL Environment Variables Reference

All M3TAL environment variables are managed via the `/etc/m3tal/.env` file. This file is read by both the M3TAL CLI and all Docker Compose stacks, ensuring a unified configuration. You can manage these variables using `m3tal config wizard` or `m3tal config set KEY value`.

This document provides a comprehensive reference for all configurable environment variables.

## Quick Reference Table

| Variable Name             | Description                                                                                                                                | Default Value      |
|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| **Core**                  |                                                                                                                                            |                    |
| `API_PORT`                | The port the M3TAL API daemon listens on.                                                                                                  | `5050`             |
| `DASHBOARD_PORT`          | The port the M3TAL Dashboard container listens on.                                                                                         | `8082`             |
| `DASHBOARD_EXPOSE_MODE`   | Controls how the M3TAL Dashboard is exposed: `local` (direct port) or `traefik` (via Traefik).                                              | `local`            |
| `HTTP_PORT`               | The port the M3TAL API daemon listens on (aliased, generally `API_PORT` is preferred).                                                     | `8080`             |
| `STATE_DIR`               | The directory where the M3TAL state database (`state.db`) is stored.                                                                       | `./state`          |
| `LOG_LEVEL`               | The logging level for M3TAL services.                                                                                                      | `info`             |
| `PUID`                    | The user ID for running containers.                                                                                                        | `1000`             |
| `PGID`                    | The group ID for running containers.                                                                                                       | `1000`             |
| `TZ`                      | The timezone to use for M3TAL services.                                                                                                    | `America/Denver`   |
| `DEBUG_MODE`              | Enables or disables debug mode for M3TAL.                                                                                                  | `false`            |
| `METRICS_ENABLED`         | Enables or disables metrics collection for M3TAL.                                                                                          | `true`             |
| **Authentication**        |                                                                                                                                            |                    |
| `DASHBOARD_SECRET`        | **Auto-generated on first `m3tal init`. Rotate only if necessary.** Secret key used for securing the M3TAL Dashboard.                      | `change_me_immediately` |
| `API_TOKEN`               | **Auto-generated on first `m3tal init`. Rotate only if necessary.** Token used for authenticating with the M3TAL API.                     | `change_me_api_token` |
| `ADMIN_PASSWORD`          | The default password for the M3TAL administrator account.                                                                                  | `admin_pass`       |
| **Network**               |                                                                                                                                            |                    |
| `NETWORK_NAME`            | The name of the Docker network M3TAL services will use.                                                                                    | `m3tal`            |
| `LOCAL_IP`                | The local IP address of the host machine.                                                                                                  | `127.0.0.1`        |
| `DOMAIN`                  | The primary domain used for Traefik routing (e.g., `mydomain.com`). Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes.            | `localhost`        |
| **Storage**               |                                                                                                                                            |                    |
| `BASE_STORAGE_PATH`       | The base path for all M3TAL storage. **Defaults to `/mnt` in production, not `./data` as in the template.**                                | `./data`           |
| `MEDIA_PATH`              | The path within `BASE_STORAGE_PATH` for storing media files.                                                                               | `./data/media`     |
| `CONFIG_PATH`             | The path within `BASE_STORAGE_PATH` for storing configuration files.                                                                       | `./data/config`    |
| `DOWNLOADS_PATH`          | The path within `BASE_STORAGE_PATH` for storing downloads.                                                                                 | `./data/downloads` |
| **Traefik**               |                                                                                                                                            |                    |
| `TRAEFIK_WEB_PORT`        | The port Traefik listens on for HTTP traffic.                                                                                              | `80`               |
| `TRAEFIK_WEBHTTPS_PORT`   | The port Traefik listens on for HTTPS traffic.                                                                                             | `443`              |
| `TRAEFIK_DASHBOARD_PORT`  | The port Traefik exposes its own dashboard on (usually for internal access).                                                               | `8080`             |
| **VPN**                   |                                                                                                                                            |                    |
| `VPN_USER`                | The username for the VPN connection.                                                                                                       | `user`             |
| `VPN_PASSWORD`            | The password for the VPN connection.                                                                                                       | `password`         |

---

## Detailed Environment Variable Reference

All environment variables are read from `/etc/m3tal/.env` by both the M3TAL CLI and all Docker Compose stacks via `--env-file`.

### Core

These variables control fundamental M3TAL operations.

*   **`API_PORT`**
    *   **Description:** The port on which the M3TAL API daemon listens for incoming requests.
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** M3TAL API daemon, M3TAL CLI

*   **`DASHBOARD_PORT`**
    *   **Description:** The port on which the M3TAL Dashboard container listens internally. This is the port that Traefik or direct port mapping will connect to.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** M3TAL Dashboard, M3TAL CLI

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the M3TAL Dashboard is exposed to the network.
        *   `local`: Exposes the dashboard via a direct port mapping, accessible at `HOST_IP:DASHBOARD_PORT`. Ideal for LAN-only access or initial setup.
        *   `traefik`: Configures Traefik to route traffic for `dash.DOMAIN` to the dashboard. Requires Traefik to be running and `DOMAIN` to be set.
    *   **Default Value:** `local`
    *   **Example Value:** `local` or `traefik`
    *   **Used By:** M3TAL CLI, M3TAL Dashboard (via compose overrides)

*   **`HTTP_PORT`**
    *   **Description:** An alternative variable for the port the M3TAL API daemon listens on. `API_PORT` is generally preferred for clarity.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** M3TAL API daemon

*   **`STATE_DIR`**
    *   **Description:** The directory where the M3TAL state database (`state.db`) is stored.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** M3TAL API daemon, M3TAL CLI

*   **`LOG_LEVEL`**
    *   **Description:** Sets the logging verbosity for M3TAL services. Common values include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard

*   **`PUID`**
    *   **Description:** The user ID (UID) that Docker containers will run as. This is important for file permissions when M3TAL interacts with host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** M3TAL Dashboard, M3TAL CLI (for compose configurations)

*   **`PGID`**
    *   **Description:** The group ID (GID) that Docker containers will run as. This is important for file permissions when M3TAL interacts with host volumes.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** M3TAL Dashboard, M3TAL CLI (for compose configurations)

*   **`TZ`**
    *   **Description:** The timezone to be used by M3TAL services. This affects log timestamps and any time-based operations.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Europe/London`
    *   **Used By:** M3TAL Dashboard

*   **`DEBUG_MODE`**
    *   **Description:** Enables or disables debug mode for M3TAL. When set to `true`, M3TAL may log more verbose information or enable additional debugging features.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** M3TAL API daemon, M3TAL Dashboard

*   **`METRICS_ENABLED`**
    *   **Description:** Enables or disables the collection and exposition of M3TAL operational metrics.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** M3TAL API daemon

### Authentication

These variables are crucial for securing your M3TAL instance.

*   **`DASHBOARD_SECRET`**
    *   **Description:** A secret key used by the M3TAL Dashboard for session management and other security-related functions. **This variable is auto-generated on the first `m3tal init` command. You should only set or modify this manually if you need to rotate the secret.**
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_very_strong_and_random_secret_key`
    *   **Used By:** M3TAL Dashboard

*   **`API_TOKEN`**
    *   **Description:** A token used for authenticating API requests to the M3TAL API daemon. **This variable is auto-generated on the first `m3tal init` command. You should only set or modify this manually if you need to rotate the token.**
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `your_super_secret_api_token_here`
    *   **Used By:** M3TAL CLI (for authenticated commands), M3TAL Dashboard (for API communication if not using `host.docker.internal` implicitly)

*   **`ADMIN_PASSWORD`**
    *   **Description:** The default password for the initial administrator account created on the M3TAL Dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `mySecureAdminPassword123!`
    *   **Used By:** M3TAL Dashboard

### Network

These variables define network-related configurations.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will connect to. This allows services to communicate with each other by service name.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal-net`
    *   **Used By:** M3TAL CLI (for compose configurations), Docker Compose

*   **`LOCAL_IP`**
    *   **Description:** The local IP address of the host machine. This is often used by services to refer to the host from within a Docker container (e.g., `host.docker.internal`).
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** M3TAL API daemon (for `host.docker.internal` resolution), M3TAL CLI

*   **`DOMAIN`**
    *   **Description:** The primary domain name for your M3TAL instance. Setting this variable is crucial for Traefik routing. When `DOMAIN` is set, Traefik will automatically configure routes for `api.DOMAIN` and `dash.DOMAIN` (if `DASHBOARD_EXPOSE_MODE=traefik`).
    *   **Default Value:** `localhost`
    *   **Example Value:** `m3tal.mydomain.com`
    *   **Used By:** Traefik, M3TAL CLI (for compose configurations)

### Storage

These variables define the storage locations for M3TAL data.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** The root directory where M3TAL will store its persistent data, including configuration files, media, and downloads. **In production deployments, this defaults to `/mnt` to ensure data persistence outside of the container's ephemeral filesystem. In development or template setups, it might default to `./data`.**
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt/m3tal-storage`
    *   **Used By:** M3TAL CLI, M3TAL Dashboard

*   **`MEDIA_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` where media files (e.g., uploaded images, videos) are stored.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/m3tal-storage/media`
    *   **Used By:** M3TAL CLI, M3TAL Dashboard

*   **`CONFIG_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` where M3TAL configuration files (other than `.env`) are stored.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/m3tal-storage/config`
    *   **Used By:** M3TAL CLI, M3TAL Dashboard

*   **`DOWNLOADS_PATH`**
    *   **Description:** The sub-directory within `BASE_STORAGE_PATH` where downloaded files or data are stored.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/m3tal-storage/downloads`
    *   **Used By:** M3TAL CLI, M3TAL Dashboard

### Traefik

These variables configure the Traefik reverse proxy.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The port on the host machine that Traefik will use as its HTTP entry point. This is typically port 80.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The port on the host machine that Traefik will use as its HTTPS entry point. This is typically port 443.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The internal port on which Traefik's own administrative dashboard is exposed. This is usually mapped to `127.0.0.1:8081` on the host for secure, local access.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik container

### VPN

These variables are for configuring a VPN connection if M3TAL needs to operate within a VPN.

*   **`VPN_USER`**
    *   **Description:** The username for authenticating with a VPN server.
    *   **Default Value:** `user`
    *   **Example Value:** `myvpnuser`
    *   **Used By:** M3TAL CLI, M3TAL API daemon (if VPN functionality is utilized)

*   **`VPN_PASSWORD`**
    *   **Description:** The password for authenticating with a VPN server.
    *   **Default Value:** `password`
    *   **Example Value:** `mySuperSecretVpnPassword`
    *   **Used By:** M3TAL CLI, M3TAL API daemon (if VPN functionality is utilized)