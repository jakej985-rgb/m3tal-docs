# Environment Variable Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive reference for M3TAL's environment variables. These variables control the core behavior and configuration of your M3TAL system, including the CLI, API daemon, Dashboard, and integrated Docker Compose stacks.

All M3TAL environment variables are primarily read from the `/etc/m3tal/.env` file. This file serves as the single source of truth for configuration across your M3TAL deployment. Both the `m3tal` CLI binary and all Docker Compose stacks (managed by `m3tal up`) explicitly load variables from this file via the `--env-file` option. You can manage these variables using the `m3tal config wizard` or `m3tal config set <KEY> <VALUE>` commands.

---

## Quick Reference

| Variable Name             | Description                                          | Default Value         | Example Value        | Components Using It                                     |
| :------------------------ | :--------------------------------------------------- | :-------------------- | :------------------- | :------------------------------------------------------ |
| `API_PORT`                | Port the M3TAL API daemon listens on.                | `5050`                | `8080`               | `m3tal-api.service`, CLI                                |
| `API_TOKEN`               | Bearer token for API authentication.                 | (Auto-generated)      | `m3t_abcdef123...`   | `m3tal-api.service`, `m3tal-dashboard`, CLI             |
| `M3TAL_STATE_DB`          | Path to the SQLite state database.                   | `/var/lib/m3tal/state.db` | `/var/lib/m3tal/state.db` | `m3tal-api.service`                                     |
| `M3TAL_PLUGINS_DIR`       | Directory for M3TAL API plugins.                     | `/opt/m3tal/plugins`  | `/opt/m3tal/plugins` | `m3tal-api.service`                                     |
| `DASHBOARD_SECRET`        | Secret key for Dashboard session management.         | (Auto-generated)      | `abcdef123xyz...`    | `m3tal-dashboard`                                       |
| `BASE_STORAGE_PATH`       | Base path for persistent media/data storage.         | `/mnt`                | `/mnt/data`          | User stacks, M3TAL internal storage components          |
| `DOMAIN`                  | Base domain for Traefik routing.                     | `""` (empty)          | `example.com`        | `routing-compose.yml` (Traefik), CLI                    |
| `DASHBOARD_EXPOSE_MODE`   | How the M3TAL Dashboard is exposed.                  | `local`               | `traefik`            | CLI (`m3tal dash up`), `m3tal-dashboard`                |
| `DASHBOARD_PORT`          | Port the M3TAL Dashboard container listens on.       | `8082`                | `9000`               | `m3tal-dashboard`, CLI (`m3tal dash up`)                |

---

## Detailed Reference

### Core Configuration

These variables define fundamental operational aspects of the M3TAL system.

#### `M3TAL_STATE_DB`

*   **Description**: Specifies the absolute path to the SQLite database file used by the M3TAL API daemon to persist its state. If not explicitly set, it defaults to the standard location.
*   **Default Value**: `/var/lib/m3tal/state.db`
*   **Example Value**: `/var/lib/m3tal/state.db`
*   **Components Using It**: `m3tal-api.service` (API daemon)

#### `M3TAL_PLUGINS_DIR`

*   **Description**: Defines the directory where the M3TAL API daemon looks for additional plugins. These plugins extend the functionality of the API.
*   **Default Value**: `/opt/m3tal/plugins`
*   **Example Value**: `/opt/m3tal/plugins`
*   **Components Using It**: `m3tal-api.service` (API daemon)

---

### Authentication & Security

Variables related to securing access to M3TAL's components.

#### `API_TOKEN`

*   **Description**: A bearer token used for authenticating requests to the M3TAL API daemon. This token is crucial for secure communication between the CLI, Dashboard, and the API. **Users should generally not set this manually**; it is automatically generated during the initial `m3tal init` process. Only set it if rotating the token.
*   **Default Value**: (Auto-generated random string on `m3tal init`)
*   **Example Value**: `m3t_abcdef1234567890abcdef1234567890abcdef1234567890`
*   **Components Using It**: `m3tal-api.service`, `m3tal-dashboard`, CLI (`/usr/bin/m3tal`)

