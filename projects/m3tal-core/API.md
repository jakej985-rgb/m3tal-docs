# `docs/COMMAND_REFERENCE.md` — M3TAL CLI Cheat-Sheet

Welcome, Operator. You are now equipped with the M3TAL Command-Line Interface (CLI), the primary interface for managing your M3TAL ecosystem. This document serves as your complete cheat-sheet, detailing every command, its purpose, and practical usage examples.

The M3TAL CLI, a unified Go binary, provides a single entry point for orchestrating your entire M3TAL stack, from system diagnostics to Docker container management.

---

## M3TAL System Architecture Overview

Before diving into commands, let's establish the fundamental architecture of the M3TAL ecosystem. Understanding these components and their interactions is key to effective management.

### Components

*   **CLI binary** (`/usr/bin/m3tal`): Your primary interface. A Go binary installed via APT, responsible for translating your commands into actions, interacting with the API daemon and Docker.
*   **API daemon** (`m3tal-api.service`): The heart of M3TAL's automation. A Go binary running as a systemd service on `http://localhost:8080`. It manages Docker operations, maintains the SQLite state database, and exposes an internal API for the CLI and other components.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask web application running inside Docker, communicating with the API daemon at `http://host.docker.internal:8080`. It provides a user-friendly graphical interface.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing your services (including the M3TAL dashboard and API) by domain name on host port 80. It uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container providing secure, zero-config internet access to your services via Cloudflare Tunnels, eliminating the need for port forwarding.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract. Familiarize yourself with these paths:

| Path                        | Purpose                                                                                                                                                                                                                                                                                        |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary Configuration File**. This file holds all critical environment variables for the M3TAL API daemon and Docker Compose stacks. It is managed by `m3tal config wizard` and `m3tal config set/get`.                                                                                       |
| `/var/lib/m3tal/state.db`   | **SQLite State Database**. An auto-created SQLite database used by the M3TAL API daemon to persist application state, configuration, and potentially plugin data.                                                                                                                                 |
| `/opt/m3tal/stack/`         | **Canonical Stack Directory**. This is the authoritative location where M3TAL's internal Docker Compose files (`m3tal-compose.yml`, `routing-compose.yml`, etc.) reside. User-defined `*-compose.yml` files can also be placed here.                                                              |
| `/docker`                   | **User-Facing Stack Symlink**. A symbolic link pointing to `/opt/m3tal/stack/`. This is the recommended and user-friendly path for placing your custom `*-compose.yml` files and other Docker-related configurations. All `m3tal up` and `m3tal down` operations iterate through this directory. |
| `/docker/users.json`        | **Dashboard Credential Store**. This file securely stores hashed credentials for the M3TAL Dashboard. It is exclusively managed by the `m3tal dashpass` command. **Do not modify manually.**                                                                                                        |
| `/docker/dynamic/`          | **Traefik Dynamic Configuration**. This directory contains `.yml` files for Traefik's file provider, allowing dynamic routing rules to be loaded and hot-reloaded without restarting Traefik. For example, `api.yml` routes `api.DOMAIN` to the M3TAL API.                                          |

### Dashboard Access — Two Modes

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Override File:** Uses `m3tal-compose.local.yml`. This override adds a direct port binding for the dashboard container.
*   **Access Method:** The dashboard container is directly exposed on a host port (default: `8082`).
*   **Access URL:** `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requirements:** No Traefik required. Works out of the box for initial setup or local network access.
*   **Best For:** LAN-only setups, first-time users, local development, or scenarios where a reverse proxy isn't desired.

**Example Configuration for Local Mode (`/etc/m3tal/.env`):**

```bash
DASHBOARD_EXPOSE_MODE=local
DASHBOARD_PORT=8082
```

#### Mode 2: `traefik`

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Override File:** Uses `m3tal-compose.traefik.yml`. This override adds specific Traefik labels to the dashboard service.
*   **Access Method:** Traefik (running via `routing-compose.yml`) intercepts requests for `dash.DOMAIN` and routes them internally to the dashboard container. No direct host port binding for the dashboard.
*   **Access URL:** `http://dash.YOUR_DOMAIN` (e.g., `http://dash.myhomelab.com`)
*   **Requirements:** Traefik must be running (`m3tal up` starts it), and your `DOMAIN` variable must be correctly configured in `/etc/m3tal/.env` and resolve to your M3TAL host.
*   **Best For:** Domain-based setups, exposing multiple services behind a unified reverse proxy, production environments.

