# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are read from the `/etc/m3tal/.env` file by both the M3TAL CLI and all Docker Compose stacks via the `--env-file` option.

## Quick Reference Table

| Variable Name          | Description                                                                                                       | Default Value | Example Value               | Used By             |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------- | :------------ | :-------------------------- | :------------------ |
| `API_PORT`             | The port on which the M3TAL API daemon listens.                                                                   | `5050`        | `8080`                      | API Daemon          |
| `API_TOKEN`            | API token for authentication. Auto-generated on first `m3tal init`. Do not set manually unless rotating.           | `""`          | `a1b2c3d4e5f67890`          | API Daemon, CLI     |
| `M3TAL_STATE_DB`       | Path to the M3TAL state database.                                                                                 | `""`          | `/var/lib/m3tal/state.db`   | API Daemon, CLI     |
| `M3TAL_PLUGINS_DIR`    | Directory where M3TAL plugins are located.                                                                        | `""`          | `/opt/m3tal/plugins`        | API Daemon, CLI     |

---

## Core Configuration

### `API_PORT`

*   **Description**: The port on which the M3TAL API daemon listens for incoming requests.
*   **Default Value**: `5050`
*   **Example Value**: `8080`
*   **Used By**: API Daemon

### `API_TOKEN`

*   **Description**: This environment variable holds the API token used for authenticating requests to the M3TAL API. **This token is auto-generated on the first `m3tal init` command and should NOT be set manually unless you are intentionally rotating the token for security reasons.**
*   **Default Value**: `""` (empty string)
*   **Example Value**: `a1b2c3d4e5f67890`
*   **Used By**: API Daemon, CLI

### `M3TAL_STATE_DB`

*   **Description**: Specifies the file path for the M3TAL state database. This SQLite database stores crucial information about your M3TAL setup. If not explicitly set, the API daemon will default to `/var/lib/m3tal/state.db`.
*   **Default Value**: `""` (empty string, defaults to `/var/lib/m3tal/state.db` within the API Daemon)
*   **Example Value**: `/var/lib/m3tal/state.db`
*   **Used By**: API Daemon, CLI

### `M3TAL_PLUGINS_DIR`

*   **Description**: Defines the directory where M3TAL plugins are located. The M3TAL CLI and API daemon will look in this directory for any installed plugins.
*   **Default Value**: `""` (empty string, typically defaults to `/opt/m3tal/plugins` in production deployments)
*   **Example Value**: `/opt/m3tal/plugins`
*   **Used By**: API Daemon, CLI

---

## Authentication

(No specific authentication-related environment variables defined in the provided JSON.)

---

## Network

(No specific network-related environment variables defined in the provided JSON.)

---

## Storage

### `BASE_STORAGE_PATH`

*   **Description**: Controls where M3TAL stores its media data and other persistent files. In production deployments, this defaults to `/mnt`. In development or template setups, it might default to `./data`. Ensure this path is accessible and has sufficient space for your data.
*   **Default Value**: `/mnt` (in production) or `./data` (in template/development)
*   **Example Value**: `/mnt/m3tal_data`
*   **Used By**: Various services requiring persistent storage.

---

## Traefik

### `DOMAIN`

*   **Description**: The base domain name for your M3TAL deployment. Setting this variable enables Traefik routing rules, allowing access to services like the dashboard and API via `dash.DOMAIN` and `api.DOMAIN` respectively.
*   **Default Value**: `""` (empty string)
*   **Example Value**: `m3tal.example.com`
*   **Used By**: Traefik, Dashboard (when `DASHBOARD_EXPOSE_MODE=traefik`)

### `DASHBOARD_EXPOSE_MODE`

*   **Description**: Determines how the M3TAL dashboard is exposed.
    *   `local`: Exposes the dashboard via a direct port binding (`${DASHBOARD_PORT:-8082}:8082`). Access is typically via `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik is required for this mode. This is the default and best for LAN-only setups or initial testing.
    *   `traefik`: Exposes the dashboard via Traefik. Traefik will route traffic from `dash.DOMAIN` to the dashboard container. Traefik must be running for this mode to work. This is best for domain-based setups with multiple services behind a reverse proxy.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Used By**: Dashboard container configuration, CLI

### `DASHBOARD_PORT`

*   **Description**: The port on which the M3TAL dashboard is exposed when `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **Default Value**: `8082`
*   **Example Value**: `8082`
*   **Used By**: Dashboard container configuration (when `DASHBOARD_EXPOSE_MODE=local`)

---

## VPN

(No specific VPN-related environment variables defined in the provided JSON.)

---

## System

(No specific system-related environment variables defined in the provided JSON.)

---

## Filesystem Contract

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` command and can also be modified using `m3tal config set KEY value`. Both the M3TAL CLI and all Docker Compose stacks read their environment variables from this file.

Other important filesystem locations include:

*   `/var/lib/m3tal/state.db`: The SQLite state database, auto-created by the API daemon.
*   `/opt/m3tal/stack/`: The canonical directory containing M3TAL stack files, including compose files and Traefik configuration.
*   `/docker/`: A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations.
*   `/docker/users.json`: Stores dashboard credentials and is managed by the `m3tal dashpass` command.

---

## APT Installation

To install M3TAL and ensure proper environment variable handling, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```