#### `DASHBOARD_SECRET`

*   **Description**: A secret key used by the M3TAL Dashboard for secure session management and other cryptographic operations. **Users should generally not set this manually**; it is automatically generated during the initial `m3tal init` process. Only set it if rotating the secret.
*   **Default Value**: (Auto-generated random string on `m3tal init`)
*   **Example Value**: `aSuperSecureRandomStringThatIsAtLeast32CharsLong`
*   **Components Using It**: `m3tal-dashboard` container

---

### Network & Routing

These variables control how M3TAL services are exposed and routed over the network, including Traefik integration.

#### `API_PORT`

*   **Description**: The TCP port on which the M3TAL API daemon listens for incoming connections. While the daemon itself typically runs internally on `8080`, this variable allows you to configure the specific port it binds to.
*   **Default Value**: `5050`
*   **Example Value**: `8080` (commonly used internally)
*   **Components Using It**: `m3tal-api.service` (API daemon), CLI (when connecting to API)

#### `DASHBOARD_PORT`

*   **Description**: The internal TCP port on which the M3TAL Dashboard container listens. When `DASHBOARD_EXPOSE_MODE` is `local`, this port is directly mapped to the host. When `DASHBOARD_EXPOSE_MODE` is `traefik`, Traefik routes traffic to this internal port.
*   **Default Value**: `8082`
*   **Example Value**: `9000`
*   **Components Using It**: `m3tal-dashboard` container, CLI (`m3tal dash up`)

#### `DOMAIN`

*   **Description**: Specifies the base domain name for Traefik routing rules. Setting this variable enables domain-based access for M3TAL services, specifically `dash.DOMAIN` for the Dashboard and `api.DOMAIN` for the API daemon. Traefik will automatically configure itself based on this value. If empty, domain-based routing is disabled for M3TAL's core components.
*   **Default Value**: `""` (empty string, meaning domain routing is disabled)
*   **Example Value**: `example.com`
*   **Components Using It**: `routing-compose.yml` (Traefik gateway), CLI (for generating Traefik dynamic config)

#### `DASHBOARD_EXPOSE_MODE`

*   **Description**: Controls how the M3TAL Dashboard container is exposed to the network.
    *   **`local` (default)**: The dashboard is exposed directly on a host port (default `8082`), accessible via `http://HOST_IP:8082`. No Traefik integration is required.
    *   **`traefik`**: The dashboard is exposed via Traefik (if running), accessible via `http://dash.DOMAIN`. This mode requires `DOMAIN` to be set and Traefik to be running.
*   **Default Value**: `local`
*   **Example Value**: `traefik`
*   **Components Using It**: CLI (`m3tal dash up`), `m3tal-dashboard` container (via specific Docker Compose override files)

---

### Storage

Variables related to managing persistent data storage.

#### `BASE_STORAGE_PATH`

*   **Description**: Defines the base directory on the host filesystem where M3TAL components and user-deployed stacks should store persistent media and application data. This is crucial for managing volumes and ensuring data persistence across container restarts. In production deployments, it defaults to a dedicated mount point.
*   **Default Value**: `/mnt` (for production deployments; might be `./data` in development templates)
*   **Example Value**: `/mnt/media` or `/opt/m3tal-data`
*   **Components Using It**: User-deployed Docker Compose stacks that manage persistent data, potentially M3TAL internal storage components.

---

### Traefik

*No dedicated environment variables are defined for this category beyond those already covered in "Network & Routing".* Traefik's behavior is primarily configured via `DOMAIN` and Docker labels.

---

### VPN

*No dedicated environment variables are currently defined for VPN integration.*

---

### System

*No dedicated environment variables are defined for this category beyond those already covered in "Core Configuration" and "Network & Routing".*