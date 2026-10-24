# `docs/COMMAND_REFERENCE.md` — M3TAL CLI Cheat Sheet

Greetings, Architect. As DocSmith, the M3TAL Ecosystem Documentation Architect, I've compiled this comprehensive CLI cheat sheet to guide you through every command of the `m3tal` binary. M3TAL provides a unified entry point for managing your home server, orchestrating Docker containers, configuring services, and monitoring your system's health.

---

## 1. M3TAL Core Concepts

Before diving into the commands, let's establish a foundational understanding of the M3TAL system architecture and its key components.

### 1.1. M3TAL System Architecture

The M3TAL ecosystem is composed of several tightly integrated components:

-   **CLI binary (`/usr/bin/m3tal`)**: Your primary interface. A single Go binary, installed via APT, acting as the orchestrator for all M3TAL operations.
-   **API daemon (`m3tal-api.service`)**: A Go binary running as a systemd service, listening on port `8080`. It manages Docker interactions, maintains the SQLite state database (`/var/lib/m3tal/state.db`), and exposes API routes for the Dashboard and other tools.
-   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask container that provides a web-based user interface. It communicates with the local API daemon at `http://host.docker.internal:8080`.
-   **Traefik gateway (`routing-compose.yml`)**: A Docker container acting as a reverse proxy. It exposes M3TAL services (and your custom stacks) by domain name on port 80 (and 443, if configured). It uses a file provider (`/docker/dynamic/`) for dynamic routing configuration.
-   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, enabling secure, zero-configuration internet access to your services without exposing ports directly.

### 1.2. Filesystem Contract

M3TAL relies on a specific filesystem layout for its operation and configuration. Understanding these paths is crucial for advanced management:

| Path                        | Purpose                                                                 |
| :-------------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary configuration file**. Stores all M3TAL environment variables. Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the `m3tal-api` daemon. |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains M3TAL's core Docker Compose files and Traefik dynamic configuration. |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path where you place all your Docker Compose files for M3TAL to manage. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                   |
| `/docker/dynamic/`          | Directory for Traefik's dynamic file provider configuration.              |

### 1.3. Dashboard Access Modes

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

-   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
-   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the `m3tal-dashboard` container.
-   **Access**:
    -   Via `http://HOST_IP:8082` (e.g., `http://192.168.1.100:8082`)
    -   Via `http://localhost:8082` (if accessing directly from the host machine)
-   **Requirements**: No Traefik required. Works immediately after installation on a home server.
-   **Best for**: LAN-only setups, first-time users, local development, or when Traefik is not desired.

#### Mode 2: `traefik`

-   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
-   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the `m3tal-dashboard` container. Traefik (running via `routing-compose.yml`) then routes requests for `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
-   **Access**: Via `http://dash.${DOMAIN}` (e.g., if `DOMAIN=example.com`, access `http://dash.example.com`).
-   **Requirements**: Traefik must be running (`m3tal up` will start `routing-compose.yml`). The `DOMAIN` environment variable in `/etc/m3tal/.env` must be correctly configured.
-   **Best for**: Domain-based access, setups with multiple services behind a reverse proxy, and integration with Cloudflared.

### 1.4. Docker / Compose Runtime

M3TAL deeply integrates with **Docker Engine** and **Docker Compose V2**. These are hard dependencies for the system's operation.

-   The `m3tal` CLI acts as a wrapper around `docker compose` commands.
-   The `m3tal up` command executes `docker compose up -d` across *all* `*-compose.yml` files found in the `/docker/` directory (which is symlinked to `/opt/m3tal/stack/`).
-   The `m3tal dash up` command has a specialized function:
    1.  It first pulls the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files from GitHub into `/docker/`.
    2.  It then reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Finally, it starts the dashboard using the appropriate compose override file, applying the correct port binding or Traefik labels.
-   To integrate your own applications, simply place your `*-compose.yml` files in `/docker/`. M3TAL will automatically pick them up when you run `m3tal up`.

### 1.5. Deployment Lifecycle — Day 2 Operations

Installing a new Docker stack with M3TAL is straightforward:

1.  **Place your compose file**: Copy your `docker-compose.yml` file into the `/docker/` directory, for example: `/docker/my-app-compose.yml`.
2.  **Configure environment variables**: If your stack requires specific environment variables, define them in `/etc/m3tal/.env`. You can use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for individual variables.
3.  **Start all stacks**: Run `m3tal up` to bring up your new stack along with all other M3TAL services.

### 1.6. Traefik Routing Architecture

Traefik, deployed as a container via `routing-compose.yml`, is the central HTTP entry point for your services:

-   It binds to port `80` (and `443` for HTTPS) on the host machine.
-   It automatically discovers services by reading Docker labels on running containers (e.g., `m3tal-compose.traefik.yml` uses labels to expose the dashboard).
-   It loads dynamic routing configurations from `/docker/dynamic/` using a file provider. These configurations can be hot-reloaded without restarting Traefik.
-   **API Routing**: Traefik routes `api.${DOMAIN}` to the M3TAL API daemon (`http://host.docker.internal:8080`) using `dynamic/api.yml`.
-   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` to the `m3tal-dashboard` container based on its labels.

### 1.7. Port Map

Understanding the port mapping is crucial for network configuration and troubleshooting:

| Port | Service                          | Access                                                                  |
| :--- | :------------------------------- | :---------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point         | Public (when `DASHBOARD_EXPOSE_MODE=traefik` or for other services)     |
| 8080 | M3TAL API daemon (Go)            | Host-local only (internal communication, Traefik routes to it)          |
| 8081 | Traefik dashboard (admin UI)     | Host-local only (e.g., `http://localhost:8081` on the host)             |
| 8082 | M3TAL Dashboard (UI container)   | Direct port (`local` mode) or via Traefik (`traefik` mode)              |

