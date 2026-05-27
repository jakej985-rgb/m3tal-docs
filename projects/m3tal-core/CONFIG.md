# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are primarily managed via the `/etc/m3tal/.env` file, which is read by both the M3TAL CLI and all Docker Compose stacks. The `m3tal config wizard` command is the recommended way to manage these settings.

## Quick Reference

| Variable Name             | Default Value       | Description                                                      |
|---------------------------|---------------------|------------------------------------------------------------------|
| **Core**                  |                     |                                                                  |
| `DASHBOARD_PORT`          | `8082`              | Port for the M3TAL dashboard.                                    |
| `HTTP_PORT`               | `8080`              | Port for the M3TAL API daemon.                                   |
| `STATE_DIR`               | `./state`           | Directory for M3TAL's state database.                            |
| `LOG_LEVEL`               | `info`              | Logging level for M3TAL services.                                |
| `DEBUG_MODE`              | `false`             | Enables debug mode for M3TAL services.                           |
| `METRICS_ENABLED`         | `true`              | Enables metrics collection for M3TAL services.                   |
| **Auth**                  |                     |                                                                  |
| `DASHBOARD_SECRET`        | `change_me_immediately` | Secret key for the M3TAL dashboard session. Auto-generated.      |
| `API_TOKEN`               | `change_me_api_token` | API token for external service authentication. Auto-generated.   |
| `ADMIN_PASSWORD`          | `admin_pass`        | Password for the M3TAL dashboard admin user.                     |
| **Network**               |                     |                                                                  |
| `NETWORK_NAME`            | `m3tal`             | Name of the Docker network M3TAL services will use.              |
| `LOCAL_IP`                | `127.0.0.1`         | Local IP address to bind services to.                            |
| `DOMAIN`                  | `localhost`         | Base domain name for Traefik routing.                            |
| **Storage**               |                     |                                                                  |
| `BASE_STORAGE_PATH`       | `./data`            | Base path for all M3TAL data storage. **Defaults to `/mnt` in production.** |
| `MEDIA_PATH`              | `./data/media`      | Path for media files.                                            |
| `CONFIG_PATH`             | `./data/config`     | Path for configuration files.                                    |
| `DOWNLOADS_PATH`          | `./data/downloads`  | Path for downloaded files.                                       |
| `PUID`                    | `1000`              | User ID for Docker container processes.                          |
| `PGID`                    | `1000`              | Group ID for Docker container processes.                         |
| **Traefik**               |                     |                                                                  |
| `TRAEFIK_WEB_PORT`        | `80`                | Port Traefik listens on for HTTP traffic.                        |
| `TRAEFIK_WEBHTTPS_PORT`   | `443`               | Port Traefik listens on for HTTPS traffic.                       |
| `TRAEFIK_DASHBOARD_PORT`  | `8080`              | Port Traefik uses internally for its dashboard.                  |
| **VPN**                   |                     |                                                                  |
| `VPN_USER`                | `user`              | Username for the VPN connection.                                 |
| `VPN_PASSWORD`            | `password`          | Password for the VPN connection.                                 |
| **System**                |                     |                                                                  |
| `DASHBOARD_EXPOSE_MODE`   | `local`             | Controls how the dashboard is exposed (`local` or `traefik`).    |
| `TZ`                      | `America/Denver`    | Timezone for M3TAL services.                                     |

---

## Core

These environment variables configure fundamental aspects of the M3TAL system.

| Name                  | Description                                                                                                 | Default Value       | Example Value       | Used By                                |
|-----------------------|-------------------------------------------------------------------------------------------------------------|---------------------|---------------------|----------------------------------------|
| `DASHBOARD_PORT`      | The port on which the M3TAL dashboard service will listen.                                                  | `8082`              | `8082`              | `m3tal-dashboard`                      |
| `HTTP_PORT`           | The port on which the M3TAL API daemon will listen.                                                         | `8080`              | `5050`              | `m3tal-api.service`                    |
| `STATE_DIR`           | The directory where M3TAL's state database (`state.db`) will be stored.                                     | `./state`           | `/var/lib/m3tal/state` | `m3tal-api.service`, `m3tal-dashboard` |
| `LOG_LEVEL`           | Controls the verbosity of logs generated by M3TAL services. Can be `debug`, `info`, `warn`, `error`.        | `info`              | `debug`             | `m3tal-api.service`, `m3tal-dashboard` |
| `DEBUG_MODE`          | Enables debug mode for M3TAL services, which may provide more verbose logging or enable additional features. | `false`             | `true`              | `m3tal-api.service`                    |
| `METRICS_ENABLED`     | Enables the collection and exposure of Prometheus metrics from M3TAL services.                              | `true`              | `false`             | `m3tal-api.service`                    |

## Auth

These variables are related to authentication and security within the M3TAL ecosystem.

**Note:** `DASHBOARD_SECRET` and `API_TOKEN` are automatically generated on the first `m3tal init`. Users should **not** set these manually unless they intend to rotate them.

| Name                  | Description                                                                                                                     | Default Value           | Example Value                               | Used By                                |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------------|-------------------------|---------------------------------------------|----------------------------------------|
| `DASHBOARD_SECRET`    | A secret key used for signing session cookies for the M3TAL dashboard. This is crucial for security.                          | `change_me_immediately` | `a_randomly_generated_secret_key_here`      | `m3tal-dashboard`                      |
| `API_TOKEN`           | A token used for authenticating API requests to the M3TAL API daemon. Useful for inter-service communication.                 | `change_me_api_token`   | `another_random_api_token_for_auth`         | `m3tal-api.service`                    |
| `ADMIN_PASSWORD`      | The password for the default `admin` user of the M3TAL dashboard.                                                               | `admin_pass`            | `my_secure_dashboard_password`              | `m3tal-dashboard`                      |

