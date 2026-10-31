# M3TAL CLI Command Reference

Welcome, M3TAL user! I am DocSmith, your M3TAL Ecosystem Documentation Architect. This document serves as a complete cheat-sheet for the M3TAL Command Line Interface (CLI), providing a comprehensive guide to managing your M3TAL ecosystem.

The M3TAL CLI, accessed via the `m3tal` binary, is your single entry point for interacting with the M3TAL API daemon, managing Docker containers, configuring your environment, and monitoring your system.

---

## M3TAL Core Concepts & Architecture

Understanding the M3TAL system architecture is key to leveraging its full power.

### Components Overview

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary installed via APT. This is the command you type into your terminal.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It manages Docker operations, maintains the SQLite state database, and provides API routes for the CLI and Dashboard.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container that provides a web-based UI. It runs internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes M3TAL services (and your custom stacks) on standard HTTP/S ports (e.g., port 80). It uses a file provider for dynamic routing based on Docker labels and configuration files.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare Tunnel container for secure, zero-config internet access to your M3TAL services without opening firewall ports.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract. These paths are critical for configuration, state, and stack management:

| Path                        | Purpose                                                                             |
| :-------------------------- | :---------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.                  |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files and Traefik configurations.  |
| `/docker`                   | **Symlink** → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                            |

### Dashboard Access Modes (Critical)

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

**1. Local Mode (Default)**

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: M3TAL uses the `m3tal-compose.local.yml` override file, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: You access the dashboard directly via your host's IP address or `localhost` on the specified port.
    *   **Example URL**: `http://YOUR_HOST_IP:8082` or `http://localhost:8082`
*   **Requirements**: No Traefik required. Works immediately on a home server.
*   **Best for**: LAN-only setups, first-time users, local development, or environments without a domain name.

**2. Traefik Mode**

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: M3TAL uses the `m3tal-compose.traefik.yml` override file. This adds specific Traefik labels to the dashboard container, allowing Traefik to automatically discover and route requests for `dash.${DOMAIN}` to the dashboard on its internal port `8082`.
*   **Access**: You access the dashboard via a subdomain routed by Traefik. Traefik (started via `m3tal up`) must be running and configured with a `DOMAIN` in `/etc/m3tal/.env`.
    *   **Example URL**: If `DOMAIN=myhomelab.com`, then `http://dash.myhomelab.com`
*   **Requirements**: Traefik must be deployed and running, typically as part of your core M3TAL stack. Requires a configured `DOMAIN` environment variable.
*   **Best for**: Domain-based setups, exposing multiple services behind a single reverse proxy, or environments where HTTPS is managed by Traefik.

### Docker / Compose Runtime

M3TAL relies heavily on **Docker Engine** and **Docker Compose V2** for container orchestration. These are hard dependencies for the M3TAL system.

