# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are read from the `/etc/m3tal/.env` file by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` flag.

**Quick Reference Table**

| Variable Name             | Description                                                                                                                                                                                                       | Default Value      | Example Value                                | Used By                                                                                                                            |
| :------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------- | :------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| **Core**                  |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `HTTP_PORT`               | The HTTP port the M3TAL API daemon listens on.                                                                                                                                                                    | `8080`             | `8080`                                       | API daemon                                                                                                                         |
| `STATE_DIR`               | The directory where M3TAL stores its state, including the SQLite database.                                                                                                                                        | `./state`          | `/var/lib/m3tal/state`                       | API daemon, Dashboard                                                                                                              |
| `LOG_LEVEL`               | Sets the logging verbosity for M3TAL services.                                                                                                                                                                    | `info`             | `debug`                                      | API daemon, Dashboard                                                                                                              |
| `PUID`                    | The user ID to run containers as.                                                                                                                                                                                 | `1000`             | `1000`                                       | Dashboard, Ollama, Traefik, Cloudflared (via compose files)                                                                        |
| `PGID`                    | The group ID to run containers as.                                                                                                                                                                                | `1000`             | `1000`                                       | Dashboard, Ollama, Traefik, Cloudflared (via compose files)                                                                        |
| `TZ`                      | Timezone setting for containers.                                                                                                                                                                                  | `America/Denver`   | `Etc/UTC`                                    | Dashboard, Ollama, Traefik, Cloudflared (via compose files)                                                                        |
| `DEBUG_MODE`              | Enables debug mode for M3TAL services.                                                                                                                                                                            | `false`            | `true`                                       | General M3TAL services                                                                                                             |
| `METRICS_ENABLED`         | Enables or disables metrics collection for M3TAL services.                                                                                                                                                        | `true`             | `false`                                      | General M3TAL services                                                                                                             |
| **Auth**                  |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `DASHBOARD_SECRET`        | **Auto-generated on first `m3tal init`**. Secret key for the dashboard session. Should not be set manually unless rotating.                                                                                        | `change_me_immediately` | `a_highly_secret_and_random_string`        | Dashboard                                                                                                                          |
| `API_TOKEN`               | **Auto-generated on first `m3tal init`**. Token for API authentication. Should not be set manually unless rotating.                                                                                               | `change_me_api_token` | `your_secure_api_token_here`               | API daemon                                                                                                                         |
| `ADMIN_PASSWORD`          | Password for the default admin user of the M3TAL dashboard.                                                                                                                                                       | `admin_pass`       | `a_strong_password`                          | Dashboard                                                                                                                          |
| **Network**               |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `API_PORT`                | The port the M3TAL API daemon listens on. (Note: This appears to be a duplicate of `HTTP_PORT` in the provided JSON, but retained for clarity if intended as distinct.)                                                | `5050`             | `5050`                                       | API daemon                                                                                                                         |
| `DASHBOARD_PORT`          | The port the M3TAL dashboard service listens on internally.                                                                                                                                                       | `8082`             | `8082`                                       | Dashboard                                                                                                                          |
| `DASHBOARD_EXPOSE_MODE`   | Controls how the dashboard is exposed. `local` exposes it directly via `DASHBOARD_PORT`. `traefik` exposes it via Traefik using `dash.DOMAIN`.                                                                    | `local`            | `traefik`                                    | Dashboard, Traefik                                                                                                                 |
| `LOCAL_IP`                | The local IP address for network services.                                                                                                                                                                        | `127.0.0.1`        | `192.168.1.100`                              | General network configuration                                                                                                      |
| `NETWORK_NAME`            | The name of the Docker network M3TAL services will use.                                                                                                                                                           | `m3tal`            | `m3tal_network`                              | General Docker networking                                                                                                          |
| **Storage**               |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `BASE_STORAGE_PATH`       | Controls where media data is stored. Defaults to `/mnt` in production deployments, not `./data` as in the template.                                                                                              | `./data`           | `/mnt`                                       | Dashboard, Ollama, Traefik, Cloudflared (via compose files)                                                                        |
| `MEDIA_PATH`              | The specific path within `BASE_STORAGE_PATH` for media files.                                                                                                                                                     | `./data/media`     | `/mnt/media`                                 | Dashboard (via compose files)                                                                                                      |
| `CONFIG_PATH`             | The specific path within `BASE_STORAGE_PATH` for configuration files.                                                                                                                                             | `./data/config`    | `/mnt/config`                                | Dashboard (via compose files)                                                                                                      |
| `DOWNLOADS_PATH`          | The specific path within `BASE_STORAGE_PATH` for downloads.                                                                                                                                                       | `./data/downloads` | `/mnt/downloads`                             | Dashboard (via compose files)                                                                                                      |
| **Traefik**               |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `DOMAIN`                  | Controls Traefik routing rules. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes.                                                                                                                        | `localhost`        | `m3tal.example.com`                          | Traefik, Dashboard                                                                                                                 |
| `TRAEFIK_WEB_PORT`        | The host port Traefik uses for HTTP traffic.                                                                                                                                                                      | `80`               | `80`                                         | Traefik                                                                                                                            |
| `TRAEFIK_WEBHTTPS_PORT`   | The host port Traefik uses for HTTPS traffic.                                                                                                                                                                     | `443`              | `443`                                        | Traefik                                                                                                                            |
| `TRAEFIK_DASHBOARD_PORT`  | The port Traefik listens on for its own dashboard. This is usually exposed on `127.0.0.1`.                                                                                                                          | `8080`             | `8080`                                       | Traefik                                                                                                                            |
| **VPN**                   |                                                                                                                                                                                                                   |                    |                                              |                                                                                                                                    |
| `VPN_USER`                | Username for VPN connection.                                                                                                                                                                                      | `user`             | `myvpnuser`                                  | Cloudflared (if configured for VPN access)                                                                                         |
| `VPN_PASSWORD`            | Password for VPN connection.                                                                                                                                                                                      | `password`         | `mysecurevpnpassword`                        | Cloudflared (if configured for VPN access)                                                                                         |

