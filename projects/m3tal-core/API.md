```markdown
# M3TAL CLI Command Reference

Welcome, M3TAL operator. I am DocSmith, your M3TAL Ecosystem Documentation Architect. This document serves as your definitive guide to the `m3tal` command-line interface, providing a complete cheat-sheet for managing your M3TAL instance.

The M3TAL CLI is a unified Go binary (`/usr/bin/m3tal`) designed to be the single entry point for all operations, interacting seamlessly with the M3TAL API daemon and your Docker infrastructure.

---

## ⚙️ Core Concepts & Architecture

Before diving into commands, let's establish the foundational elements of your M3TAL system.

### M3TAL System Architecture
Your M3TAL instance orchestrates several key components:

*   **CLI binary (`/usr/bin/m3tal`):** The tool you're currently learning to use.
*   **API daemon (`m3tal-api.service`):** A Go binary running as a `systemd` service on `http://localhost:8080`. It manages Docker, maintains the SQLite state database, and exposes API routes. The CLI communicates with this daemon.
*   **Dashboard container (`m3tal-dashboard`):** A Python/Flask web application running inside Docker, communicating with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway (`routing-compose.yml`):** A reverse proxy container that exposes services (like the dashboard or API) via domain names on port 80. It uses Docker labels and a file provider for dynamic routing.
*   **Cloudflared (`routing-compose.yml`):** An optional Cloudflare tunnel container for secure, zero-config internet access to your services.

### 🗂️ Filesystem Contract

Understanding where M3TAL stores its critical files is paramount:

| Path                        | Purpose                                                                                | Managed By                                    |
| :-------------------------- | :------------------------------------------------------------------------------------- | :-------------------------------------------- |
| `/etc/m3tal/.env`           | Primary system configuration file. **Crucial for all operations.**                     | `m3tal config wizard`, `m3tal config set`     |
| `/var/lib/m3tal/state.db`   | SQLite state database for the API daemon.                                              | M3TAL API daemon (auto-created)               |
| `/opt/m3tal/stack/`         | Canonical directory for Docker Compose files and Traefik config.                       | M3TAL installation, user-managed for stacks |
| `/docker`                   | **User-facing symlink** to `/opt/m3tal/stack/`. Use this path for all stack operations. | M3TAL installation                            |
| `/docker/users.json`        | Credential store for the M3TAL Dashboard.                                              | `m3tal dashpass`                              |
| `/docker/dynamic/`          | Directory for user-defined Traefik dynamic configuration files.                        | User-managed                                  |

### 🌐 Dashboard Access Modes

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration:** Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** M3TAL uses the `m3tal-compose.local.yml` override, which directly binds the dashboard container's port to the host machine.
*   **Access:** Via `http://YOUR_HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik required. Works immediately after `m3tal dash up`.
*   **Use Case:** Ideal for initial setup, local testing, or LAN-only deployments on a home server.

#### Mode 2: `traefik`

*   **Configuration:** Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** M3TAL uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik (running via `m3tal up`) then routes requests for `dash.${DOMAIN}` to the dashboard.
*   **Access:** Via `http://dash.YOUR_DOMAIN` (e.g., `http://dash.myhome.com`).
*   **Requirements:** Traefik must be running (`m3tal up`). Your `DOMAIN` environment variable must be correctly configured in `/etc/m3tal/.env`.
*   **Use Case:** Best for domain-based access, integrating into existing reverse proxy setups, and exposing services behind a single entry point.

### 🐳 Docker & Docker Compose Runtime

M3TAL relies heavily on **Docker Engine** and **Docker Compose V2** as hard dependencies.

*   The `m3tal` CLI abstracts away direct `docker compose` commands for most operations.
*   `m3tal up` executes `docker compose up -d` for **all** `*-compose.yml` files found in the `/docker/` directory.
*   `m3tal dash up` specifically manages the M3TAL dashboard container. It downloads the latest dashboard compose configurations (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) and starts the dashboard using the appropriate override based on `DASHBOARD_EXPOSE_MODE`.
*   User-defined Docker Compose stacks should be placed as `*-compose.yml` files directly into `/docker/`.

### 🚀 Deployment Lifecycle: Installing New Stacks

