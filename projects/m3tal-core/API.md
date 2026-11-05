# M3TAL CLI Command Reference

Welcome to the M3TAL Ecosystem. This document serves as your definitive cheat-sheet for the `m3tal` command-line interface (CLI), providing comprehensive usage examples and insights into the underlying architecture.

The `m3tal` CLI is a single Go binary (`/usr/bin/m3tal`) that acts as the unified entrypoint for managing your M3TAL home server environment. It interacts with the M3TAL API daemon (a Go service running on port 8080) and orchestrates Docker containers using Docker Compose V2.

---

## M3TAL Filesystem Contract

Understanding the M3TAL filesystem layout is crucial for effective management.

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary system configuration file. Manages environment variables for the M3TAL API and Docker Compose stacks. |
| `/var/lib/m3tal/state.db` | SQLite state database used by the M3TAL API daemon to persist application state. Automatically created. |
| `/opt/m3tal/stack/` | Canonical directory for Docker Compose files and Traefik dynamic configuration. |
| `/docker` | **Symlink to `/opt/m3tal/stack/`**. This is the user-facing path where all Docker Compose stacks reside. |
| `/docker/users.json` | Credential store for the M3TAL Dashboard. Managed securely by `m3tal dashpass`. |

---

## Core CLI Commands

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where the GUI can be displayed.

```bash
m3tal
# Or explicitly:
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode. This is ideal for home servers or environments without a graphical interface. The M3TAL API daemon will start on its configured port (default 8080), but this command can also launch a separate UI if needed, listening on the specified port.

```bash
# Start M3TAL in headless mode (API daemon will run)
m3tal headless

# Start M3TAL in headless mode, with a UI available on port 9000
m3tal headless 9000
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Provides a quick overview of your system's health and performance.

```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check to ensure your M3TAL environment is ready. It verifies Docker connectivity, checks for `.env` file validity, and scans for port availability.

```bash
m3tal doctor
```

---

## Configuration Management

M3TAL's core configuration is stored in `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL environment.

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env`. This command is useful for quick, non-interactive changes.

```bash
# Set the primary domain for Traefik routing
m3tal config set DOMAIN myhome.lan

# Configure the dashboard's exposure mode
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

```bash
# Retrieve the configured domain
m3tal config get DOMAIN
```

### `m3tal config scan`

Scans and lists all environment variables referenced across all Docker Compose files in `/docker/` (i.e., `/opt/m3tal/stack/`). This helps identify all required variables for your deployed stacks.

```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of the `/etc/m3tal/.env` file to the terminal.

```bash
m3tal config list
```

---

## Dashboard Management

The M3TAL Dashboard is a central web interface for managing your system.

### `m3tal dashpass [username] [password]`

Manages user credentials for the M3TAL Dashboard. This command updates the `/docker/users.json` file. If `username` and `password` are omitted, it will launch an interactive prompt.

```bash
# Set a new password for the 'admin' user
m3tal dashpass admin MySecureDashboardPass123

# Interactively create or update a user password
m3tal dashpass
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub, then starts the `m3tal-dashboard` container. This command intelligently selects the correct Compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

```bash
m3tal dash up
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container.

```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams aggregated logs from the `m3tal-dashboard` container, useful for debugging.

```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, unhealthy).

```bash
m3tal dash status
```

---

## Stack Management

M3TAL uses Docker Compose to manage all services as "stacks." User-defined stacks are placed in `/docker/`.

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command brings up all your configured services, including Traefik if `routing-compose.yml` is present.

```bash
# Start all M3TAL-managed Docker Compose stacks
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes associated with your M3TAL stacks.

```bash
# Stop and remove all M3TAL-managed Docker Compose stacks
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a unified view of all service output.

```bash
m3tal logs
```

---

## System Monitoring & Health

### `m3tal doctor` (already listed, but worth reinforcing here)

Performs a pre-flight health check to ensure your M3TAL environment is ready. It verifies Docker connectivity, checks for `.env` file validity, and scans for port availability.

```bash
m3tal doctor
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage it directly using `systemctl`.

### Check API service status

```bash
systemctl status m3tal-api
```

### Restart the API service

```bash
systemctl restart m3tal-api
```

### Stream API service logs

```bash
journalctl -u m3tal-api -f
```

---

## Docker Fallback Commands

M3TAL relies on **Docker Engine** and **Docker Compose V2**. The `m3tal` CLI abstracts many `docker compose` commands, but you can always interact directly with Docker if needed.

All M3TAL's Docker Compose files reside in `/docker/` (which symlinks to `/opt/m3tal/stack/`).

### Common Docker Compose Operations

To bring up specific stacks directly:

```bash
# Start the routing (Traefik/Cloudflared) stack
docker compose -f /docker/routing-compose.yml up -d

# Start your custom user stack
docker compose -f /docker/my-custom-app-compose.yml up -d
```

To stop specific stacks directly:

```bash
# Stop the routing stack
docker compose -f /docker/routing-compose.yml down

# Stop your custom user stack
docker compose -f /docker/my-custom-app-compose.yml down
```

### Dashboard-Specific Docker Compose Considerations

The M3TAL Dashboard (`m3tal-dashboard`) uses different compose files based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