---

## Detailed Environment Variable Breakdown

All environment variables are read from `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` and `m3tal config set` commands.

### Core

These variables define fundamental operational parameters for M3TAL.

*   **`HTTP_PORT`**
    *   **Description:** The HTTP port the M3TAL API daemon listens on.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** API daemon (`m3tal-api.service`)

*   **`STATE_DIR`**
    *   **Description:** The directory where M3TAL stores its state, including the SQLite database.
    *   **Default Value:** `./state`
    *   **Example Value:** `/var/lib/m3tal/state`
    *   **Used By:** API daemon, Dashboard container

*   **`LOG_LEVEL`**
    *   **Description:** Sets the logging verbosity for M3TAL services. Accepted values typically include `debug`, `info`, `warn`, `error`.
    *   **Default Value:** `info`
    *   **Example Value:** `debug`
    *   **Used By:** API daemon, Dashboard container

*   **`PUID`**
    *   **Description:** The user ID (UID) to run containers as. This is important for file permissions within the Docker environment.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Dashboard container, Ollama container, Traefik container, Cloudflared container (defined in their respective compose files).

*   **`PGID`**
    *   **Description:** The group ID (GID) to run containers as. This is important for file permissions within the Docker environment.
    *   **Default Value:** `1000`
    *   **Example Value:** `1000`
    *   **Used By:** Dashboard container, Ollama container, Traefik container, Cloudflared container (defined in their respective compose files).

*   **`TZ`**
    *   **Description:** Timezone setting for containers. This ensures consistent time logging and operations across services.
    *   **Default Value:** `America/Denver`
    *   **Example Value:** `Etc/UTC`
    *   **Used By:** Dashboard container, Ollama container, Traefik container, Cloudflared container (defined in their respective compose files).

*   **`DEBUG_MODE`**
    *   **Description:** Enables debug mode for M3TAL services, potentially providing more verbose logging or enabling specific debugging features.
    *   **Default Value:** `false`
    *   **Example Value:** `true`
    *   **Used By:** General M3TAL services.

*   **`METRICS_ENABLED`**
    *   **Description:** Enables or disables metrics collection for M3TAL services. When enabled, services may expose metrics endpoints for monitoring.
    *   **Default Value:** `true`
    *   **Example Value:** `false`
    *   **Used By:** General M3TAL services.

### Auth

These variables are crucial for securing M3TAL services.

*   **`DASHBOARD_SECRET`**
    *   **Description:** **Auto-generated on first `m3tal init`**. This is a secret key used for signing dashboard sessions. Users should **NOT** set this manually unless rotating the secret.
    *   **Default Value:** `change_me_immediately`
    *   **Example Value:** `a_highly_secret_and_random_string`
    *   **Used By:** Dashboard container

*   **`API_TOKEN`**
    *   **Description:** **Auto-generated on first `m3tal init`**. This token is used for authenticating API requests. Users should **NOT** set this manually unless rotating the token.
    *   **Default Value:** `change_me_api_token`
    *   **Example Value:** `your_secure_api_token_here`
    *   **Used By:** API daemon (for validating incoming requests if a token is provided)

*   **`ADMIN_PASSWORD`**
    *   **Description:** Sets the password for the default administrator user of the M3TAL dashboard.
    *   **Default Value:** `admin_pass`
    *   **Example Value:** `a_strong_password`
    *   **Used By:** Dashboard container

### Network

These variables configure network-related aspects of M3TAL.