1.  **Place Compose File:** Copy your new Docker Compose stack file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
2.  **Configure Environment:** Ensure all necessary environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value`.
3.  **Start All Stacks:** Run `m3tal up` to bring up your new stack along with all other configured services.

### 🚦 Traefik Routing Architecture

Traefik (deployed by `routing-compose.yml`) acts as your system's intelligent reverse proxy:

*   **Entry Point:** Binds host port 80 (HTTP) to serve as the primary entry point for web traffic.
*   **Service Discovery:** Automatically discovers services declared in Docker Compose files via their Docker labels.
*   **Dynamic Configuration:** Loads additional routing rules from `/docker/dynamic/` (using a file provider with hot-reload).
    *   Example: `api.DOMAIN` is routed to the M3TAL API daemon (`http://host.docker.internal:8080`) via a file in `/docker/dynamic/api.yml`.
    *   Example: `dash.DOMAIN` is routed to the dashboard container via Traefik labels within `m3tal-compose.traefik.yml` (when `DASHBOARD_EXPOSE_MODE=traefik`).

### 🔌 Port Map

| Port | Service                                | Access                 | Notes                                            |
| :--- | :------------------------------------- | :--------------------- | :----------------------------------------------- |
| 80   | Traefik HTTP Entry Point               | Public (Traefik mode)  | Primary web access                               |
| 8080 | M3TAL API Daemon (Go binary)           | Host-local             | Listens on `localhost:8080`.                     |
| 8081 | Traefik Dashboard                      | Host-local only        | Access via `http://127.0.0.1:8081`.              |
| 8082 | M3TAL Dashboard (container)            | Direct or via Traefik  | `http://HOST_IP:8082` (`local` mode) or `http://dash.DOMAIN` (`traefik` mode) |

---

## 🎯 `m3tal` CLI Commands

Here's the comprehensive list of `m3tal` commands, each with a practical usage example.

### 🖥️ Desktop & Headless Operations

*   **`m3tal`**
    Launches the M3TAL Linux Desktop Application. This command starts the graphical user interface.
    ```bash
    m3tal
    ```

*   **`m3tal ui`**
    An alias for `m3tal`, also launches the M3TAL Linux Desktop Application.
    ```bash
    m3tal ui
    ```

*   **`m3tal headless [port]`**
    Starts M3TAL in headless mode, ideal for home servers or environments without a graphical display. This typically refers to the API daemon (though the API is managed by systemd, this command might be a legacy or specialized runner for a different component). If `port` is omitted, it defaults.
    ```bash
    m3tal headless 8080
    ```
    *Note: The primary M3TAL API daemon is managed as a systemd service. This command may serve as an alternative or debug entry point.*

### 🩺 System Health & Information

*   **`m3tal stats`**
    Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Useful for quick health checks.
    ```bash
    m3tal stats
    ```

*   **`m3tal doctor`**
    Performs a pre-flight health check of your M3TAL environment. This includes verifying Docker connectivity, validating `.env` configuration, and checking for port availability.
    ```bash
    m3tal doctor
    ```
    *Example output snippet:*
    ```
    🩺 M3TAL Doctor: Pre-flight Health Check

    [✓] Docker Daemon Connected
    [✓] /etc/m3tal/.env file exists
    [✓] Required ENV vars present (API_PORT, DASHBOARD_PORT, DOMAIN)
    [✓] Port 8080 (M3TAL API) is available
    [✓] Port 8082 (M3TAL Dashboard) is available
    [✓] Docker network 'proxy' exists
    ```

### ⚙️ Configuration Management (`/etc/m3tal/.env`)

These commands manage the primary configuration file, `/etc/m3tal/.env`.

*   **`m3tal config wizard`**
    Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended way to manage your M3TAL environment variables.
    ```bash
    m3tal config wizard
    ```

*   **`m3tal config set KEY VALUE`**
    Sets a single environment variable in `/etc/m3tal/.env`. Overwrites existing values or adds new ones.
    ```bash
    m3tal config set DOMAIN myhome.lan
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    m3tal config set PUID 1001
    ```

*   **`m3tal config get KEY`**
    Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.
    ```bash
    m3tal config get DOMAIN
    # Expected output: myhome.lan
    ```

*   **`m3tal config scan`**
    Lists all environment variables that M3TAL is aware of, across all stacks, including their default values. This is useful for discovering available configuration options.
    ```bash
    m3tal config scan
    ```

*   **`m3tal config list`**
    Prints the current contents of your `/etc/m3tal/.env` file to the terminal.
    ```bash
    m3tal config list
    ```

### 📊 Dashboard Management (`m3tal-dashboard` container)

These commands specifically manage the M3TAL Dashboard container.

*   **`m3tal dashpass [username] [password]`**
    Updates the password for a dashboard user. If `username` and `password` are omitted, an interactive prompt will guide you. Credentials are stored in `/docker/users.json`.
    ```bash
    # Interactive mode:
    m3tal dashpass

    # Direct mode:
    m3tal dashpass admin "MyStrongPa$$w0rd"
    ```