*   The `m3tal up` command acts as a wrapper for `docker compose`, executing it across all `*-compose.yml` files found in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container. It first downloads the latest dashboard compose files from GitHub (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`). It then reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env` and starts the dashboard using the appropriate compose override file.
*   You can extend M3TAL by placing your own Docker Compose files (e.g., `my-app-compose.yml`) directly into the `/docker/` directory. M3TAL will automatically include these when running `m3tal up`.

### Deployment Lifecycle — Day 2 Operations

To deploy a new application or service:

1.  **Place your compose file**: Copy your `docker-compose.yml` file into the `/docker/` directory. For example: `/docker/my-app-compose.yml`
2.  **Configure environment variables**: Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. You can use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for specific variables.
3.  **Start all stacks**: Run `m3tal up` to start your new application along with all other M3TAL services.

### Traefik Routing Architecture

Traefik, deployed as a container via `routing-compose.yml`, acts as the central reverse proxy for your M3TAL ecosystem:

*   It binds to port 80 (and 443 if configured for HTTPS) on the host system as its primary entry points.
*   It automatically discovers services by inspecting Docker labels on running containers (e.g., the M3TAL Dashboard when in `traefik` mode).
*   It loads additional dynamic routing configuration from `/etc/traefik/dynamic/` (which typically lives within `/docker/dynamic` on the host due to volume mounts).
*   **API Routing**: It routes `api.${DOMAIN}` to the M3TAL API daemon (running on the host) at `http://host.docker.internal:8080` using a dynamic configuration file like `dynamic/api.yml`.
*   **Dashboard Routing**: It routes `dash.${DOMAIN}` to the M3TAL Dashboard container on its internal port `8082` using Docker labels (only when `DASHBOARD_EXPOSE_MODE=traefik`).

### Port Map

This table outlines the default port assignments within the M3TAL ecosystem:

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public (if Traefik is running)              |
| 8080 | M3TAL API daemon (Go)       | Host-local (from containers via `host.docker.internal`) |
| 8081 | Traefik dashboard           | Host-local only (e.g., `http://127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard (container) | Direct port (local mode) or via Traefik (traefik mode) |

---

## Command Reference

This section provides a complete listing of M3TAL CLI commands with descriptions and practical usage examples.

### Core M3TAL Commands

#### `m3tal` / `m3tal ui`

*   **Description**: Launches the M3TAL Linux Desktop Application, providing a graphical interface for managing your system. This command is primarily for desktop environments.
*   **Usage Example**:
    ```bash
    m3tal
    # OR
    m3tal ui
    ```

#### `m3tal headless [port]`

*   **Description**: Starts M3TAL in headless mode. This is ideal for server environments (e.g., home servers) where a graphical desktop is not available or desired. If no port is specified, it defaults to a standard M3TAL headless port.
*   **Usage Example**:
    ```bash
    # Start headless mode on the default port
    m3tal headless

    # Start headless mode on a specific port, e.g., 9000
    m3tal headless 9000
    ```

#### `m3tal stats`

*   **Description**: Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Provides a quick snapshot of your hardware's health and performance.
*   **Usage Example**:
    ```bash
    m3tal stats
    ```

#### `m3tal doctor`

*   **Description**: Performs a pre-flight health check of your M3TAL installation. This includes verifying Docker connectivity, checking the validity and completeness of `/etc/m3tal/.env`, and ensuring required ports are available. Essential for troubleshooting initial setups.
*   **Usage Example**:
    ```bash
    m3tal doctor
    ```

### Configuration Management (`m3tal config`)

#### `m3tal config wizard`

*   **Description**: Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL environment variables.
*   **Usage Example**:
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`

*   **Description**: Sets a single environment variable within your `/etc/m3tal/.env` file. This command will create or update the specified key-value pair.
*   **Usage Example**:
    ```bash
    # Set your domain for Traefik routing
    m3tal config set DOMAIN myhomelab.com

    # Switch the dashboard to Traefik exposure mode
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`

*   **Description**: Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.
*   **Usage Example**:
    ```bash
    # Get the currently configured User ID
    m3tal config get PUID
    ```

#### `m3tal config scan`

*   **Description**: Scans all active Docker Compose files in `/docker/` (and their overrides) to identify and list all environment variables referenced. It provides an overview of which variables are expected across your entire stack.
*   **Usage Example**:
    ```bash
    m3tal config scan
    ```

#### `m3tal config list`

*   **Description**: Prints the entire content of your current `/etc/m3tal/.env` file to the terminal, showing all configured environment variables.
*   **Usage Example**:
    ```bash
    m3tal config list
    ```

### Dashboard Management (`m3tal dash`)

#### `m3tal dashpass [username] [password]`

*   **Description**: Updates the password for a user on the M3TAL Dashboard. If `username` and `password` are omitted, the command will launch an interactive prompt to securely set credentials. This manages the `/docker/users.json` file.
*   **Usage Example**:
    ```bash
    # Set the 'admin' user's password to 'SuperStrongPassword123'
    m3tal dashpass admin SuperStrongPassword123

    # Interactively set a password (prompts for username and password)
    m3tal dashpass
    ```

#### `m3tal dash up`

*   **Description**: Pulls the latest M3TAL Dashboard Docker Compose configuration from GitHub and then starts the dashboard container. It respects the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env` to apply the correct override (local or Traefik).
*   **Usage Example**:
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`

*   **Description**: Stops and removes the M3TAL Dashboard container.
*   **Usage Example**:
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`

*   **Description**: Restarts the M3TAL Dashboard container. Useful after configuration changes or for troubleshooting.
*   **Usage Example**:
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`

*   **Description**: Streams the logs from the M3TAL Dashboard container to your terminal. Useful for monitoring activity or diagnosing issues.
*   **Usage Example**:
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`

*   **Description**: Shows the current status of the M3TAL Dashboard container (e.g., running, stopped, exited).
*   **Usage Example**:
    ```bash
    m3tal dash status
    ```

### Stack Management (`m3tal up`, `m3tal down`, `m3tal logs`)

#### `m3tal up`

*   **Description**: Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`). This command starts all your M3TAL services and any custom stacks you've added. It should be run from the host, and ideally in detached mode.
*   **Usage Example**:
    ```bash
    # Start all M3TAL stacks in detached mode (recommended for servers)
    m3tal up -d

    # Start all M3TAL stacks in foreground (for debugging)
    m3tal up
    ```

#### `m3tal down`

*   **Description**: Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory, gracefully stopping and removing all services, networks, and volumes associated with your M3TAL stacks.
*   **Usage Example**:
    ```bash
    m3tal down
    ```

#### `m3tal logs`

*   **Description**: Streams aggregated logs from all currently running M3TAL Docker containers. This provides a consolidated view of activity across your entire M3TAL ecosystem.
*   **Usage Example**:
    ```bash
    # Stream logs from all containers in real-time
    m3tal logs -f

    # View logs from all containers since last restart
    m3tal logs
    ```

---

## Advanced Topics

### Systemd Service Management

The M3TAL API daemon runs as a `systemd` service named `m3tal-api.service`. This allows it to start automatically on boot and be managed like other system services.

*   **Check API daemon status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart the API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **View API daemon logs in real-time**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Enable API daemon to start on boot**:
    ```bash
    sudo systemctl enable m3tal-api
    ```
*   **Disable API daemon from starting on boot**:
    ```bash
    sudo systemctl disable m3tal-api
    ```

### Direct Docker Compose Fallback

While the `m3tal` CLI is the primary interface, understanding how to use `docker compose` directly can be invaluable for advanced debugging or specific container operations. M3TAL utilizes Docker Compose V2.

All M3TAL's core compose files and your custom stacks are located in `/docker/` (which is `/opt/m3tal/stack/`).

*   **Run a specific stack directly in detached mode**:
    ```bash
    # Example: Starting only the Traefik and Cloudflared services
    sudo docker compose -f /docker/routing-compose.yml up -d

    # Example: Starting the dashboard with its local override
    sudo docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
    ```
*   **Stop a specific stack directly**:
    ```bash
    # Example: Stopping only the Traefik and Cloudflared services
    sudo docker compose -f /docker/routing-compose.yml down
    ```
*   **View logs for specific services using direct compose**:
    ```bash
    # Example: Viewing logs for the dashboard
    sudo docker compose -f /docker/m3tal-compose.yml logs -f m3tal-dashboard
    ```
*   **List all active compose projects and their containers**:
    ```bash
    sudo docker compose ls
    sudo docker ps
    ```

---

## Installation

To install M3TAL on your Debian/Ubuntu-based system, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```