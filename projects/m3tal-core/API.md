# M3TAL CLI Command Reference

Welcome, Architect! This document serves as your definitive guide to the M3TAL Command Line Interface (CLI), your primary interface for managing the M3TAL ecosystem. The `m3tal` binary provides a unified entry point for deploying, configuring, monitoring, and troubleshooting your M3TAL instance, from local desktop applications to headless home server deployments.

M3TAL leverages **Docker Engine** and **Docker Compose V2** for container orchestration, simplifying the deployment of complex services. All commands are designed to abstract away the underlying Docker complexities, allowing you to focus on managing your stacks.

---

## 1. M3TAL System Architecture Overview

The M3TAL ecosystem is built upon several interconnected components:

*   **CLI binary (`/usr/bin/m3tal`)**: A single Go binary installed via APT, serving as the central control point.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a `systemd` service, listening on `localhost:8080`. It manages Docker interactions, maintains the SQLite state database (`/var/lib/m3tal/state.db`), and exposes a REST API for the dashboard and other tools.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask application running within Docker (internal port 8082). It communicates with the API daemon at `http://host.docker.internal:8080` to display system status and provide management controls.
*   **Traefik gateway (`routing-compose.yml`)**: An optional, yet highly recommended, reverse proxy container. It exposes services via domain names on port 80/443, using Docker labels for automatic discovery and a file provider for dynamic configuration (e.g., routing `api.DOMAIN` to the API daemon).
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, providing secure, zero-config internet access to your services without exposing ports directly.

### 1.1 Filesystem Contract

M3TAL relies on a specific filesystem layout for its operation:

| Path                        | Purpose                                                                             |
| :-------------------------- | :---------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary configuration file**. Contains environment variables for M3TAL components. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Stores M3TAL's internal state. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose files and Traefik dynamic configurations. |
| `/docker`                   | **User-facing symlink** to `/opt/m3tal/stack/`. This is where you place your `*-compose.yml` files. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                            |
| `/docker/dynamic/`          | Traefik dynamic configuration files (e.g., `api.yml`).                              |

### 1.2 Dashboard Access Modes

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requires**: No Traefik required. Works out-of-the-box, ideal for first-time users, LAN-only setups, or local testing.

#### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard container. Traefik (if running) then routes `dash.${DOMAIN}` to the dashboard on its internal port 8082.
*   **Access**: `http://dash.DOMAIN` (e.g., `http://dash.yourdomain.com`).
*   **Requires**: Traefik must be running via `m3tal up` (which includes `routing-compose.yml`). Best for domain-based setups and integrating with other services behind a reverse proxy.

### 1.3 Port Map

| Port | Service                    | Access                        |
| :--- | :------------------------- | :---------------------------- |
| 80   | Traefik HTTP entry point   | Public (Traefik mode)         |
| 8080 | M3TAL API daemon (Go)      | Host-local (for dashboard/CLI) |
| 8081 | Traefik dashboard (admin)  | Host-local only               |
| 8082 | M3TAL Dashboard (container) | Direct port (local mode) or via Traefik (traefik mode) |

---

## 2. APT Installation

To install the `m3tal` CLI and its associated daemon, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 3. Core M3TAL Commands

This section details the primary commands for interacting with your M3TAL system.

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a GUI is available.

*   **Usage Example**:
    ```bash
    m3tal
    ```
    *(A new M3TAL Desktop application window will open, connecting to the local API daemon.)*

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for server environments without a graphical interface. This command typically refers to the API daemon's standalone execution for debugging, though in a production system, it's managed by `systemd`. If `port` is provided, it overrides the `API_PORT` for this instance.

*   **Usage Example**:
    ```bash
    m3tal headless
    ```
    *(Starts the API daemon on `localhost:8080` (or `API_PORT` from `.env`) in the foreground.)*
    ```bash
    m3tal headless 9000
    ```
    *(Starts the API daemon on `localhost:9000` for testing purposes.)*

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. Useful for quick health checks and performance monitoring.

*   **Usage Example**:
    ```bash
    m3tal stats
    ```
    *(Outputs current CPU, RAM, disk I/O, GPU utilization (if available), and temperature readings.)*