**Example Configuration for Traefik Mode (`/etc/m3tal/.env`):**

```bash
DASHBOARD_EXPOSE_MODE=traefik
DOMAIN=myhomelab.com
```

### Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its underlying containerization and orchestration platform. These are hard dependencies for the M3TAL ecosystem.

*   **Unified Stack Management:** The `m3tal up` command orchestrates all Docker Compose stacks defined by `*-compose.yml` files found within the `/docker/` directory. It effectively runs `docker compose --project-directory /docker/ up -d`.
*   **Dashboard-Specific Management:** The `m3tal dash up` command provides specialized management for the M3TAL dashboard. It performs the following steps:
    1.  Downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files from GitHub to ensure your dashboard is always up-to-date.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the base `m3tal-compose.yml` combined with the appropriate override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the configured exposure mode.
*   **User Stacks:** To add your own custom Docker services, simply place your `my-stack-compose.yml` file into the `/docker/` directory. M3TAL will automatically pick it up with the next `m3tal up` command.

### Deployment Lifecycle — Day 2 Operations

Integrating a new Docker stack into your M3TAL ecosystem is straightforward:

1.  **Place Compose File:** Put your Docker Compose definition file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
2.  **Configure Environment:** Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive guided setup, or `m3tal config set KEY value` for direct assignment.
3.  **Start All Stacks:** Run `m3tal up` to initialize and start your newly added stack along with all other M3TAL-managed services.

### Traefik Routing Architecture

Traefik, deployed via `routing-compose.yml`, acts as the ingress controller for your M3TAL services.

*   **HTTP Entry Point:** It binds to host port 80, serving as the primary HTTP entry point for all external traffic.
*   **Service Discovery:** Traefik automatically discovers services by reading Docker labels applied to your containers.
*   **Dynamic Configuration:** It utilizes a file provider to load dynamic routing configurations from `/docker/dynamic/`. These configuration files are hot-reloaded, meaning changes are applied without restarting Traefik.
*   **M3TAL API Routing:** The API daemon (running on `http://localhost:8080`) is exposed via Traefik. The `/docker/dynamic/api.yml` file configures Traefik to route `api.YOUR_DOMAIN` (e.g., `api.myhomelab.com`) to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, specific Traefik labels are added to the M3TAL Dashboard container, instructing Traefik to route `dash.YOUR_DOMAIN` (e.g., `dash.myhomelab.com`) to the dashboard container's internal port 8082.

### Port Map

Understanding which services listen on which ports is crucial for troubleshooting and network configuration:

| Port | Service                            | Access                                                                                                 |
| :--- | :--------------------------------- | :----------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point           | **Public (Traefik mode)**: Exposed to the internet (if configured) or local network for domain-based access. |
| 8080 | M3TAL API daemon (Go)              | **Host-local**: Accessible only from the M3TAL host or other Docker containers via `host.docker.internal`. |
| 8081 | Traefik dashboard (internal)       | **Host-local only**: Traefik's own dashboard for monitoring, usually exposed via a separate Traefik rule.  |
| 8082 | M3TAL Dashboard (Python/Flask)     | **Direct port (Local mode)**: Exposed directly on the host. <br> **Via Traefik (Traefik mode)**: Routed internally by Traefik. |

---

## M3TAL CLI Commands Reference

Here is a comprehensive list of all M3TAL CLI commands, complete with their purpose and practical usage examples.

### Core Application & UI

#### `m3tal` or `m3tal ui`
Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a graphical interface is available.
*   **Usage:**
    ```bash
    m3tal
    # OR
    m3tal ui
    ```

