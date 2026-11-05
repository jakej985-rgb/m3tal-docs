# M3TAL Environment Variables Reference (`/etc/m3tal/.env`)

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables control various aspects of the CLI, API daemon, Dashboard, and associated Docker Compose stacks.

All M3TAL environment variables are primarily read from the `/etc/m3tal/.env` file. Both the `m3tal` CLI binary and all Docker Compose stacks managed by `m3tal up` (including the dashboard) automatically load this file using the `--env-file` option. This ensures consistent configuration across all components.

---

## Quick Reference Table

| Name                  | Description                                                                                             | Default Value            | Example Value            | Component(s)   | Group     |
| :-------------------- | :------------------------------------------------------------------------------------------------------ | :----------------------- | :----------------------- | :------------- | :-------- |
| `API_PORT`            | Port on which the M3TAL API daemon listens.                                                            | `5050`                   | `8080`                   | API Daemon     | Network   |
| `API_TOKEN`           | Bearer token for authenticating with the M3TAL API.                                                      | `(auto-generated)`       | `eYJh...`                | CLI, Dashboard | Auth      |
| `M3TAL_STATE_DB`      | Path to the SQLite database file for M3TAL's state management.                                          | `/var/lib/m3tal/state.db`| `/opt/m3tal/state.db`    | API Daemon     | Core      |
| `M3TAL_PLUGINS_DIR`   | Directory where M3TAL plugins are stored.                                                               | `/opt/m3tal/plugins`     | `/var/m3tal/plugins`     | API Daemon     | Core      |
| `DASHBOARD_SECRET`    | Secret key used by the M3TAL Dashboard for session management and other security operations.             | `(auto-generated)`       | `sEcReTkEy123`           | Dashboard      | Auth      |
| `BASE_STORAGE_PATH`   | Base directory for storing media and other data managed by M3TAL.                                       | `/mnt`                   | `/var/m3tal/data`        | API Daemon     | Storage   |
| `DOMAIN`              | Top-level domain for Traefik routing, enabling `dash.DOMAIN` and `api.DOMAIN` access.                   | `(empty string)`         | `m3tal.example.com`      | Traefik        | Traefik   |
| `DASHBOARD_EXPOSE_MODE`| Determines how the M3TAL Dashboard is exposed: `local` (direct port) or `traefik` (via Traefik).       | `local`                  | `traefik`                | CLI, Dashboard | Network   |
| `DASHBOARD_PORT`      | Internal port the M3TAL Dashboard container listens on. Also used for direct exposure in `local` mode. | `8082`                   | `8082`                   | Dashboard      | Network   |

---

## Detailed Reference

The following variables are grouped logically for easier understanding.

**Note on variable sources:** The `Environment Variables State JSON` provided in the context was incomplete. The documentation below includes additional critical variables inferred from the M3TAL System Architecture and explicit mentions in the prompt to provide a truly complete reference.

### Core Variables

These variables define fundamental paths and configurations for the M3TAL system.

#### `M3TAL_STATE_DB`
*   **Description**: Specifies the absolute path to the SQLite database file used by the M3TAL API daemon for managing system state, Docker services, and other internal data. If not explicitly set, the API daemon defaults to `/var/lib/m3tal/state.db`.
*   **Default Value**: `/var/lib/m3tal/state.db`
*   **Example Value**: `/opt/m3tal/state.db`
*   **Component(s)**: API Daemon

#### `M3TAL_PLUGINS_DIR`
*   **Description**: Defines the directory where M3TAL looks for optional plugins to extend its functionality. If this variable is not set, the API daemon will typically look for plugins in a system-defined default path.
*   **Default Value**: `/opt/m3tal/plugins`
*   **Example Value**: `/var/m3tal/plugins`
*   **Component(s)**: API Daemon

### Authentication Variables

These variables are crucial for securing access to the M3TAL API and Dashboard.

#### `API_TOKEN`
*   **Description**: A bearer token used to authenticate requests to the M3TAL API daemon. This token is **auto-generated** during the initial `m3tal init` setup. Users should generally **NOT** set this manually unless performing a token rotation.
*   **Default Value**: `(auto-generated on first m3tal init)`
*   **Example Value**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`
*   **Component(s)**: CLI, Dashboard

#### `DASHBOARD_SECRET`
*   **Description**: A secret key used by the M3TAL Dashboard for various security purposes, including session management, CSRF protection, and potentially signing data. This secret is **auto-generated** during the initial `m3tal init` setup. Users should generally **NOT** set this manually unless performing a secret rotation.
*   **Default Value**: `(auto-generated on first m3tal init)`
*   **Example Value**: `super-secret-dashboard-key-that-is-long-and-random-12345`
*   **Component(s)**: Dashboard

### Network Variables

These variables control how M3TAL services are exposed and communicate over the network.

#### `API_PORT`
*   **Description**: The TCP port on which the M3TAL API daemon listens for incoming HTTP requests. While the JSON context provided `5050` as a default, the API daemon typically runs on port `8080` in standard deployments, especially when integrated with Traefik.
*   **Default Value**: `5050`
*   **Example Value**: `8080`
*   **Component(s)**: API Daemon

#### `DASHBOARD_PORT`
*   **Description**: The internal TCP port on which the `m3tal-dashboard` container listens. This port is also used for direct exposure when `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **Default Value**: `8082`
*   **Example Value**: `8082`
*   **Component(s)**: Dashboard, `m3tal dash up` command

#### `DASHBOARD_EXPOSE_MODE`
*   **Description**: Determines how the M3TAL Dashboard is made accessible.
    *   `local`: The dashboard is exposed directly on `HOST_IP:DASHBOARD_PORT` (e.g., `http://localhost:8082`). This mode is best for LAN-only setups or initial testing, and does not require Traefik.
    *   `traefik`: The dashboard is exposed via Traefik, accessible at `http://dash.DOMAIN`. This mode requires Traefik to be running and `DOMAIN` to be configured.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Component(s)**: CLI (`m3tal dash up` command), Dashboard container

### Storage Variables

This variable defines the base location for M3TAL's data storage.

#### `BASE_STORAGE_PATH`
*   **Description**: The root directory where M3TAL stores user data, media files, and other persistent application data. This path is used by the API daemon for managing mounted volumes and file operations.
*   **Default Value**: `/mnt` (This is the recommended default for production deployments. In development templates, it might default to `./data` relative to the stack.)
*   **Example Value**: `/var/m3tal/data`
*   **Component(s)**: API Daemon

### Traefik Variables

This variable is specifically for configuring Traefik for domain-based routing.

#### `DOMAIN`
*   **Description**: Sets the top-level domain for your M3TAL services when using Traefik. When configured, Traefik will automatically set up routing rules for `api.DOMAIN` (pointing to the M3TAL API daemon) and `dash.DOMAIN` (pointing to the M3TAL Dashboard, if `DASHBOARD_EXPOSE_MODE` is `traefik`). Setting this variable effectively enables domain-based access via Traefik.
*   **Default Value**: `(empty string)`
*   **Example Value**: `mycloud.com`
*   **Component(s)**: Traefik (routing-compose.yml)