*   **`API_PORT`**
    *   **Description:** The port the M3TAL API daemon listens on. (Note: This appears to be a duplicate of `HTTP_PORT` in the provided JSON, but retained for clarity if intended as distinct.)
    *   **Default Value:** `5050`
    *   **Example Value:** `5050`
    *   **Used By:** API daemon

*   **`DASHBOARD_PORT`**
    *   **Description:** The port the M3TAL dashboard service listens on internally. This is used for direct access in `local` mode or for Traefik to route to.
    *   **Default Value:** `8082`
    *   **Example Value:** `8082`
    *   **Used By:** Dashboard container

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description:** Controls how the dashboard is exposed.
        *   `local`: The dashboard is exposed directly via a port binding on the host (`DASHBOARD_PORT`). Access via `http://HOST_IP:DASHBOARD_PORT`. No Traefik is required for this mode.
        *   `traefik`: The dashboard is exposed via Traefik. Traefik will route traffic to `dash.DOMAIN`. This mode requires Traefik to be running.
    *   **Default Value:** `local`
    *   **Example Value:** `traefik`
    *   **Used By:** Dashboard container (via compose overrides), Traefik (for routing rules)

*   **`LOCAL_IP`**
    *   **Description:** The local IP address for network services. This can be used for services to bind to specific interfaces or for inter-service communication.
    *   **Default Value:** `127.0.0.1`
    *   **Example Value:** `192.168.1.100`
    *   **Used By:** General network configuration for services.

*   **`NETWORK_NAME`**
    *   **Description:** The name of the Docker network that M3TAL services will use for communication.
    *   **Default Value:** `m3tal`
    *   **Example Value:** `m3tal_network`
    *   **Used By:** General Docker networking configuration for M3TAL services.

### Storage

These variables define the locations for M3TAL's data and configuration.

*   **`BASE_STORAGE_PATH`**
    *   **Description:** Controls where media data is stored. **Defaults to `/mnt` in production deployments**, not `./data` as in the template. This is a crucial variable for production setups.
    *   **Default Value:** `./data`
    *   **Example Value:** `/mnt`
    *   **Used By:** Dashboard container, Ollama container, Traefik container, Cloudflared container (via compose files, defining persistent volumes).

*   **`MEDIA_PATH`**
    *   **Description:** The specific path within `BASE_STORAGE_PATH` designated for storing media files.
    *   **Default Value:** `./data/media`
    *   **Example Value:** `/mnt/media`
    *   **Used By:** Dashboard container (via compose files, mapping volumes).

*   **`CONFIG_PATH`**
    *   **Description:** The specific path within `BASE_STORAGE_PATH` designated for storing configuration files.
    *   **Default Value:** `./data/config`
    *   **Example Value:** `/mnt/config`
    *   **Used By:** Dashboard container (via compose files, mapping volumes).

*   **`DOWNLOADS_PATH`**
    *   **Description:** The specific path within `BASE_STORAGE_PATH` designated for storing downloaded files.
    *   **Default Value:** `./data/downloads`
    *   **Example Value:** `/mnt/downloads`
    *   **Used By:** Dashboard container (via compose files, mapping volumes).

### Traefik

These variables are used to configure the Traefik reverse proxy.

*   **`DOMAIN`**
    *   **Description:** Controls Traefik routing rules. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes, allowing access to services via custom domain names.
    *   **Default Value:** `localhost`
    *   **Example Value:** `m3tal.example.com`
    *   **Used By:** Traefik (for routing rules), Dashboard container (for routing labels).

*   **`TRAEFIK_WEB_PORT`**
    *   **Description:** The host port Traefik listens on for incoming HTTP traffic.
    *   **Default Value:** `80`
    *   **Example Value:** `80`
    *   **Used By:** Traefik container.

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description:** The host port Traefik listens on for incoming HTTPS traffic.
    *   **Default Value:** `443`
    *   **Example Value:** `443`
    *   **Used By:** Traefik container.

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description:** The port Traefik's own dashboard is exposed on. By default, it's bound to `127.0.0.1` for local access.
    *   **Default Value:** `8080`
    *   **Example Value:** `8080`
    *   **Used By:** Traefik container.

### VPN

These variables are used if you are configuring M3TAL to connect through a VPN.

*   **`VPN_USER`**
    *   **Description:** Username for establishing a VPN connection.
    *   **Default Value:** `user`
    *   **Example Value:** `myvpnuser`
    *   **Used By:** Cloudflared (if configured for VPN access), or other VPN-related services.

*   **`VPN_PASSWORD`**
    *   **Description:** Password for establishing a VPN connection.
    *   **Default Value:** `password`
    *   **Example Value:** `mysecurevpnpassword`
    *   **Used By:** Cloudflared (if configured for VPN access), or other VPN-related services.