## Network

These variables define network configurations for M3TAL services.

| Name          | Description                                                                                                                                    | Default Value   | Example Value   | Used By                                  |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|-----------------|------------------------------------------|
| `NETWORK_NAME`| The name of the Docker network that M3TAL services will be connected to. This is used for service discovery.                                    | `m3tal`         | `m3tal_net`     | `m3tal-api.service`, `m3tal-dashboard`   |
| `LOCAL_IP`    | The local IP address that M3TAL services should bind to. `127.0.0.1` means they will only be accessible from the host machine.                     | `127.0.0.1`     | `192.168.1.100` | `m3tal-api.service`, `m3tal-dashboard`   |
| `DOMAIN`      | The base domain name used for Traefik routing. When set to a valid domain (e.g., `mydomain.com`), M3TAL will enable routes like `api.mydomain.com` and `dash.mydomain.com`. | `localhost`     | `example.com`   | `traefik`, `m3tal-dashboard` (via labels) |

## Storage

These variables control the location and organization of data stored by M3TAL.

**Note:** In production deployments, `BASE_STORAGE_PATH` defaults to `/mnt`. For development or template setups, it defaults to `./data`.

| Name                  | Description                                                                                                                                                                         | Default Value       | Example Value             | Used By                                |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|---------------------------|----------------------------------------|
| `BASE_STORAGE_PATH`   | The root directory where all M3TAL data will be stored. This includes configuration, media, downloads, and state if not overridden by `STATE_DIR`. **Defaults to `/mnt` in production.** | `./data`            | `/mnt`                    | `m3tal-api.service`, `m3tal-dashboard` |
| `MEDIA_PATH`          | The subdirectory within `BASE_STORAGE_PATH` where media files are stored.                                                                                                           | `./data/media`      | `/mnt/media`              | `m3tal-api.service`, `m3tal-dashboard` |
| `CONFIG_PATH`         | The subdirectory within `BASE_STORAGE_PATH` where configuration-related files are stored.                                                                                           | `./data/config`     | `/mnt/config`             | `m3tal-api.service`, `m3tal-dashboard` |
| `DOWNLOADS_PATH`      | The subdirectory within `BASE_STORAGE_PATH` where downloaded files are stored.                                                                                                      | `./data/downloads`  | `/mnt/downloads`          | `m3tal-api.service`, `m3tal-dashboard` |
| `PUID`                | The User ID (UID) that Docker containers should run as. This is important for file permissions on the host system.                                                                  | `1000`              | `1001`                    | `m3tal-dashboard`                      |
| `PGID`                | The Group ID (GID) that Docker containers should run as. This is important for file permissions on the host system.                                                                 | `1000`              | `1001`                    | `m3tal-dashboard`                      |

## Traefik

These variables configure the Traefik reverse proxy used for routing traffic to M3TAL services.

| Name                     | Description                                                                                                                              | Default Value | Example Value | Used By          |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|------------------|
| `TRAEFIK_WEB_PORT`       | The port on the host machine that Traefik will listen on for incoming HTTP (unencrypted) traffic.                                        | `80`          | `80`          | `traefik`        |
| `TRAEFIK_WEBHTTPS_PORT`  | The port on the host machine that Traefik will listen on for incoming HTTPS (encrypted) traffic.                                         | `443`         | `443`         | `traefik`        |
| `TRAEFIK_DASHBOARD_PORT` | The internal port within the Traefik container that the Traefik dashboard listens on. Accessible via `127.0.0.1:8081` by default.            | `8080`        | `8080`        | `traefik`        |

## VPN

These variables are used to configure VPN client connectivity within the M3TAL ecosystem.

| Name         | Description                                              | Default Value | Example Value | Used By |
|--------------|----------------------------------------------------------|---------------|---------------|---------|
| `VPN_USER`   | The username for connecting to the VPN service.          | `user`        | `vpnuser`     | N/A     |
| `VPN_PASSWORD`| The password for connecting to the VPN service.          | `password`    | `vpnsecret`   | N/A     |

## System

These variables control system-level behavior and operational modes of M3TAL.

| Name                  | Description                                                                                                                                                                                                                                                                                                                             | Default Value | Example Value | Used By                                |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|----------------------------------------|
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL dashboard is exposed to the network. <br/> - `local` (default): Exposes the dashboard via a direct port mapping (`DASHBOARD_PORT`). Suitable for LAN-only access or initial setup. Access via `http://HOST_IP:DASHBOARD_PORT`. <br/> - `traefik`: Exposes the dashboard via Traefik routing rules. Requires `DOMAIN` to be set. Access via `http://dash.DOMAIN`. | `local`       | `traefik`     | `m3tal-dashboard` (via compose overrides) |
| `TZ`                  | Specifies the timezone for M3TAL services. This ensures accurate timestamps in logs and for time-sensitive operations.                                                                                                                                                                                                                   | `America/Denver` | `UTC`         | `m3tal-api.service`, `m3tal-dashboard` |

---

**Note on Configuration:**

All environment variables listed above are read from the `/etc/m3tal/.env` file. This file is managed by the M3TAL CLI.

*   **Initialization:** Use `m3tal init` to create and populate this file with sensible defaults.
*   **Configuration Wizard:** Run `m3tal config wizard` to interactively set or modify variables.
*   **Direct Setting:** Use `m3tal config set KEY VALUE` to programmatically set individual variables.
*   **Manual Editing:** You can directly edit `/etc/m3tal/.env` using your preferred text editor, but this is generally not recommended as it bypasses validation and best practices.

When you run `m3tal up` or `m3tal dash up`, Docker Compose will read this `.env` file and inject the values as environment variables into the respective containers.