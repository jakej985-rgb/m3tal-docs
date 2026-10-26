# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are primarily managed through the `/etc/m3tal/.env` file, which is automatically read by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option.

## Quick Reference Table

| Variable Name           | Description                                                                                                                                                                                                                                                                                          | Default Value      | Example Value                                    | Used By                                                                                                                            |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------- | :----------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| **Core**                |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `HTTP_PORT`             | The HTTP port the API daemon will listen on.                                                                                                                                                                                                                                                         | `8080`             | `8080`                                           | API daemon, Traefik                                                                                                                |
| `STATE_DIR`             | The directory where the M3TAL state database (`state.db`) will be stored.                                                                                                                                                                                                                            | `./state`          | `/mnt/config/m3tal/state`                        | API daemon, Dashboard container                                                                                                    |
| `LOG_LEVEL`             | Sets the logging verbosity for M3TAL services. Possible values include `debug`, `info`, `warn`, `error`.                                                                                                                                                                                           | `info`             | `debug`                                          | API daemon                                                                                                                         |
| `DASHBOARD_SECRET`      | **Auto-generated on first `m3tal init`**. A secret key used for securing the dashboard session. **Do not set manually unless rotating.**                                                                                                                                                                | `change_me_immediately` | `very_secret_key_for_dashboard_sessions`       | Dashboard container                                                                                                                |
| `API_TOKEN`             | **Auto-generated on first `m3tal init`**. A token used for authenticating API requests. **Do not set manually unless rotating.**                                                                                                                                                                       | `change_me_api_token` | `super_secure_api_authentication_token`          | API daemon (for internal use, e.g., dashboard to API communication)                                                                |
| `ADMIN_PASSWORD`        | The password for the default admin user of the M3TAL dashboard.                                                                                                                                                                                                                                      | `admin_pass`       | `my_strong_admin_password`                       | Dashboard container                                                                                                                |
| `PUID`                  | The User ID (UID) to run containers as.                                                                                                                                                                                                                                                              | `1000`             | `1000`                                           | Docker containers (Dashboard)                                                                                                      |
| `PGID`                  | The Group ID (GID) to run containers as.                                                                                                                                                                                                                                                             | `1000`             | `1000`                                           | Docker containers (Dashboard)                                                                                                      |
| `TZ`                    | The timezone to be used by M3TAL services.                                                                                                                                                                                                                                                           | `America/Denver`  | `Europe/London`                                  | Docker containers (Dashboard)                                                                                                      |
| `DEBUG_MODE`            | Enables or disables debug mode for M3TAL services.                                                                                                                                                                                                                                                   | `false`            | `true`                                           | API daemon, Dashboard container                                                                                                    |
| `METRICS_ENABLED`       | Enables or disables the collection and export of metrics for M3TAL services.                                                                                                                                                                                                                         | `true`             | `false`                                          | API daemon                                                                                                                         |
| **Auth**                |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `DASHBOARD_SECRET`      | **Auto-generated on first `m3tal init`**. A secret key used for securing the dashboard session. **Do not set manually unless rotating.**                                                                                                                                                                | `change_me_immediately` | `very_secret_key_for_dashboard_sessions`       | Dashboard container                                                                                                                |
| `API_TOKEN`             | **Auto-generated on first `m3tal init`**. A token used for authenticating API requests. **Do not set manually unless rotating.**                                                                                                                                                                       | `change_me_api_token` | `super_secure_api_authentication_token`          | API daemon (for internal use, e.g., dashboard to API communication)                                                                |
| `ADMIN_PASSWORD`        | The password for the default admin user of the M3TAL dashboard.                                                                                                                                                                                                                                      | `admin_pass`       | `my_strong_admin_password`                       | Dashboard container                                                                                                                |
| **Network**             |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `API_PORT`              | The port the M3TAL API daemon listens on. This is an internal port and typically not directly exposed to the user unless `DASHBOARD_EXPOSE_MODE` is `local`.                                                                                                                                         | `5050`             | `5050`                                           | Dashboard container (as `GO_API_URL`)                                                                                              |
| `DASHBOARD_PORT`        | The port the M3TAL dashboard container listens on. This can be accessed directly or via Traefik depending on `DASHBOARD_EXPOSE_MODE`.                                                                                                                                                               | `8082`             | `8082`                                           | Dashboard container, Traefik (via labels)                                                                                          |
| `NETWORK_NAME`          | The name of the Docker network M3TAL services will use.                                                                                                                                                                                                                                              | `m3tal`            | `m3tal_network`                                  | Docker Compose                                                                                                                     |
| `LOCAL_IP`              | The local IP address to use for host-specific configurations.                                                                                                                                                                                                                                        | `127.0.0.1`        | `127.0.0.1`                                      | Dashboard container (`host.docker.internal`)                                                                                       |
| `DOMAIN`                | The base domain for M3TAL services. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes via Traefik.                                                                                                                                                                                           | `localhost`        | `myhomelab.local`                                | Traefik (routing rules), Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)                                                |
| `VPN_USER`              | The username for the VPN connection.                                                                                                                                                                                                                                                                 | `user`             | `myvpnuser`                                      | VPN client (if configured)                                                                                                         |
| `VPN_PASSWORD`          | The password for the VPN connection.                                                                                                                                                                                                                                                                 | `password`         | `mysecurevpnpassword`                            | VPN client (if configured)                                                                                                         |
| **Storage**             |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `BASE_STORAGE_PATH`     | The base directory for all M3TAL data storage. **Defaults to `/mnt` in production deployments.**                                                                                                                                                                                                      | `./data`           | `/mnt`                                           | Docker containers (Dashboard, API daemon), CLI                                                                                     |
| `MEDIA_PATH`            | The subdirectory within `BASE_STORAGE_PATH` for media files.                                                                                                                                                                                                                                         | `./data/media`     | `/mnt/media`                                     | Docker containers (Dashboard, API daemon)                                                                                          |
| `CONFIG_PATH`           | The subdirectory within `BASE_STORAGE_PATH` for configuration files.                                                                                                                                                                                                                                 | `./data/config`    | `/mnt/config`                                    | Docker containers (Dashboard, API daemon)                                                                                          |
| `DOWNLOADS_PATH`        | The subdirectory within `BASE_STORAGE_PATH` for downloaded files.                                                                                                                                                                                                                                    | `./data/downloads` | `/mnt/downloads`                                 | Docker containers (Dashboard, API daemon)                                                                                          |
| **Traefik**             |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `TRAEFIK_WEB_PORT`      | The host port that Traefik will use for HTTP traffic.                                                                                                                                                                                                                                                | `80`               | `80`                                             | Traefik container                                                                                                                  |
| `TRAEFIK_WEBHTTPS_PORT` | The host port that Traefik will use for HTTPS traffic.                                                                                                                                                                                                                                               | `443`              | `443`                                            | Traefik container                                                                                                                  |
| `TRAEFIK_DASHBOARD_PORT`| The port Traefik itself listens on for its dashboard.                                                                                                                                                                                                                                                | `8080`             | `8080`                                           | Traefik container                                                                                                                  |
| `DOMAIN`                | The base domain for M3TAL services. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes via Traefik.                                                                                                                                                                                           | `localhost`        | `myhomelab.local`                                | Traefik (routing rules), Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)                                                |
| **Dashboard Access**    |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL dashboard is exposed. Options: `local` (direct port access) or `traefik` (via Traefik reverse proxy).                                                                                                                                                                        | `local`            | `traefik`                                        | Docker Compose (dashboard override), CLI                                                                                           |
| `DASHBOARD_PORT`        | The port the M3TAL dashboard container listens on. This can be accessed directly or via Traefik depending on `DASHBOARD_EXPOSE_MODE`.                                                                                                                                                               | `8082`             | `8082`                                           | Dashboard container, Traefik (via labels)                                                                                          |
| **System**              |                                                                                                                                                                                                                                                                                                      |                    |                                                  |                                                                                                                                    |
| `STATE_DIR`             | The directory where the M3TAL state database (`state.db`) will be stored.                                                                                                                                                                                                                            | `./state`          | `/mnt/config/m3tal/state`                        | API daemon, Dashboard container                                                                                                    |
| `LOG_LEVEL`             | Sets the logging verbosity for M3TAL services. Possible values include `debug`, `info`, `warn`, `error`.                                                                                                                                                                                           | `info`             | `debug`                                          | API daemon                                                                                                                         |
| `PUID`                  | The User ID (UID) to run containers as.                                                                                                                                                                                                                                                              | `1000`             | `1000`                                           | Docker containers (Dashboard)                                                                                                      |
| `PGID`                  | The Group ID (GID) to run containers as.                                                                                                                                                                                                                                                             | `1000`             | `1000`                                           | Docker containers (Dashboard)                                                                                                      |
| `TZ`                    | The timezone to be used by M3TAL services.                                                                                                                                                                                                                                                           | `America/Denver`  | `Europe/London`                                  | Docker containers (Dashboard)                                                                                                      |
| `DEBUG_MODE`            | Enables or disables debug mode for M3TAL services.                                                                                                                                                                                                                                                   | `false`            | `true`                                           | API daemon, Dashboard container                                                                                                    |
| `METRICS_ENABLED`       | Enables or disables the collection and export of metrics for M3TAL services.                                                                                                                                                                                                                         | `true`             | `false`                                          | API daemon                                                                                                                         |