### `m3tal doctor`

Performs a pre-flight health check of essential M3TAL dependencies and configurations. This includes verifying Docker connectivity, `DASHBOARD_EXPOSE_MODE` validity, required environment variables in `/etc/m3tal/.env`, and port availability.

*   **Usage Example**:
    ```bash
    m3tal doctor
    ```
    *(Checks for Docker daemon, `.env` file presence, critical `DASHBOARD_EXPOSE_MODE` setting, and confirms ports 80, 8080, 8082 are available or in use as expected.)*

---

## 4. Configuration Management (`m3tal config`)

M3TAL configuration is primarily managed through `/etc/m3tal/.env`. The `m3tal config` subcommands provide interactive and direct methods to manage these variables.

### `m3tal config wizard`

Launches an interactive wizard to configure or update the `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL environment or make comprehensive changes.

*   **Usage Example**:
    ```bash
    sudo m3tal config wizard
    ```
    *(Starts a guided prompt asking for values like `DOMAIN`, `DASHBOARD_EXPOSE_MODE`, `PUID`, `PGID`, etc., and saves them to `/etc/m3tal/.env`.)*

### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`. This command is useful for quick, targeted changes.

*   **Usage Example**:
    ```bash
    sudo m3tal config set DOMAIN myhome.tech
    sudo m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```
    *(Updates the `DOMAIN` and `DASHBOARD_EXPOSE_MODE` variables in `/etc/m3tal/.env`.)*

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

*   **Usage Example**:
    ```bash
    m3tal config get PUID
    ```
    *(Outputs the current `PUID` value from `/etc/m3tal/.env`.)*

### `m3tal config scan`

Lists all known environment variables across all configured stacks and their current values (if set in `/etc/m3tal/.env` or derived from defaults).

*   **Usage Example**:
    ```bash
    m3tal config scan
    ```
    *(Displays a comprehensive list of all M3TAL-related environment variables, their defaults, and current overrides.)*

### `m3tal config list`

Prints the entire contents of the current `/etc/m3tal/.env` file to the terminal.

*   **Usage Example**:
    ```bash
    m3tal config list
    ```
    *(Shows all key-value pairs present in your `/etc/m3tal/.env` file.)*

---

## 5. Dashboard Management (`m3tal dash`)

These commands are dedicated to managing the M3TAL Dashboard container (`m3tal-dashboard`). Remember that `m3tal dash up` will download the latest dashboard compose files and apply the correct override based on `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.

### `m3tal dashpass [username] [password]`

Updates the password for a dashboard user. If `username` and `password` are omitted, an interactive prompt will guide you. This modifies `/docker/users.json`.

*   **Usage Example**:
    ```bash
    m3tal dashpass admin newSecurePassword123
    ```
    *(Sets the password for the 'admin' user in `/docker/users.json` to `newSecurePassword123`.)*
    ```bash
    m3tal dashpass
    ```
    *(Prompts you interactively to enter a username and new password.)*

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub, then starts the dashboard container with the appropriate `DASHBOARD_EXPOSE_MODE` override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).

*   **Usage Example**:
    ```bash
    m3tal dash up
    ```
    *(Downloads latest dashboard configs, pulls the dashboard Docker image, and starts `m3tal-dashboard`.)*

    *Access Examples:*
    *   **If `DASHBOARD_EXPOSE_MODE=local`**: Access at `http://localhost:8082` (or `http://YOUR_SERVER_IP:8082`).
    *   **If `DASHBOARD_EXPOSE_MODE=traefik`**: Access at `http://dash.yourdomain.com` (assuming `DOMAIN` is set and Traefik is running via `m3tal up`).

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

*   **Usage Example**:
    ```bash
    m3tal dash down
    ```
    *(Stops and removes the dashboard container.)*

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.

*   **Usage Example**:
    ```bash
    m3tal dash restart
    ```
    *(Restarts the `m3tal-dashboard` container.)*

### `m3tal dash logs`

Streams logs from the `m3tal-dashboard` container to the terminal. Useful for debugging dashboard-related issues.

*   **Usage Example**:
    ```bash
    m3tal dash logs
    ```
    *(Displays live logs from the dashboard container.)*

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container.