#### `m3tal headless [port]`
Starts M3TAL in headless mode. This is ideal for server environments (like home servers or remote VPS) where a graphical desktop is not present or desired. The M3TAL API daemon remains active, allowing management via the CLI or the web dashboard.
*   **Arguments:**
    *   `[port]`: Optional. Specifies the port the headless API daemon should listen on. Defaults to 8080 if not provided.
*   **Usage:**
    ```bash
    # Start headless mode on the default port (8080)
    m3tal headless

    # Start headless mode on a custom port, e.g., 9000
    m3tal headless 9000
    ```

### System Diagnostics & Health

#### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Provides a quick snapshot of your system's performance and health.
*   **Usage:**
    ```bash
    m3tal stats
    ```
    *(Example Output Snippet - actual output will vary)*
    ```
    CPU:  15% (8 cores)
    RAM:  4.2 GB / 16 GB (26%)
    GPU:  NVIDIA RTX 3070 - 45°C, 10% Usage, 2.1 GB VRAM
    Disk: /dev/sda1 - 120 GB / 500 GB (24%)
    Temp: CPU: 55°C, System: 48°C
    ```

#### `m3tal doctor`
Performs a pre-flight health check of your M3TAL installation. This command verifies critical dependencies and configurations, including Docker connectivity, `/etc/m3tal/.env` validity, and port availability. Essential for troubleshooting initial setup or unexpected issues.
*   **Usage:**
    ```bash
    m3tal doctor
    ```
    *(Example Output Snippet - actual output will vary)*
    ```
    [✓] Docker Engine detected and running.
    [✓] Docker Compose V2 detected.
    [✓] /etc/m3tal/.env file exists and is valid.
    [✓] Port 8080 (M3TAL API) is available.
    [✗] Port 80 (Traefik HTTP) is in use by another process.
        -> Recommendation: Investigate PID 1234 or change Traefik port.
    [✓] All required M3TAL directories are present.
    ```

### Configuration Management

M3TAL's primary configuration lives in `/etc/m3tal/.env`. These commands provide robust tools for managing this file.