---

## Detailed Environment Variable Reference

All M3TAL components read their configuration from environment variables, primarily sourced from the `/etc/m3tal/.env` file. This file is managed by the `m3tal config wizard` and can be updated using `m3tal config set KEY value`.

### Core

These variables control fundamental aspects of the M3TAL system's operation.

*   **`HTTP_PORT`**
    *   **Description**: The HTTP port the API daemon will listen on.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Used By**: API daemon, Traefik

*   **`STATE_DIR`**
    *   **Description**: The directory where the M3TAL state database (`state.db`) will be stored.
    *   **Default Value**: `./state`
    *   **Example Value**: `/mnt/config/m3tal/state`
    *   **Used By**: API daemon, Dashboard container

*   **`LOG_LEVEL`**
    *   **Description**: Sets the logging verbosity for M3TAL services. Possible values include `debug`, `info`, `warn`, `error`.
    *   **Default Value**: `info`
    *   **Example Value**: `debug`
    *   **Used By**: API daemon

*   **`DASHBOARD_SECRET`**
    *   **Description**: **Auto-generated on first `m3tal init`**. A secret key used for securing the dashboard session. Users should **NOT** set this manually unless rotating the secret.
    *   **Default Value**: `change_me_immediately`
    *   **Example Value**: `very_secret_key_for_dashboard_sessions`
    *   **Used By**: Dashboard container

