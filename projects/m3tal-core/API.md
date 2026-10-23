# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I've compiled this comprehensive guide to navigating and controlling your M3TAL environment using the command-line interface. This cheat-sheet covers every `m3tal` command, providing practical examples and detailing the underlying architecture.

---

## 🚀 Installation

The `m3tal` CLI binary is distributed via an APT repository, ensuring easy installation and updates on Debian/Ubuntu-based systems.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 📖 Core M3TAL Commands

These commands provide fundamental interactions with the M3TAL ecosystem, from launching the UI to performing system diagnostics.

### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application, providing a graphical user interface for managing your stacks and services.
*   **Usage Example:**
    ```bash
    m3tal
    # Or explicitly:
    m3tal ui
    ```

### `m3tal headless [port]`
Starts the M3TAL API daemon in headless mode, ideal for server environments where a graphical interface is not desired. This is the same Go binary that runs as the `m3tal-api.service` but launched directly. If no port is specified, it defaults to the `API_PORT` defined in `/etc/m3tal/.env` (default 8080).
*   **Usage Example:**
    ```bash
    m3tal headless
    # To start the API on a specific port, e.g., 9000:
    m3tal headless 9000
    ```

### `m3tal stats`
Prints instant host system metrics directly to your terminal, including CPU usage, GPU status (if available), thermal readings, and storage utilization.
*   **Usage Example:**
    ```bash
    m3tal stats
    ```

### `m3tal doctor`
Performs a pre-flight health check of your M3TAL environment. This diagnostic tool verifies Docker connectivity, validates the `/etc/m3tal/.env` configuration, and checks for port availability, helping you troubleshoot common issues.
*   **Usage Example:**
    ```bash
    m3tal doctor
    ```

---

## ⚙️ Configuration Management (`m3tal config`)

The `m3tal config` commands are essential for managing the `/etc/m3tal/.env` primary configuration file, which dictates the behavior of your M3TAL system and its deployed services.

### `m3tal config wizard`
Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended method for initial setup and major configuration changes.
*   **Usage Example:**
    ```bash
    m3tal config wizard
    ```

### `m3tal config set KEY VALUE`
Sets a single environment variable within the `/etc/m3tal/.env` file. This command is useful for quick, targeted adjustments.
*   **Usage Example:**
    ```bash
    # Switch the dashboard to Traefik-managed access:
    m3tal config set DASHBOARD_EXPOSE_MODE traefik

    # Update the local domain:
    m3tal config set DOMAIN myhomelab.local
    ```

### `m3tal config get KEY`
Retrieves and displays the value of a specific environment variable from `/etc/m3tal/.env`.
*   **Usage Example:**
    ```bash
    # Check the current dashboard exposure mode:
    m3tal config get DASHBOARD_EXPOSE_MODE

    # Get the base storage path:
    m3tal config get BASE_STORAGE_PATH
    ```

### `m3tal config scan`
Lists all recognized environment variables across all M3TAL-managed stacks and services, along with their current values (if set in `.env`) and their default values. This provides a comprehensive overview of configurable parameters.
*   **Usage Example:**
    ```bash
    m3tal config scan
    ```

### `m3tal config list`
Displays the entire contents of the `/etc/m3tal/.env` file, showing all currently set environment variables.
*   **Usage Example:**
    ```bash
    m3tal config list
    ```

---

## 📊 Dashboard Management (`m3tal dash`)

These commands provide dedicated control over the `m3tal-dashboard` container, allowing you to manage its lifecycle and credentials independently.

### `m3tal dashpass [username] [password]`
Updates the password for a user accessing the M3TAL dashboard. If no arguments are provided, it initiates an interactive prompt for username and password. This command manages the `/docker/users.json` file.
*   **Usage Example:**
    ```bash
    # Set a new password for the 'admin' user:
    m3tal dashpass admin MyNewSecurePassword123!

    # Run interactively to update a password:
    m3tal dashpass
    ```

### `m3tal dash up`
Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the `m3tal-dashboard` container using the appropriate override based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.
*   **Usage Example:**
    ```bash
    m3tal dash up
    ```

### `m3tal dash down`
Stops and removes the `m3tal-dashboard` container.
*   **Usage Example:**
    ```bash
    m3tal dash down
    ```

### `m3tal dash restart`
Restarts the `m3tal-dashboard` container, applying any recent configuration changes or simply refreshing the service.
*   **Usage Example:**
    ```bash
    m3tal dash restart
    ```

### `m3tal dash logs`
Streams real-time logs from the `m3tal-dashboard` container, useful for monitoring its operation and debugging issues.
*   **Usage Example:**
    ```bash
    m3tal dash logs
    ```

