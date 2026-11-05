# Environment Variables Reference

Greetings, M3TAL operator. DocSmith here. This document serves as the canonical reference for all environment variables that configure your M3TAL system.

All M3TAL environment variables are read from the `/etc/m3tal/.env` file. Both the `m3tal` CLI binary and all Docker Compose stacks managed by M3TAL (including core services like the Dashboard) are configured to load these variables using the `--env-file` option. This ensures a consistent configuration across your entire deployment.

You can manage these variables using the `m3tal config wizard` for an interactive setup, or `m3tal config set <KEY> <VALUE>` for direct modification.

---

## Quick Reference

| Variable Name         | Description                                        | Default Value | Component(s) Used By                       |
|-----------------------|----------------------------------------------------|---------------|--------------------------------------------|
| `API_PORT`            | Port on which the M3TAL API daemon listens.        | `5050`        | API Daemon, Dashboard, CLI                 |
| `API_TOKEN`           | Bearer token for authenticating with the M3TAL API. | `(empty)`     | API Daemon, Dashboard, CLI                 |
| `M3TAL_STATE_DB`      | Absolute path to the M3TAL SQLite state database.  | `(empty)`     | API Daemon                                 |
| `M3TAL_PLUGINS_DIR`   | Absolute path to the directory containing M3TAL plugins. | `(empty)` | API Daemon                                 |

---

## Detailed Reference

### Core System Variables

These variables configure the fundamental operation and storage paths for the M3TAL system itself.

#### `M3TAL_STATE_DB`

*   **Description**: Defines the absolute filesystem path where the M3TAL API daemon stores its SQLite state database. This database manages the internal state of Docker containers, user configurations, and other system data.
*   **Default Value**: `(empty)` (While the default in `.env` is empty, the API daemon typically auto-creates and uses `/var/lib/m3tal/state.db` if not explicitly set.)
*   **Example Value**: `/var/lib/m3tal/state.db`
*   **Component(s) Used By**: M3TAL API Daemon

#### `M3TAL_PLUGINS_DIR`

*   **Description**: Specifies the absolute filesystem path to the directory where M3TAL plugins are installed and loaded from. Plugins extend the functionality of the M3TAL API.
*   **Default Value**: `(empty)` (Similar to the state database, the API daemon will often look for plugins in a canonical location like `/opt/m3tal/plugins` if this variable is not explicitly defined.)
*   **Example Value**: `/opt/m3tal/plugins`
*   **Component(s) Used By**: M3TAL API Daemon

### Authentication Variables

These variables control how components authenticate with the M3TAL API.

#### `API_TOKEN`

*   **Description**: A bearer token used to authenticate requests to the M3TAL API daemon. This token secures access to M3TAL's internal operations and data.
*   **Default Value**: `(empty)`
*   **Example Value**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ik1JMzNUQUwiLCJpYXQiOjE1MTYyMzkwMjJ9.jS6M_M3t4L_T0k3n_2k6S_M3t4L_T0k3n` (a hypothetical JWT)
*   **Component(s) Used By**: M3TAL API Daemon (validation), M3TAL Dashboard (requests), M3TAL CLI (requests)
*   **Note**: This token is **auto-generated** during the initial `m3tal init` process. Users should generally *not* set or modify this manually unless performing a token rotation or advanced security operations.

### Network Variables

These variables define network-related settings for M3TAL services.

#### `API_PORT`

*   **Description**: Determines the TCP port on which the M3TAL API daemon will listen for incoming connections. This port is typically accessed internally by other M3TAL components (like the Dashboard) or via a reverse proxy (like Traefik).
*   **Default Value**: `5050`
*   **Example Value**: `8080` (commonly used for internal APIs to avoid conflicts with Traefik's default external port 80/443, or other system services)
*   **Component(s) Used By**: M3TAL API Daemon, M3TAL Dashboard, M3TAL CLI (when connecting to local API)