*   **`API_TOKEN`**
    *   **Description**: **Auto-generated on first `m3tal init`**. A token used for authenticating API requests. Users should **NOT** set this manually unless rotating the token.
    *   **Default Value**: `change_me_api_token`
    *   **Example Value**: `super_secure_api_authentication_token`
    *   **Used By**: API daemon (for internal use, e.g., dashboard to API communication)

*   **`ADMIN_PASSWORD`**
    *   **Description**: The password for the default admin user of the M3TAL dashboard.
    *   **Default Value**: `admin_pass`
    *   **Example Value**: `my_strong_admin_password`
    *   **Used By**: Dashboard container

*   **`PUID`**
    *   **Description**: The User ID (UID) to run containers as. This is important for file permissions within Docker.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker containers (Dashboard)

*   **`PGID`**
    *   **Description**: The Group ID (GID) to run containers as. This is important for file permissions within Docker.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker containers (Dashboard)

*   **`TZ`**
    *   **Description**: The timezone to be used by M3TAL services. This ensures correct timestamps in logs and for time-sensitive operations.
    *   **Default Value**: `America/Denver`
    *   **Example Value**: `Europe/London`
    *   **Used By**: Docker containers (Dashboard)

*   **`DEBUG_MODE`**
    *   **Description**: Enables or disables debug mode for M3TAL services. When enabled, more verbose logging and debugging information may be available.
    *   **Default Value**: `false`
    *   **Example Value**: `true`
    *   **Used By**: API daemon, Dashboard container

*   **`METRICS_ENABLED`**
    *   **Description**: Enables or disables the collection and export of metrics for M3TAL services. These metrics can be used for monitoring and performance analysis.
    *   **Default Value**: `true`
    *   **Example Value**: `false`
    *   **Used By**: API daemon

### Auth

Authentication-related settings.

*   **`DASHBOARD_SECRET`**
    *   **Description**: **Auto-generated on first `m3tal init`**. A secret key used for securing the dashboard session. Users should **NOT** set this manually unless rotating the secret.
    *   **Default Value**: `change_me_immediately`
    *   **Example Value**: `very_secret_key_for_dashboard_sessions`
    *   **Used By**: Dashboard container