#### `m3tal config wizard`
Launches an interactive wizard to configure or update the `/etc/m3tal/.env` file. This is the recommended method for initial setup or making significant configuration changes, as it guides you through available options.
*   **Usage:**
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`
Sets or updates a single environment variable within the `/etc/m3tal/.env` file. This command is useful for quick, precise configuration adjustments.
*   **Arguments:**
    *   `KEY`: The name of the environment variable (e.g., `DOMAIN`, `DASHBOARD_PORT`).
    *   `VALUE`: The value to assign to the variable (e.g., `myhomelab.com`, `8082`).
*   **Usage:**
    ```bash
    # Set the primary domain for Traefik
    m3tal config set DOMAIN myhomelab.com

    # Switch the dashboard to Traefik mode
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`
Reads and prints the value of a single environment variable from `/etc/m3tal/.env` to the terminal.
*   **Arguments:**
    *   `KEY`: The name of the environment variable.
*   **Usage:**
    ```bash
    # Get the currently configured domain
    m3tal config get DOMAIN

    # Output: myhomelab.com
    ```

#### `m3tal config scan`
Scans and lists all unique environment variables detected across all M3TAL-managed Docker Compose stacks (i.e., `*-compose.yml` files in `/docker/`) and their default values. This helps identify all potential configuration options.
*   **Usage:**
    ```bash
    m3tal config scan
    ```
    *(Example Output Snippet - actual output will vary)*
    ```
    Detected Environment Variables:
    - DOMAIN (default: example.com)
    - DASHBOARD_EXPOSE_MODE (default: local)
    - DASHBOARD_PORT (default: 8082)
    - API_PORT (default: 5050)
    - API_TOKEN (default: <empty>)
    - CLOUDFLARED_TUNNEL_ID (default: <empty>)
    - TZ (default: UTC)
    ```

#### `m3tal config list`
Prints the entire contents of the `/etc/m3tal/.env` file to the terminal. This provides a complete overview of your current M3TAL configuration.
*   **Usage:**
    ```bash
    m3tal config list
    ```
    *(Example Output Snippet - actual output will vary)*
    ```bash
    # M3TAL Ecosystem Configuration
    # Managed by m3tal config wizard

    DOMAIN=myhomelab.com
    DASHBOARD_EXPOSE_MODE=traefik
    DASHBOARD_PORT=8082
    API_PORT=8080
    API_TOKEN=your_secret_api_token
    M3TAL_STATE_DB=/var/lib/m3tal/state.db
    TZ=America/New_York
    ```

### Dashboard Management

These commands are specifically for managing the M3TAL Dashboard container and its credentials.

#### `m3tal dashpass [username] [password]`
Updates the M3TAL Dashboard user password. This command manages entries in `/docker/users.json`. If `username` and `password` are omitted, the command will launch an interactive prompt.
*   **Arguments:**
    *   `[username]`: The dashboard username (e.g., `admin`).
    *   `[password]`: The new password for the user.
*   **Usage:**
    ```bash
    # Update the 'admin' user's password directly
    m3tal dashpass admin newSecureP@ssword!23

    # Interactively set or reset a password
    m3tal dashpass
    ```

#### `m3tal dash up`
Pulls the latest dashboard Docker Compose configuration from GitHub, then starts the M3TAL Dashboard container with the appropriate `DASHBOARD_EXPOSE_MODE` override. This ensures your dashboard is always running the most recent version.
*   **Usage:**
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`
Stops and removes the M3TAL Dashboard container. This does not remove persistent data.
*   **Usage:**
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`
Restarts the M3TAL Dashboard container. Useful for applying new configurations or resolving minor issues.
*   **Usage:**
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`
Streams aggregated logs from the M3TAL Dashboard container to the terminal. Essential for debugging dashboard-specific issues.
*   **Usage:**
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`
Shows the current status of the M3TAL Dashboard container (e.g., `running`, `exited`, `restarting`).
*   **Usage:**
    ```bash
    m3tal dash status
    ```

### Docker Compose Stack Management

These commands are used to manage all Docker Compose stacks defined in the `/docker/` directory.

#### `m3tal up`
Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts (or recreates if necessary) all your M3TAL services and any custom user stacks in detached mode.
*   **Usage:**
    ```bash
    m3tal up
    ```

#### `m3tal down`
Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes defined by your M3TAL stacks.
*   **Usage:**
    ```bash
    m3tal down
    ```

#### `m3tal logs`
Streams aggregated logs from all running Docker containers managed by M3TAL (i.e., all containers across all stacks defined in `/docker/`). This provides a consolidated view of your entire M3TAL ecosystem's activity.
*   **Usage:**
    ```bash
    m3tal logs
    ```

---

## M3TAL API Daemon Service Management (systemd)

The M3TAL API daemon (`m3tal-api`) runs as a systemd service. You can manage its lifecycle using standard `systemctl` commands.

*   **Check Status:** To verify if the API daemon is running and check its current status:
    ```bash
    sudo systemctl status m3tal-api
    ```

*   **Restart Service:** To restart the API daemon (e.g., after manual changes to `/etc/m3tal/.env` that require the API to pick them up):
    ```bash
    sudo systemctl restart m3tal-api
    ```

*   **View Logs:** To stream live logs from the API daemon, which is invaluable for debugging issues with the M3TAL API itself or its interaction with Docker:
    ```bash
    sudo journalctl -u m3tal-api -f
    ```

*   **Start Service:** If the service is stopped:
    ```bash
    sudo systemctl start m3tal-api
    ```

*   **Stop Service:** To stop the API daemon:
    ```bash
    sudo systemctl stop m3tal-api
    ```

---

## Direct Docker Compose Commands (Fallback)

While the `m3tal` CLI simplifies most operations, it's useful to know the underlying `docker compose` commands for advanced debugging or direct interaction. M3TAL relies on **Docker Engine** and **Docker Compose V2**.

All M3TAL-managed Docker Compose files (`*-compose.yml`) reside in the `/docker/` directory.

*   **Manage all stacks in `/docker/` directly:**
    ```bash
    # Start all services defined in /docker/ in detached mode
    docker compose --project-directory /docker/ up -d

    # Stop all services defined in /docker/
    docker compose --project-directory /docker/ down

    # View logs from all services in /docker/
    docker compose --project-directory /docker/ logs -f

    # List all services and their status in /docker/
    docker compose --project-directory /docker/ ps
    ```

*   **Manage the M3TAL Dashboard specifically (manual approach):**
    The dashboard compose files are: `m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`. You would typically use `m3tal dash up`, but for direct control:

    ```bash
    # Example: Start dashboard in local mode, ensuring you have the latest files
    # First, manually ensure m3tal-compose.yml and m3tal-compose.local.yml are present in /docker/
    # (m3tal dash up handles this automatically)
    docker compose --project-directory /docker/ -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d

    # Example: Start dashboard in traefik mode
    docker compose --project-directory /docker/ -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d

    # Stop the dashboard (regardless of mode, assuming it's the only service in m3tal-compose.yml)
    # Note: A simple `docker compose down` will stop everything in /docker/
    docker compose --project-directory /docker/ -f /docker/m3tal-compose.yml down
    ```
    *Note: When using direct `docker compose` for the dashboard, you need to manually specify the base compose file and the correct override file (`.local.yml` or `.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.*