*   **`m3tal dash up`**
    Pulls the latest dashboard Docker Compose configuration files from GitHub (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) and then starts the dashboard container with the appropriate override based on `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    ```bash
    m3tal dash up
    ```

*   **`m3tal dash down`**
    Stops and removes the M3TAL dashboard container.
    ```bash
    m3tal dash down
    ```

*   **`m3tal dash restart`**
    Restarts the M3TAL dashboard container.
    ```bash
    m3tal dash restart
    ```

*   **`m3tal dash logs`**
    Streams real-time logs from the M3TAL dashboard container. Press `Ctrl+C` to exit.
    ```bash
    m3tal dash logs
    ```

*   **`m3tal dash status`**
    Shows the current status of the M3TAL dashboard container (e.g., `running`, `exited`).
    ```bash
    m3tal dash status
    ```

### 📦 Stack Management (All Docker Compose stacks)

These commands manage all Docker Compose stacks defined by `*-compose.yml` files in the `/docker/` directory.

*   **`m3tal up`**
    Runs `docker compose up -d` across **all** `*-compose.yml` files found in `/docker/`. This command starts or recreates all services defined in your M3TAL ecosystem.
    ```bash
    m3tal up
    ```

*   **`m3tal down`**
    Runs `docker compose down` across **all** `*-compose.yml` files in `/docker/`. This command stops and removes all services and their associated networks defined in your M3TAL ecosystem.
    ```bash
    m3tal down
    ```

*   **`m3tal logs`**
    Streams aggregated logs from all currently running Docker Compose stacks. Useful for monitoring the entire ecosystem. Press `Ctrl+C` to exit.
    ```bash
    m3tal logs
    ```

---

## 🛠️ Systemd Service Management

The M3TAL API daemon (`m3tal-api`) runs as a `systemd` service, ensuring it starts automatically on boot and can be managed independently of the CLI.

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```
    *Expected output snippet:*
    ```
    ● m3tal-api.service - M3TAL API Daemon
         Loaded: loaded (/etc/systemd/system/m3tal-api.service; enabled; vendor preset: enabled)
         Active: active (running) since Tue 2023-10-26 10:30:00 UTC; 5min ago
       Main PID: 12345 (m3tal-api)
          Tasks: 7 (limit: 9270)
         Memory: 25.0M
            CPU: 120ms
         CGroup: /system.slice/m3tal-api.service
                 └─12345 /usr/bin/m3tal-api
    ```

*   **Stream API service logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
    This command displays real-time logs from the M3TAL API daemon. Press `Ctrl+C` to exit.

*   **Restart API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

*   **Enable API service (start on boot):**
    ```bash
    sudo systemctl enable m3tal-api
    ```

*   **Disable API service (do not start on boot):**
    ```bash
    sudo systemctl disable m3tal-api
    ```

---

## 🐳 Direct Docker Compose Fallback

While the `m3tal` CLI abstracts most Docker Compose interactions, understanding the underlying commands can be useful for debugging or advanced scenarios. M3TAL leverages Docker Compose V2.

The `m3tal` CLI essentially executes `docker compose` commands against the files in `/docker/`.

*   **View all Compose files M3TAL manages:**
    ```bash
    ls -l /docker/*.yml
    ```

*   **Start a specific stack (e.g., routing):**
    ```bash
    docker compose -f /docker/routing-compose.yml up -d
    ```

*   **Stop a specific stack (e.g., m3tal-dashboard):**
    ```bash
    # First, identify which compose file defines the dashboard based on your DASHBOARD_EXPOSE_MODE
    # If DASHBOARD_EXPOSE_MODE=local:
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down

    # If DASHBOARD_EXPOSE_MODE=traefik:
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml down
    ```
    *Note: `m3tal dash down` handles this complexity for you.*

*   **View logs for all Docker services directly:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-app-compose.yml logs -f
    ```
    *Note: `m3tal logs` provides an aggregated view more simply.*

*   **Build/Rebuild all images for all services:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml build
    ```

*   **Prune stopped containers and unused volumes (use with caution!):**
    ```bash
    docker system prune -a --volumes
    ```

---

## 🚀 APT Installation

For a fresh M3TAL installation or to ensure your system is ready:

1.  **Add the GPG signing key:**
    ```bash
    curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
    ```

2.  **Add the APT repository:**
    ```bash
    echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list
    ```

3.  **Install:**
    ```bash
    sudo apt update && sudo apt install -y m3tal
    ```

---

This concludes your `m3tal` CLI cheat-sheet. May your M3TAL ecosystem run flawlessly.
```