*   **`API_TOKEN`**
    *   **Description**: **Auto-generated on first `m3tal init`**. A token used for authenticating API requests. Users should **NOT** set this manually unless rotating the token.
    *   **Default Value**: `change_me_api_token`
    *   **Example Value**: `super_secure_api_authentication_token`
    *   **Used By**: API daemon (for internal use, e.g., dashboard to API communication)

*   **`ADMIN_PASSWORD`**
    *   **Description**: The password for the default admin user of the M3TAL dashboard.
    *   **Default Value**: `admin_pass`
    *   **Example Value**: `my_strong_admin_password`
    *   **Used By**: Dashboard container

### Network

Variables related to network configuration and service discovery.

*   **`API_PORT`**
    *   **Description**: The port the M3TAL API daemon listens on. This is an internal port and typically not directly exposed to the user unless `DASHBOARD_EXPOSE_MODE` is `local`.
    *   **Default Value**: `5050`
    *   **Example Value**: `5050`
    *   **Used By**: Dashboard container (as `GO_API_URL`)

*   **`DASHBOARD_PORT`**
    *   **Description**: The port the M3TAL dashboard container listens on. This can be accessed directly or via Traefik depending on `DASHBOARD_EXPOSE_MODE`.
    *   **Default Value**: `8082`
    *   **Example Value**: `8082`
    *   **Used By**: Dashboard container, Traefik (via labels)

*   **`NETWORK_NAME`**
    *   **Description**: The name of the Docker network M3TAL services will use for communication.
    *   **Default Value**: `m3tal`
    *   **Example Value**: `m3tal_network`
    *   **Used By**: Docker Compose

*   **`LOCAL_IP`**
    *   **Description**: The local IP address to use for host-specific configurations, particularly for enabling `host.docker.internal` communication.
    *   **Default Value**: `127.0.0.1`
    *   **Example Value**: `127.0.0.1`
    *   **Used By**: Dashboard container (`host.docker.internal`)

*   **`DOMAIN`**
    *   **Description**: The base domain for M3TAL services. Setting this variable enables Traefik routing rules for `api.DOMAIN` and `dash.DOMAIN`. If not set, Traefik will default to routing for `localhost`.
    *   **Default Value**: `localhost`
    *   **Example Value**: `myhomelab.local`
    *   **Used By**: Traefik (routing rules), Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)

*   **`VPN_USER`**
    *   **Description**: The username for the VPN connection, if a VPN client is configured and utilized by M3TAL.
    *   **Default Value**: `user`
    *   **Example Value**: `myvpnuser`
    *   **Used By**: VPN client (if configured)

*   **`VPN_PASSWORD`**
    *   **Description**: The password for the VPN connection, if a VPN client is configured and utilized by M3TAL.
    *   **Default Value**: `password`
    *   **Example Value**: `mysecurevpnpassword`
    *   **Used By**: VPN client (if configured)

### Storage

These variables define the locations for M3TAL's data storage.

*   **`BASE_STORAGE_PATH`**
    *   **Description**: The base directory for all M3TAL data storage. **Defaults to `/mnt` in production deployments**, rather than `./data` as seen in template configurations. This is where media, configuration, and downloads will be housed.
    *   **Default Value**: `./data`
    *   **Example Value**: `/mnt`
    *   **Used By**: Docker containers (Dashboard, API daemon), CLI

*   **`MEDIA_PATH`**
    *   **Description**: The subdirectory within `BASE_STORAGE_PATH` where media files are stored.
    *   **Default Value**: `./data/media`
    *   **Example Value**: `/mnt/media`
    *   **Used By**: Docker containers (Dashboard, API daemon)

*   **`CONFIG_PATH`**
    *   **Description**: The subdirectory within `BASE_STORAGE_PATH` where M3TAL configuration files are stored.
    *   **Default Value**: `./data/config`
    *   **Example Value**: `/mnt/config`
    *   **Used By**: Docker containers (Dashboard, API daemon)

*   **`DOWNLOADS_PATH`**
    *   **Description**: The subdirectory within `BASE_STORAGE_PATH` for downloaded files.
    *   **Default Value**: `./data/downloads`
    *   **Example Value**: `/mnt/downloads`
    *   **Used By**: Docker containers (Dashboard, API daemon)