---

## Environment Variables

These are key environment variables that control the behavior of the M3TAL API daemon and its integrated services. Most are managed via `m3tal config wizard` or `m3tal config set`.

| Key                   | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Default Value |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| `DOMAIN`              | The primary domain used for Traefik routing. E.g., if set to `myhomelab.com`, Traefik will expose `dash.myhomelab.com` and `api.myhomelab.com`.                                                                                                                                                                                                                                                                                                                                                  | (empty)       |
| `DASHBOARD_EXPOSE_MODE` | Controls how the M3TAL Dashboard is exposed. Set to `local` for direct port access (`http://HOST_IP:8082`) or `traefik` for domain-based access via Traefik (`http://dash.DOMAIN`).                                                                                                                                                                                                                                                                                                            | `local`       |
| `DASHBOARD_PORT`      | The host port on which the M3TAL Dashboard will be directly exposed when `DASHBOARD_EXPOSE_MODE` is set to `local`.                                                                                                                                                                                                                                                                                                                                                                              | `8082`        |
| `API_PORT`            | The port on which the M3TAL API daemon (Go binary) listens for incoming requests. This is an internal host-local port, typically accessed by the M3TAL CLI, dashboard, or other local services.                                                                                                                                                                                                                                                                                                   | `5050`        |
| `API_TOKEN`           | An optional authentication token for securing the M3TAL API. If set, clients (like the M3TAL Dashboard or CLI plugins) will need to include this token in their requests for authentication. **Strongly recommended for production environments.**                                                                                                                                                                                                                                               | (empty)       |
| `M3TAL_STATE_DB`      | Specifies the full path to the SQLite database file used by the M3TAL API daemon to store its state and configuration.                                                                                                                                                                                                                                                                                                                                                                          | `/var/lib/m3tal/state.db` |
| `M3TAL_PLUGINS_DIR`   | Defines the directory where M3TAL API daemon plugins are located. These plugins extend the functionality of the M3TAL ecosystem.                                                                                                                                                                                                                                                                                                                                                              | (empty)       |
| `TZ`                  | Sets the timezone for containers. This affects logging timestamps and time-sensitive operations within your Docker services. E.g., `America/New_York`, `Europe/London`.                                                                                                                                                                                                                                                                                                                         | `UTC`         |
| `CLOUDFLARED_TUNNEL_ID` | The ID of your Cloudflare Tunnel. Required if you are using Cloudflared via `routing-compose.yml` for secure external access to your services. You get this ID from the Cloudflare Zero Trust dashboard.                                                                                                                                                                                                                                                                                        | (empty)       |
| `CLOUDFLARED_TUNNEL_TOKEN` | The token for your Cloudflare Tunnel. Required if you are using Cloudflared to authenticate your tunnel. This is typically used with named tunnels.                                                                                                                                                                                                                                                                                                                                              | (empty)       |

---

## APT Installation

To install the M3TAL CLI and associated components on Debian/Ubuntu-based systems, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```