*   **Usage Example**:
    ```bash
    m3tal dash status
    ```
    *(Reports if the dashboard container is running, stopped, or in another state.)*

---

## 6. Stack Management (`m3tal up`/`down`/`logs`)

These commands manage all Docker Compose stacks defined by `*-compose.yml` files in the `/docker/` directory. Remember `/docker/` is a symlink to `/opt/m3tal/stack/`.

### Deployment Lifecycle - Day 2 Operations

To deploy a new application or service stack:
1.  Place your Docker Compose file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
2.  Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `sudo m3tal config wizard` or `sudo m3tal config set KEY value`.
3.  Run `m3tal up` to start all stacks, including your newly added one.

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command starts or recreates all defined services in detached mode.

*   **Usage Example**:
    ```bash
    m3tal up
    ```
    *(Starts all services defined in `m3tal-compose.yml`, `routing-compose.yml`, `ollama-compose.yml`, etc., that are present in `/docker/`.)*

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command stops and removes all containers, networks, and volumes defined by your stacks.

*   **Usage Example**:
    ```bash
    m3tal down
    ```
    *(Stops and removes all running M3TAL and user-defined Docker containers.)*

### `m3tal logs`

Streams aggregated logs from all running M3TAL Docker containers (including the dashboard, Traefik, etc.) to the terminal.

*   **Usage Example**:
    ```bash
    m3tal logs
    ```
    *(Displays a consolidated, live stream of logs from all active Docker services managed by M3TAL.)*

---

## 7. Systemd Service Management

The M3TAL API daemon runs as a `systemd` service named `m3tal-api.service`. You can manage this daemon directly using `systemctl`.

*   **Check API daemon status**:
    ```bash
    systemctl status m3tal-api
    ```
    *(Shows if the API daemon is active, running, and its recent logs.)*

*   **Restart the API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
    *(Restarts the M3TAL API daemon, useful after modifying `/etc/m3tal/.env` or for troubleshooting.)*

*   **Stream API daemon logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```
    *(Provides a live stream of logs specifically from the M3TAL API daemon.)*

---

## 8. Fallback: Direct Docker Compose Commands

While `m3tal` is the preferred and most convenient interface, it's essential to understand that it orchestrates **Docker Engine** and **Docker Compose V2** commands under the hood. For advanced debugging or specific scenarios, you can directly interact with Docker Compose.

Remember that `m3tal` operates on all `*-compose.yml` files in `/docker/`. The dashboard specifically uses `m3tal-compose.yml` and its overrides.

*   **Start all M3TAL stacks (equivalent to `m3tal up`)**:
    ```bash
    # Navigate to the stacks directory
    cd /docker/

    # Start all compose files found in the current directory
    docker compose --project-directory . -f *-compose.yml up -d
    ```
    *Note: The `-f *-compose.yml` syntax may require specific shell globbing features. A more robust way might be to list them explicitly or iterate.*
    A more common approach in scripting would be:
    ```bash
    cd /docker/
    for f in *-compose.yml; do docker compose -f "$f" up -d; done
    ```
    Or, using `docker compose`'s multiple file support:
    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f routing-compose.yml -f ollama-compose.yml up -d
    ```

*   **Stop all M3TAL stacks (equivalent to `m3tal down`)**:
    ```bash
    cd /docker/
    docker compose --project-directory . -f *-compose.yml down
    ```
    Or:
    ```bash
    cd /docker/
    for f in *-compose.yml; do docker compose -f "$f" down; done
    ```