### `m3tal dash status`
Displays the current operational status of the `m3tal-dashboard` container (e.g., running, exited, starting).
*   **Usage Example:**
    ```bash
    m3tal dash status
    ```

---

## 📦 Stack Management

These commands are used to manage all Docker Compose-based application stacks deployed within your M3TAL environment.

### `m3tal up`
Executes `docker compose up` across all `*-compose.yml` files located in the `/docker/` directory. This command brings up all your configured services and stacks.
*   **Usage Example:**
    ```bash
    m3tal up
    ```

### `m3tal down`
Executes `docker compose down` across all `*-compose.yml` files found in the `/docker/` directory. This command stops and removes all M3TAL-managed services and their associated networks.
*   **Usage Example:**
    ```bash
    m3tal down
    ```

### `m3tal logs`
Streams aggregated, real-time logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of all service activity.
*   **Usage Example:**
    ```bash
    m3tal logs
    ```

---

## 🛠️ Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service, providing a robust and persistent backend for the CLI and dashboard. You can manage this service using standard systemctl commands.

*   **Check the status of the M3TAL API daemon:**
    ```bash
    systemctl status m3tal-api
    ```

*   **Restart the M3TAL API daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

*   **Follow real-time logs for the M3TAL API daemon:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## 📁 Filesystem Contract

The M3TAL ecosystem maintains a strict filesystem contract. Understanding these paths is crucial for configuration, data management, and troubleshooting.

| Path                        | Purpose                                                                                                                                                                                                                                                               |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary Configuration File.** This file contains all environment variables that configure the M3TAL system and its services. It is primarily managed by `m3tal config wizard` and `m3tal config set`.                                                               |
| `/var/lib/m3tal/state.db`   | **SQLite State Database.** An SQLite database used by the M3TAL API daemon to store its operational state, service information, and other internal data. This file is automatically created and managed by the API.                                                    |
| `/opt/m3tal/stack/`         | **Canonical Stack Directory.** This is the internal, canonical location where all Docker Compose files and Traefik dynamic configuration snippets reside.                                                                                                                |
| `/docker`                   | **User-Facing Stack Directory (Symlink).** A symbolic link to `/opt/m3tal/stack/`. This is the primary directory users interact with for adding, modifying, or inspecting Docker Compose files. When you create `my-app-compose.yml`, place it here.                   |
| `/docker/users.json`        | **Dashboard Credential Store.** A JSON file storing hashed usernames and passwords for the M3TAL dashboard. This file is managed by the `m3tal dashpass` command.                                                                                                       |

---

## 🏗️ M3TAL System Architecture & Dashboard Access

M3TAL is built upon a modular architecture leveraging Docker Compose for service orchestration and a Go-based API daemon for system control.

### Components Overview

*   **CLI Binary (`/usr/bin/m3tal`):** The single entrypoint for all M3TAL operations, installed via APT.
*   **API Daemon (`m3tal-api.service`):** A Go binary running as a systemd service on port `8080`. It manages Docker interactions, maintains the SQLite state database (`/var/lib/m3tal/state.db`), and exposes the core API routes.
*   **Dashboard Container (`m3tal-dashboard`):** A Python/Flask application running within Docker (internally on port `8082`). It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`):** A Docker container acting as a reverse proxy, exposing services on host port `80`. It uses a file provider for dynamic configuration (from `/docker/dynamic/`) and automatically discovers services via Docker labels.
*   **Cloudflared (`routing-compose.yml`):** An optional Cloudflare Tunnel container that provides secure, zero-config internet access to your services without exposing ports directly.

### Dashboard Access — Two Modes

Access to the M3TAL dashboard is controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`. This variable determines which Docker Compose override file is used, configuring the dashboard for either direct port access or Traefik routing.

1.  **Mode 1: `local` (Default)**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
    *   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port (`8082`) to a host port (`${DASHBOARD_PORT:-8082}`).
    *   **Access:** Via `http://HOST_IP:8082` or `http://localhost:8082`.
    *   **Requirements:** No Traefik required. Works immediately after `m3tal dash up`.
    *   **Best for:** LAN-only setups, first-time users, and local development/testing.

2.  **Mode 2: `traefik`**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
    *   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which adds Traefik-specific labels to the dashboard container. Traefik then routes incoming requests for `dash.${DOMAIN}` (e.g., `dash.myhomelab.local`) to the dashboard on its internal port `8082`.
    *   **Access:** Via `http://dash.DOMAIN` (e.g., `http://dash.myhomelab.local`). Requires Traefik to be running (`m3tal up` will start it via `routing-compose.yml`).
    *   **Requirements:** Traefik must be deployed and configured (typically part of the `routing` stack). `DOMAIN` must be set in `/etc/m3tal/.env`.
    *   **Best for:** Domain-based access, integrating with other services behind Traefik, and leveraging Traefik's advanced routing features.