---

## 2. M3TAL CLI Command Reference

This section details every command available through the `m3tal` binary, complete with descriptions and real usage examples.

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a GUI is available.

```bash
# Launch the M3TAL desktop application
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode. This is ideal for home servers or environments without a graphical interface. It starts the necessary background processes and potentially the API daemon directly, or ensures it's running. If a `port` is specified, it might override the default API port.

```bash
# Start M3TAL in headless mode, listening on the default API port (8080)
m3tal headless

# Start M3TAL in headless mode, with the API daemon listening on port 5000
m3tal headless 5000
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to the terminal. Provides a quick snapshot of your server's health and performance.

```bash
# Get an instant readout of system statistics
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check on your M3TAL installation. This command verifies critical dependencies and configurations:

-   Docker connectivity and status.
-   Validity of environment variables in `/etc/m3tal/.env`.
-   Availability of essential ports (e.g., 80, 8080, 8082).
-   Status of the `m3tal-api` systemd service.

```bash
# Run a comprehensive health check
m3tal doctor
```

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL environment or modify key variables.

```bash
# Start the interactive configuration wizard
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets or updates a single environment variable in `/etc/m3tal/.env`. This command is useful for quick, non-interactive changes.

```bash
# Set the primary domain for Traefik routing
m3tal config set DOMAIN myhomeserver.com

# Change the dashboard expose mode to use Traefik
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

```bash
# Get the currently configured domain
m3tal config get DOMAIN

# Check the dashboard exposure mode
m3tal config get DASHBOARD_EXPOSE_MODE
```

### `m3tal config scan`

Lists all known environment variables across all configured stacks (M3TAL's internal variables and those defined in your `*-compose.yml` files). This helps in understanding what variables are available and their default values.

```bash
# Scan and list all environment variables
m3tal config scan
```

### `m3tal config list`

Prints the current contents of the primary M3TAL environment file, `/etc/m3tal/.env`, to the terminal.

```bash
# Display the contents of the .env file
m3tal config list
```

### `m3tal dashpass [username] [password]`

Updates the password for a dashboard user. If `username` and `password` arguments are omitted, the command will launch an interactive prompt. The credentials are stored in `/docker/users.json`.

```bash
# Interactively set a new password for the 'admin' user
m3tal dashpass

# Set the password for user 'admin' non-interactively
m3tal dashpass admin MyNewSecurePassword123!
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub and then starts or updates the `m3tal-dashboard` container. This command ensures you're running the most recent dashboard version with the correct `DASHBOARD_EXPOSE_MODE` override.

```bash
# Ensure the dashboard is running and up-to-date
m3tal dash up
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

```bash
# Stop the M3TAL dashboard
m3tal dash down
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.

```bash
# Restart the M3TAL dashboard container
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the `m3tal-dashboard` container to your terminal. Essential for debugging dashboard-related issues.

```bash
# Stream real-time logs from the M3TAL dashboard
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., `running`, `exited`, `restarting`).

```bash
# Check the operational status of the M3TAL dashboard
m3tal dash status
```

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts all your M3TAL services and any custom stacks you've deployed.

```bash
# Start all M3TAL services and user-defined stacks in the background
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all stacks defined by `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes associated with M3TAL and your custom stacks.

```bash
# Stop and remove all M3TAL services and user-defined stacks
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of activity across your entire Docker ecosystem.

```bash
# Stream combined logs from all active M3TAL containers
m3tal logs
```

---

## 3. Advanced Topics

### 3.1. Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage this critical component directly using `systemctl` and view its logs with `journalctl`.

-   **Check the status of the API daemon**:
    ```bash
    systemctl status m3tal-api
    ```
-   **Restart the API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
-   **Stop the API daemon**:
    ```bash
    sudo systemctl stop m3tal-api
    ```
-   **Start the API daemon**:
    ```bash
    sudo systemctl start m3tal-api
    ```
-   **Stream real-time logs from the API daemon**:
    ```bash
    journalctl -u m3tal-api -f
    ```

### 3.2. Direct Docker & Docker Compose Commands

While `m3tal` provides a convenient wrapper, you can always interact with Docker and Docker Compose directly. This is useful for advanced debugging, inspecting specific containers, or executing commands not directly exposed by the `m3tal` CLI.

Remember that M3TAL primarily manages compose files located in `/docker/`.

-   **List all Docker containers**:
    ```bash
    docker ps -a
    ```
-   **Inspect a specific container (e.g., the dashboard)**:
    ```bash
    docker inspect m3tal-dashboard
    ```
-   **View logs for a specific container (e.g., Traefik)**:
    ```bash
    docker logs -f traefik
    ```
-   **Manually start a specific M3TAL stack (e.g., the dashboard in local mode)**:
    *(Note: This bypasses `m3tal dash up`'s GitHub pull logic)*
    ```bash
    # Ensure you are in the /docker/ directory or specify full paths
    sudo docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
    ```
-   **Manually start the routing stack (Traefik and Cloudflared)**:
    ```bash
    sudo docker compose -f /docker/routing-compose.yml up -d
    ```
-   **Stop a specific stack**:
    ```bash
    # Example for stopping the dashboard stack (using the base compose file)
    sudo docker compose -f /docker/m3tal-compose.yml down
    ```
-   **Build a service image locally (if modifying source)**:
    ```bash
    # Assuming you have a Dockerfile in /docker/my-service/
    sudo docker compose -f /docker/my-service-compose.yml build my-service-name
    ```

---

## 4. Installation

To install the `m3tal` CLI binary and its dependencies on your Debian/Ubuntu-based system, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```