- **If `DASHBOARD_EXPOSE_MODE=local`**:
  ```bash
  # Bring up the dashboard in local mode
  docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
  ```

- **If `DASHBOARD_EXPOSE_MODE=traefik`**:
  ```bash
  # Bring up the dashboard for Traefik routing
  docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d
  ```

The base `m3tal-compose.yml` provides the core dashboard definition, and the override files add specific network or port configurations.

---

## M3TAL System Architecture Overview

### Components

-   **CLI binary** (`/usr/bin/m3tal`): Your primary interface, a unified Go binary installed via APT.
-   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on `http://localhost:8080`. It manages Docker, the state DB, and provides API routes for the CLI and other components.
-   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container running internally on port 8082. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
-   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services on host port 80. It uses a file provider for dynamic routing and automatically discovers services via Docker labels.
-   **Cloudflared** (`routing-compose.yml`): An optional Docker container that establishes a Cloudflare Tunnel for zero-config, secure internet access to your services.

---

## Dashboard Access — Two Modes

The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Mode 1: `local` (Default)

-   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
-   **Compose Files**: Uses the base `m3tal-compose.yml` combined with the `m3tal-compose.local.yml` override.
-   **Port Binding**: This mode adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
-   **Access URL**: `http://HOST_IP:8082` or `http://localhost:8082`
-   **Traefik Requirement**: No Traefik required. Works out of the box.
-   **Best For**: LAN-only setups, first-time users, local testing, or scenarios where you don't need domain-based access.

### Mode 2: `traefik`

-   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
-   **Compose Files**: Uses the base `m3tal-compose.yml` combined with the `m3tal-compose.traefik.yml` override.
-   **Traefik Integration**: This mode adds specific Traefik labels to the dashboard container, allowing Traefik to route `dash.${DOMAIN}` to the dashboard on its internal port 8082.
-   **Access URL**: `http://dash.YOUR_CONFIGURED_DOMAIN` (e.g., `http://dash.myhome.lan`). Requires Traefik to be running via `m3tal up`.
-   **Best For**: Domain-based setups, integrating the dashboard behind a reverse proxy, and managing multiple services under a single domain.

---

## Traefik Routing Architecture

Traefik is crucial for exposing M3TAL services via domain names.

-   **Deployment**: Traefik runs as a Docker container, deployed via `routing-compose.yml` in `/docker/`.
-   **Entry Point**: It binds host port 80 (and optionally 443 for HTTPS) as its HTTP entry point.
-   **Service Discovery**: Traefik automatically discovers services by reading Docker labels on running containers.
-   **Dynamic Configuration**: It loads additional dynamic configuration from `/docker/dynamic/` (which uses a file provider for hot-reloading changes).
-   **API Daemon Routing**: A dynamic configuration file like `dynamic/api.yml` routes `api.DOMAIN` directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080`.
-   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, the `m3tal-dashboard` container's `m3tal-compose.traefik.yml` override adds Traefik labels, routing `dash.DOMAIN` to the dashboard container's internal port 8082.

---

## Deployment Lifecycle — Day 2 Operations

To add and manage your own Docker Compose stacks with M3TAL:

1.  **Place your Compose file**: Create or place your `my-stack-compose.yml` file directly into `/docker/`.
    *Example: `/docker/plex-compose.yml`*
2.  **Configure environment variables**: Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`.
    *Use `m3tal config wizard` for interactive setup, or `m3tal config set KEY value` for specific variables.*
3.  **Start all stacks**: Run `m3tal up` to deploy your new stack alongside all other M3TAL services.
    *This command will discover your new `*-compose.yml` file and include it in the `docker compose up` operation.*

---

## M3TAL Port Map

| Port | Service | Access |
|------|---------|--------|
| 80 | Traefik HTTP entry point | Public (when Traefik is active) |
| 8080 | M3TAL API daemon (Go) | Host-local |
| 8081 | Traefik dashboard | Host-local only (for Traefik management UI) |
| 8082 | M3TAL Dashboard container | Direct port (local mode) or via Traefik (traefik mode) |

---

## APT Installation

To install the M3TAL CLI and API daemon on Debian/Ubuntu-based systems:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## Environment Variables

These are some of the environment variables M3TAL interacts with, primarily configured in `/etc/m3tal/.env`.

| Key | Purpose | Default |
|-----|---------|---------|
| `API_PORT` | Port the M3TAL API daemon listens on (though daemon defaults to 8080) | `5050` (Configurable, actual daemon port is 8080 by default) |
| `API_TOKEN` | Authentication token for API access | `""` (Empty by default, should be set for security) |
| `M3TAL_STATE_DB` | Path to the SQLite state database | `/var/lib/m3tal/state.db` |
| `M3TAL_PLUGINS_DIR` | Directory where M3TAL plugins are stored | `/opt/m3tal/plugins/` |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed (`local` or `traefik`) | `local` |
| `DASHBOARD_PORT` | Host port for direct dashboard access (if `local` mode) | `8082` |
| `DOMAIN` | Primary domain name for Traefik routing | `""` (Must be set for Traefik mode) |
| `TZ` | Timezone for containers | `UTC` (or your local timezone, e.g., `America/New_York`) |