*   **Stream aggregated logs from all running containers (equivalent to `m3tal logs`)**:
    ```bash
    cd /docker/
    docker compose --project-directory . -f *-compose.yml logs -f
    ```
    Or:
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/ollama-compose.yml logs -f
    ```

*   **Specific dashboard management using direct Docker Compose**:
    *(Remember `m3tal dash up` selects `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` based on `DASHBOARD_EXPOSE_MODE`.)*
    **To start dashboard in `local` mode**:
    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d m3tal-dashboard
    ```
    **To start dashboard in `traefik` mode**:
    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```
    **To stop the dashboard**:
    ```bash
    cd /docker/
    # You need to specify the main compose file to ensure it knows about the service
    docker compose -f m3tal-compose.yml down m3tal-dashboard
    ```
    **To view dashboard logs**:
    ```bash
    docker compose -f /docker/m3tal-compose.yml logs -f m3tal-dashboard
    ```

---

## 9. Appendix: Environment Variables Reference

These are the core environment variables that M3TAL uses, primarily configured in `/etc/m3tal/.env`.

| Key                         | Default                    | Purpose                                                                                                                              |
| :-------------------------- | :------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| `API_PORT`                  | `5050`                     | Internal port for the M3TAL API daemon (Go binary).                                                                                  |
| `DASHBOARD_PORT`            | `8082`                     | Internal port for the M3TAL Dashboard container.                                                                                     |
| `DASHBOARD_EXPOSE_MODE`     | `local`                    | **CRITICAL**: Controls how the dashboard is exposed (`local` or `traefik`). See [Dashboard Access Modes](#12-dashboard-access-modes). |
| `HTTP_PORT`                 | `8080`                     | (Legacy/Internal) May refer to a general HTTP port. Primarily `API_PORT` and `DASHBOARD_PORT` are in active use.                      |
| `STATE_DIR`                 | `./state`                  | Directory for the SQLite state database and other runtime data. Mapped to `/var/lib/m3tal/state.db`.                                 |
| `LOG_LEVEL`                 | `info`                     | Logging verbosity for the M3TAL API daemon and CLI (`debug`, `info`, `warn`, `error`).                                             |
| `DASHBOARD_SECRET`          | `change_me_immediately`    | Secret key for the M3TAL Dashboard for session management. **Change this immediately!**                                              |
| `API_TOKEN`                 | `change_me_api_token`      | Token for securing access to the M3TAL API. **Change this!**                                                                         |
| `ADMIN_PASSWORD`            | `admin_pass`               | Default admin password for initial dashboard setup. **Change this via `m3tal dashpass`!**                                            |
| `NETWORK_NAME`              | `m3tal`                    | Name of the default Docker network created for M3TAL services.                                                                       |
| `LOCAL_IP`                  | `127.0.0.1`                | Local IP address of the host machine, often used for internal communication.                                                         |
| `DOMAIN`                    | `localhost`                | Your primary domain name, used by Traefik for routing (e.g., `yourdomain.com`).                                                      |
| `VPN_USER`                  | `user`                     | Username for VPN services (if integrated).                                                                                           |
| `VPN_PASSWORD`              | `password`                 | Password for VPN services (if integrated).                                                                                           |
| `BASE_STORAGE_PATH`         | `./data`                   | Root directory for all M3TAL application data and persistent storage.                                                                |
| `MEDIA_PATH`                | `./data/media`             | Path dedicated to media storage.                                                                                                     |
| `CONFIG_PATH`               | `./data/config`            | Path dedicated to application configuration files.                                                                                   |
| `DOWNLOADS_PATH`            | `./data/downloads`         | Path dedicated to downloaded content.                                                                                                |
| `PUID`                      | `1000`                     | User ID for containers to run as, ensuring correct file permissions.                                                                 |
| `PGID`                      | `1000`                     | Group ID for containers to run as, ensuring correct file permissions.                                                                |
| `TZ`                        | `America/Denver`           | Timezone for containers and services.                                                                                                |
| `TRAEFIK_WEB_PORT`          | `80`                       | Host port Traefik binds for HTTP traffic.                                                                                            |
| `TRAEFIK_WEBHTTPS_PORT`     | `443`                      | Host port Traefik binds for HTTPS traffic (if TLS configured).                                                                       |
| `TRAEFIK_DASHBOARD_PORT`    | `8080`                     | Internal port for Traefik's own dashboard, exposed on `127.0.0.1:8081` by `routing-compose.yml`.                                     |
| `DEBUG_MODE`                | `false`                    | Enables debug logging and features across components.                                                                                |
| `METRICS_ENABLED`           | `true`                     | Enables system and application metrics collection.                                                                                   |

---

This concludes the M3TAL CLI Command Reference. With these tools and a clear understanding of the M3TAL architecture, you are well-equipped to manage your M3TAL ecosystem effectively. Happy orchestrating!