### Docker / Compose Runtime

M3TAL relies on **Docker Engine** and **Docker Compose V2** as hard dependencies.

*   The `m3tal up` command orchestrates all Docker Compose files ending in `*-compose.yml` found in `/docker/`. This includes core M3TAL services like Traefik (`routing-compose.yml`) and any user-defined application stacks.
*   The `m3tal dash up` command performs a specialized `docker compose` operation for the dashboard:
    1.  It first fetches the latest `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
    2.  It then reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env` to determine which override to apply (either `local` or `traefik`).
    3.  Finally, it starts the `m3tal-dashboard` container with the correct configuration.
*   **User Stacks:** To deploy a new application stack, simply place your `my-app-compose.yml` file into the `/docker/` directory, ensure any required variables are set in `/etc/m3tal/.env`, and run `m3tal up`.

### Traefik Routing Architecture

Traefik, deployed via `routing-compose.yml`, acts as the central ingress point for your M3TAL services when `DASHBOARD_EXPOSE_MODE` is `traefik`.

*   **Entry Point:** It binds to host port `80` (and `443` for HTTPS, if configured) as its `web` entry point.
*   **Service Discovery:** Traefik automatically discovers services by inspecting Docker container labels, especially for services within the `proxy` network.
*   **Dynamic Configuration:** It uses a file provider to load additional dynamic configuration from `/docker/dynamic/`, which supports hot-reloading.
*   **API Routing:** An example of dynamic configuration, `dynamic/api.yml`, routes requests for `api.${DOMAIN}` directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, the `m3tal-dashboard` container's labels (from `m3tal-compose.traefik.yml`) instruct Traefik to route `dash.${DOMAIN}` to the dashboard container on port `8082`.

### Port Map

Understanding the port assignments is essential for firewall configuration and direct access.

| Port | Service                               | Access                                                          |
| :--- | :------------------------------------ | :-------------------------------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (when Traefik is running)                                |
| 8080 | M3TAL API daemon (Go binary)          | Host-local only (accessed by dashboard and Traefik)             |
| 8081 | Traefik dashboard (internal)          | Host-local only (`127.0.0.1:8081:8080` container port mapping) |
| 8082 | M3TAL Dashboard container (internal) | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

---

## 🐋 Docker Fallback Commands

While the `m3tal` CLI provides a convenient abstraction, you can always interact directly with Docker Compose for troubleshooting or advanced scenarios. These commands mirror the functionality of `m3tal up`, `m3tal down`, and `m3tal logs`.

### Starting All Stacks Manually

To replicate `m3tal up`, you would typically list all relevant `*-compose.yml` files found in `/docker/`. For example, if you have `routing-compose.yml`, `m3tal-compose.yml`, and `ollama-compose.yml`:

```bash
docker compose -f /docker/routing-compose.yml \
               -f /docker/m3tal-compose.yml \
               -f /docker/m3tal-compose.$(m3tal config get DASHBOARD_EXPOSE_MODE).yml \
               -f /docker/ollama-compose.yml \
               up -d
```
*Note: The `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` override is selected based on `DASHBOARD_EXPOSE_MODE`.*

### Starting the Dashboard Manually

To replicate `m3tal dash up`, specifically for the dashboard, you would:

**If `DASHBOARD_EXPOSE_MODE` is `local`:**
```bash
docker compose -f /docker/m3tal-compose.yml \
               -f /docker/m3tal-compose.local.yml \
               up -d m3tal-dashboard
```

**If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
```bash
docker compose -f /docker/m3tal-compose.yml \
               -f /docker/m3tal-compose.traefik.yml \
               up -d m3tal-dashboard
```

### Stopping All Stacks Manually

To replicate `m3tal down`:

```bash
docker compose -f /docker/routing-compose.yml \
               -f /docker/m3tal-compose.yml \
               -f /docker/m3tal-compose.local.yml \
               -f /docker/m3tal-compose.traefik.yml \
               -f /docker/ollama-compose.yml \
               down
```
*Note: Include all active compose files, including both dashboard overrides to ensure a clean shutdown regardless of the active mode.*

### Streaming Aggregated Logs Manually

To replicate `m3tal logs`:

```bash
docker compose -f /docker/routing-compose.yml \
               -f /docker/m3tal-compose.yml \
               -f /docker/m3tal-compose.local.yml \
               -f /docker/m3tal-compose.traefik.yml \
               -f /docker/ollama-compose.yml \
               logs -f
```

---