### Traefik

Configuration variables specific to the Traefik reverse proxy.

*   **`TRAEFIK_WEB_PORT`**
    *   **Description**: The host port that Traefik will use for incoming HTTP traffic.
    *   **Default Value**: `80`
    *   **Example Value**: `80`
    *   **Used By**: Traefik container

*   **`TRAEFIK_WEBHTTPS_PORT`**
    *   **Description**: The host port that Traefik will use for incoming HTTPS traffic.
    *   **Default Value**: `443`
    *   **Example Value**: `443`
    *   **Used By**: Traefik container

*   **`TRAEFIK_DASHBOARD_PORT`**
    *   **Description**: The port Traefik itself listens on for its administrative dashboard. This is typically exposed on `127.0.0.1:8081`.
    *   **Default Value**: `8080`
    *   **Example Value**: `8080`
    *   **Used By**: Traefik container

*   **`DOMAIN`**
    *   **Description**: The base domain for M3TAL services. Setting this variable enables Traefik routing rules for `api.DOMAIN` and `dash.DOMAIN`. If not set, Traefik will default to routing for `localhost`.
    *   **Default Value**: `localhost`
    *   **Example Value**: `myhomelab.local`
    *   **Used By**: Traefik (routing rules), Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)

### Dashboard Access

These variables control how the M3TAL dashboard is made available.

*   **`DASHBOARD_EXPOSE_MODE`**
    *   **Description**: Controls how the M3TAL dashboard is exposed to the network.
        *   `local`: The dashboard is exposed directly via `DASHBOARD_PORT` (default `8082`) on the host machine. Access via `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik is required for this mode.
        *   `traefik`: The dashboard is exposed via the Traefik reverse proxy. Access via `http://dash.DOMAIN`. Requires Traefik to be running.
    *   **Default Value**: `local`
    *   **Example Value**: `traefik`
    *   **Used By**: Docker Compose (dashboard override), CLI

*   **`DASHBOARD_PORT`**
    *   **Description**: The port the M3TAL dashboard container listens on. This port is used directly when `DASHBOARD_EXPOSE_MODE` is `local`, and it's the target port for Traefik when `DASHBOARD_EXPOSE_MODE` is `traefik`.
    *   **Default Value**: `8082`
    *   **Example Value**: `8082`
    *   **Used By**: Dashboard container, Traefik (via labels)

### System

System-level and operational configurations.

*   **`STATE_DIR`**
    *   **Description**: The directory where the M3TAL state database (`state.db`) will be stored.
    *   **Default Value**: `./state`
    *   **Example Value**: `/mnt/config/m3tal/state`
    *   **Used By**: API daemon, Dashboard container

*   **`LOG_LEVEL`**
    *   **Description**: Sets the logging verbosity for M3TAL services. Possible values include `debug`, `info`, `warn`, `error`.
    *   **Default Value**: `info`
    *   **Example Value**: `debug`
    *   **Used By**: API daemon

*   **`PUID`**
    *   **Description**: The User ID (UID) to run containers as. This is important for file permissions within Docker.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker containers (Dashboard)

*   **`PGID`**
    *   **Description**: The Group ID (GID) to run containers as. This is important for file permissions within Docker.
    *   **Default Value**: `1000`
    *   **Example Value**: `1000`
    *   **Used By**: Docker containers (Dashboard)

*   **`TZ`**
    *   **Description**: The timezone to be used by M3TAL services. This ensures correct timestamps in logs and for time-sensitive operations.
    *   **Default Value**: `America/Denver`
    *   **Example Value**: `Europe/London`
    *   **Used By**: Docker containers (Dashboard)

*   **`DEBUG_MODE`**
    *   **Description**: Enables or disables debug mode for M3TAL services. When enabled, more verbose logging and debugging information may be available.
    *   **Default Value**: `false`
    *   **Example Value**: `true`
    *   **Used By**: API daemon, Dashboard container

*   **`METRICS_ENABLED`**
    *   **Description**: Enables or disables the collection and export of metrics for M3TAL services. These metrics can be used for monitoring and performance analysis.
    *   **Default Value**: `true`
    *   **Example Value**: `false`
    *   **Used By**: API daemon