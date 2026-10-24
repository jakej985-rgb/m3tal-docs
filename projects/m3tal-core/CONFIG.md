# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are the primary mechanism for configuring your M3TAL installation.

All environment variables are read from the `/etc/m3tal/.env` file by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option.

## Quick Reference Table

| Category      | Variable Name           | Default Value      |
|---------------|-------------------------|--------------------|
| Core          | `API_PORT`              | `5050`             |
| Core          | `DASHBOARD_PORT`        | `8082`             |
| Core          | `DASHBOARD_EXPOSE_MODE` | `local`            |
| Core          | `HTTP_PORT`             | `8080`             |
| Core          | `STATE_DIR`             | `./state`          |
| Core          | `LOG_LEVEL`             | `info`             |
| Core          | `PUID`                  | `1000`             |
| Core          | `PGID`                  | `1000`             |
| Core          | `TZ`                    | `America/Denver`   |
| Core          | `DEBUG_MODE`            | `false`            |
| Core          | `METRICS_ENABLED`       | `true`             |
| Auth          | `DASHBOARD_SECRET`      | `change_me_immediately` |
| Auth          | `API_TOKEN`             | `change_me_api_token` |
| Auth          | `ADMIN_PASSWORD`        | `admin_pass`       |
| Network       | `NETWORK_NAME`          | `m3tal`            |
| Network       | `LOCAL_IP`              | `127.0.0.1`        |
| Network       | `DOMAIN`                | `localhost`        |
| Storage       | `BASE_STORAGE_PATH`     | `./data`           |
| Storage       | `MEDIA_PATH`            | `./data/media`     |
| Storage       | `CONFIG_PATH`           | `./data/config`    |
| Storage       | `DOWNLOADS_PATH`        | `./data/downloads` |
| Traefik       | `TRAEFIK_WEB_PORT`      | `80`               |
| Traefik       | `TRAEFIK_WEBHTTPS_PORT` | `443`              |
| Traefik       | `TRAEFIK_DASHBOARD_PORT`| `8080`             |
| VPN           | `VPN_USER`              | `user`             |
| VPN           | `VPN_PASSWORD`          | `password`         |

---

## Core

These variables configure fundamental aspects of the M3TAL system.

### `API_PORT`

*   **Description:** The port on which the M3TAL API daemon (Go binary) listens for incoming connections. This port is accessible internally via `host.docker.internal:API_PORT`.
*   **Default Value:** `5050`
*   **Example Value:** `5050`
*   **Used By:**
    *   M3TAL API daemon (Go binary)
    *   M3TAL Dashboard container

### `DASHBOARD_PORT`

*   **Description:** The port on which the M3TAL Dashboard container listens internally. This is the port used for direct access in `local` mode and the target port for Traefik routing in `traefik` mode.
*   **Default Value:** `8082`
*   **Example Value:** `8082`
*   **Used By:**
    *   M3TAL Dashboard container

### `DASHBOARD_EXPOSE_MODE`

*   **Description:** Controls how the M3TAL Dashboard is exposed.
    *   `local`: Exposes the dashboard via a direct port binding (`DASHBOARD_PORT`) on the host. Access via `http://HOST_IP:DASHBOARD_PORT`. Best for LAN-only or initial setups.
    *   `traefik`: Exposes the dashboard through Traefik, typically at `http://dash.DOMAIN`. Traefik must be running. Best for domain-based setups.
*   **Default Value:** `local`
*   **Example Value:** `traefik`
*   **Used By:**
    *   M3TAL Dashboard container (to apply appropriate compose override)

### `HTTP_PORT`

*   **Description:** This variable is not explicitly used in the provided compose files for any specific service port mapping. It might be a legacy or intended for future use. In the context of Traefik, port 80 is the default HTTP entry point.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:** (Currently not explicitly mapped to a service port in provided configs)

### `STATE_DIR`

*   **Description:** The directory where M3TAL stores its state, including the SQLite database (`state.db`).
*   **Default Value:** `./state`
*   **Example Value:** `/var/lib/m3tal/state`
*   **Used By:**
    *   M3TAL API daemon (Go binary)
    *   M3TAL Dashboard container

### `LOG_LEVEL`

*   **Description:** Sets the verbosity of logging for M3TAL services. Common values include `debug`, `info`, `warn`, `error`.
*   **Default Value:** `info`
*   **Example Value:** `debug`
*   **Used By:**
    *   M3TAL API daemon (Go binary)
    *   M3TAL Dashboard container

### `PUID`

*   **Description:** The User ID (UID) under which containers will run. This is important for file permissions on the host system.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:**
    *   M3TAL Dashboard container

### `PGID`

*   **Description:** The Group ID (GID) under which containers will run. This is important for file permissions on the host system.
*   **Default Value:** `1000`
*   **Example Value:** `1000`
*   **Used By:**
    *   M3TAL Dashboard container

### `TZ`

*   **Description:** The timezone to be used by containers, ensuring consistent time logging and operations.
*   **Default Value:** `America/Denver`
*   **Example Value:** `Europe/London`
*   **Used By:**
    *   M3TAL Dashboard container

### `DEBUG_MODE`

*   **Description:** Enables or disables debug mode for the M3TAL system. This can affect logging levels and enable additional diagnostic features.
*   **Default Value:** `false`
*   **Example Value:** `true`
*   **Used By:** (Likely used internally by M3TAL components for conditional logic)

### `METRICS_ENABLED`

*   **Description:** Controls whether performance metrics are collected and exposed by M3TAL services.
*   **Default Value:** `true`
*   **Example Value:** `false`
*   **Used By:** (Likely used internally by M3TAL components for metric collection)

---

## Auth

These variables are critical for securing your M3TAL installation.

### `DASHBOARD_SECRET`

*   **Description:** A secret key used by the M3TAL Dashboard for session management and other security-related functions.
    **NOTE:** This secret is auto-generated on first `m3tal init`. Users should NOT set it manually unless rotating.
*   **Default Value:** `change_me_immediately`
*   **Example Value:** (Auto-generated long string)
*   **Used By:**
    *   M3TAL Dashboard container

### `API_TOKEN`

*   **Description:** A token used for authenticating with the M3TAL API.
    **NOTE:** This token is auto-generated on first `m3tal init`. Users should NOT set it manually unless rotating.
*   **Default Value:** `change_me_api_token`
*   **Example Value:** (Auto-generated long string)
*   **Used By:**
    *   M3TAL API daemon (Go binary) (Likely for internal API-to-API communication or external authentication)

### `ADMIN_PASSWORD`

*   **Description:** The password for the default administrator account of the M3TAL Dashboard.
*   **Default Value:** `admin_pass`
*   **Example Value:** `mySuperSecurePassword123`
*   **Used By:**
    *   M3TAL Dashboard container (for initial login)

---

## Network

These variables control network-related configurations, including domain routing.

### `NETWORK_NAME`

*   **Description:** The name of the Docker network that M3TAL services will use for inter-container communication.
*   **Default Value:** `m3tal`
*   **Example Value:** `m3tal-net`
*   **Used By:**
    *   Docker Compose files

### `LOCAL_IP`

*   **Description:** The IP address used for local network access. Typically `127.0.0.1` for direct host access.
*   **Default Value:** `127.0.0.1`
*   **Example Value:** `127.0.0.1`
*   **Used By:**
    *   M3TAL Dashboard container (in `GO_API_URL` construction if not using `host.docker.internal`)

### `DOMAIN`

*   **Description:** The primary domain name for your M3TAL installation. Setting this variable enables Traefik routing rules for services like `dash.DOMAIN` and `api.DOMAIN`. If `DOMAIN` is not set, Traefik will default to `localhost` for routing.
*   **Default Value:** `localhost`
*   **Example Value:** `m3tal.mydomain.com`
*   **Used By:**
    *   Traefik container (for routing rules)
    *   M3TAL Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`)

---

## Storage

These variables define the locations for storing M3TAL's data.

### `BASE_STORAGE_PATH`

*   **Description:** The base directory for all M3TAL data storage, including configuration, media, and downloads.
    **NOTE:** In production deployments, this defaults to `/mnt`. In template/development setups, it defaults to `./data`.
*   **Default Value:** `./data`
*   **Example Value:** `/mnt/m3tal_storage`
*   **Used By:**
    *   M3TAL Dashboard container (volume mounts)
    *   M3TAL API daemon (Go binary) (for defining path logic)

### `MEDIA_PATH`

*   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` for storing media files.
*   **Default Value:** `./data/media`
*   **Example Value:** `/mnt/m3tal_storage/media`
*   **Used By:**
    *   M3TAL Dashboard container (volume mounts)
    *   M3TAL API daemon (Go binary)

### `CONFIG_PATH`

*   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` for storing configuration files.
*   **Default Value:** `./data/config`
*   **Example Value:** `/mnt/m3tal_storage/config`
*   **Used By:**
    *   M3TAL Dashboard container (volume mounts)
    *   M3TAL API daemon (Go binary)

### `DOWNLOADS_PATH`

*   **Description:** The specific subdirectory within `BASE_STORAGE_PATH` for storing downloaded files.
*   **Default Value:** `./data/downloads`
*   **Example Value:** `/mnt/m3tal_storage/downloads`
*   **Used By:**
    *   M3TAL Dashboard container (volume mounts)
    *   M3TAL API daemon (Go binary)

---

## Traefik

These variables configure the Traefik reverse proxy.

### `TRAEFIK_WEB_PORT`

*   **Description:** The host port that Traefik will use for its HTTP entry point.
*   **Default Value:** `80`
*   **Example Value:** `80`
*   **Used By:**
    *   Traefik container (port mapping)

### `TRAEFIK_WEBHTTPS_PORT`

*   **Description:** The host port that Traefik will use for its HTTPS entry point.
*   **Default Value:** `443`
*   **Example Value:** `443`
*   **Used By:**
    *   Traefik container (port mapping)

### `TRAEFIK_DASHBOARD_PORT`

*   **Description:** The port on which Traefik's own management dashboard is exposed internally on the host. This is typically accessed via `127.0.0.1:TRAEFIK_DASHBOARD_PORT`.
*   **Default Value:** `8080`
*   **Example Value:** `8080`
*   **Used By:**
    *   Traefik container (port mapping)

---

## VPN

These variables are for configuring VPN client settings if M3TAL integrates with a VPN.

### `VPN_USER`

*   **Description:** The username for connecting to a VPN service.
*   **Default Value:** `user`
*   **Example Value:** `myvpnuser`
*   **Used By:** (Likely used by a VPN client component within M3TAL)

### `VPN_PASSWORD`

*   **Description:** The password for connecting to a VPN service.
*   **Default Value:** `password`
*   **Example Value:** `myVpnSecurePassword`
*   **Used By:** (Likely used by a VPN client component within M3TAL)