# M3TAL CLI Command Reference

Welcome to the M3TAL Ecosystem, Documented by DocSmith, the M3TAL Ecosystem Documentation Architect. This guide serves as a complete cheat-sheet for the `m3tal` CLI, your primary interface for managing your self-hosted server stack.

M3TAL provides a unified Go binary (`/usr/bin/m3tal`) for all operations, abstracting away complex Docker commands and system service management. Underneath, M3TAL relies on **Docker Engine** and **Docker Compose V2** to orchestrate services, and a **Go API daemon** (`m3tal-api.service`) running on port `8080` to manage state and Docker interactions.

---

## 1. M3TAL Installation

M3TAL is distributed via an APT repository for easy installation and updates.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 2. Core M3TAL CLI Commands

These commands provide immediate interaction with the M3TAL ecosystem or launch the main applications.

### `m3tal` (or `m3tal ui`) — Launch Desktop Application
Launches the M3TAL Linux Desktop Application, providing a graphical interface for managing your system.
```bash
m3tal
```
**Example Usage:**
```bash
# Launch the M3TAL desktop UI
m3tal
```

### `m3tal headless [port]` — Start M3TAL in Headless Mode
Starts the M3TAL API daemon directly in the terminal, bypassing the desktop UI. This is ideal for home servers or when you need to run M3TAL without a GUI, allowing remote management via the dashboard or API. If no port is specified, it will use the default (`8080`).
```bash
m3tal headless
```
**Example Usage:**
```bash
# Start M3TAL in headless mode on the default port 8080
m3tal headless

# Start M3TAL in headless mode on a custom port 9000
m3tal headless 9000
```

### `m3tal stats` — Print Instant System Metrics
Displays real-time host system, GPU, thermal, and storage metrics directly in your terminal. Useful for quick diagnostics and monitoring.
```bash
m3tal stats
```
**Example Usage:**
```bash
# Get an immediate snapshot of system performance
m3tal stats
```

### `m3tal doctor` — Pre-flight Health Check
Runs a comprehensive health check, validating Docker connectivity, `/etc/m3tal/.env` configuration, and critical port availability. Essential for troubleshooting initial setup or connectivity issues.
```bash
m3tal doctor
```
**Example Usage:**
```bash
# Check the health of your M3TAL installation
m3tal doctor
```

---

## 3. Configuration Management

M3TAL's primary configuration file is located at `/etc/m3tal/.env`. These commands help you manage environment variables crucial for your services.

### `m3tal config wizard` — Interactive Configuration Wizard
Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended way for first-time setup.
```bash
m3tal config wizard
```
**Example Usage:**
```bash
# Start the guided setup for M3TAL's environment variables
m3tal config wizard
```

### `m3tal config set KEY VALUE` — Set a Single Environment Variable
Sets or updates a specific environment variable in `/etc/m3tal/.env`.
```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```
**Example Usage:**
```bash
# Set the dashboard to use Traefik routing
m3tal config set DASHBOARD_EXPOSE_MODE traefik

# Change the base storage path for all services
m3tal config set BASE_STORAGE_PATH /mnt/my-data
```

### `m3tal config get KEY` — Read a Single Environment Variable
Retrieves the value of a specific environment variable from `/etc/m3tal/.env`.
```bash
m3tal config get DOMAIN
```
**Example Usage:**
```bash
# Get the currently configured domain
m3tal config get DOMAIN

# Check the user ID for container permissions
m3tal config get PUID
```

### `m3tal config scan` — List All Environment Variables Across Stacks
Scans all Docker Compose files in `/docker/` and lists all environment variables declared or used, including their defaults. This helps identify all possible configuration options for your stacks.
```bash
m3tal config scan
```
**Example Usage:**
```bash
# Discover all available environment variables used by your M3TAL stacks
m3tal config scan
```

### `m3tal config list` — List Current .env File Contents
Displays the current contents of your primary configuration file, `/etc/m3tal/.env`, to the terminal.
```bash
m3tal config list
```
**Example Usage:**
```bash
# View the active configuration for your M3TAL system
m3tal config list
```

---

## 4. Dashboard Management

The M3TAL Dashboard (`m3tal-dashboard` container) is a web interface for managing your M3TAL instance. These commands control its lifecycle and access. Its credentials are stored in `/docker/users.json`.

**Critical: Dashboard Access Modes**

The dashboard has two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. The `m3tal dash up` command automatically applies the correct Docker Compose override based on this setting.

1.  **`local` (Default Mode)**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
    *   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port (`8082`) to the host's port (`${DASHBOARD_PORT:-8082}`).
    *   **Access:** `http://HOST_IP:8082` or `http://localhost:8082`
    *   **Best for:** LAN-only setups, first-time users, or local testing where Traefik is not yet configured or desired. No Traefik required.

2.  **`traefik` Mode**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
    *   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik then routes requests for `dash.${DOMAIN}` to the dashboard on port `8082`.
    *   **Access:** `http://dash.DOMAIN` (e.g., `http://dash.mydomain.com`)
    *   **Best for:** Domain-based setups, integrating with other services behind Traefik, or when you need HTTPS (managed by Traefik). Requires Traefik to be running via `m3tal up`.

### `m3tal dashpass [username] [password]` — Update Dashboard User Password
Updates credentials for accessing the M3TAL Dashboard. If `username` and `password` are omitted, the command will prompt you interactively. Stores credentials in `/docker/users.json`.
```bash
m3tal dashpass admin new_secure_password
```
**Example Usage:**
```bash
# Set the password for the 'admin' user to 'new_secure_password'
m3tal dashpass admin new_secure_password

# Interactively update a user's password
m3tal dashpass
```

### `m3tal dash up` — Start M3TAL Dashboard Container
Pulls the latest dashboard Docker Compose configuration from GitHub and starts the `m3tal-dashboard` container with the appropriate `DASHBOARD_EXPOSE_MODE` override (`local.yml` or `traefik.yml`).
```bash
m3tal dash up
```
**Example Usage:**
```bash
# Ensure the dashboard is running, pulling updates if available
m3tal dash up
```

### `m3tal dash down` — Stop M3TAL Dashboard Container
Stops and removes the `m3tal-dashboard` container.
```bash
m3tal dash down
```
**Example Usage:**
```bash
# Halt the M3TAL dashboard service
m3tal dash down
```

### `m3tal dash restart` — Restart M3TAL Dashboard Container
Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.
```bash
m3tal dash restart
```
**Example Usage:**
```bash
# Restart the dashboard after changing its environment variables
m3tal dash restart
```

### `m3tal dash logs` — Stream M3TAL Dashboard Container Logs
Streams real-time logs from the `m3tal-dashboard` container to your terminal. Press `Ctrl+C` to exit.
```bash
m3tal dash logs
```
**Example Usage:**
```bash
# Monitor the dashboard for errors or activity
m3tal dash logs
```

### `m3tal dash status` — Show M3TAL Dashboard Container Status
Displays the current operational status of the `m3tal-dashboard` container (e.g., running, stopped, exited).
```bash
m3tal dash status
```
**Example Usage:**
```bash
# Check if the dashboard is currently active
m3tal dash status
```

---

## 5. Stack Management

M3TAL uses Docker Compose to manage all your services, referred to as "stacks." All Compose files (`*-compose.yml`) are located in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).

### `m3tal up` — Run Docker Compose Up Across All Stacks
Executes `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command starts or recreates all services defined in your M3TAL ecosystem, including the Traefik gateway if configured.
```bash
m3tal up
```
**Example Usage:**
```bash
# Start all M3TAL services and any user-defined stacks
m3tal up
```

### `m3tal down` — Run Docker Compose Down Across All Stacks
Executes `docker compose down` across all `*-compose.yml` files in `/docker/`. This command stops and removes all services, networks, and volumes defined in your M3TAL ecosystem.
```bash
m3tal down
```
**Example Usage:**
```bash
# Stop and remove all M3TAL services and user stacks
m3tal down
```

### `m3tal logs` — Stream Aggregated Logs from All Running Stacks
Streams real-time, aggregated logs from all currently running Docker containers managed by M3TAL Compose. Press `Ctrl+C` to exit.
```bash
m3tal logs
```
**Example Usage:**
```bash
# Monitor logs from all services for a comprehensive view
m3tal logs
```

---

## 6. System Services & Diagnostics

The M3TAL API daemon (`m3tal-api`) is a Go binary running as a systemd service, essential for M3TAL's functionality.

### `systemctl status m3tal-api` — Check API Daemon Status
Checks the current status of the `m3tal-api.service` systemd unit. This shows if the API daemon is running, stopped, or if there were any errors.
```bash
sudo systemctl status m3tal-api
```
**Example Usage:**
```bash
# Verify the M3TAL API daemon is active and healthy
sudo systemctl status m3tal-api
```

### `systemctl restart m3tal-api` — Restart API Daemon
Restarts the `m3tal-api.service`. This is often necessary after changing `/etc/m3tal/.env` variables that the API daemon relies on, or for troubleshooting.
```bash
sudo systemctl restart m3tal-api
```
**Example Usage:**
```bash
# Apply new configuration changes to the M3TAL API
sudo systemctl restart m3tal-api
```

### `journalctl -u m3tal-api -f` — Stream API Daemon Logs
Streams real-time logs from the `m3tal-api.service` systemd unit to your terminal. This is crucial for debugging issues with the M3TAL API itself. Press `Ctrl+C` to exit.
```bash
sudo journalctl -u m3tal-api -f
```
**Example Usage:**
```bash
# Monitor the M3TAL API daemon for errors or activity
sudo journalctl -u m3tal-api -f
```

---

## 7. M3TAL Filesystem Contract

M3TAL adheres to a strict filesystem contract for configuration and data management.

| Path                     | Purpose                                                                 |
| :----------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.           |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the API daemon.      |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains Docker Compose files and Traefik config. |
| `/docker`                | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.                |

---

## 8. Docker Fallback Commands

While the `m3tal` CLI is the preferred interface, you can always interact directly with Docker Compose V2. This is useful for advanced debugging or specific operations.

**Key Concepts:**
*   M3TAL uses **Docker Engine + Docker Compose V2**.
*   All Docker Compose files are located in `/docker/` (a symlink to `/opt/m3tal/stack/`).
*   To manage a specific stack, you'll generally pass its compose file(s) with `-f`.

### Starting All Stacks Manually
Equivalent to `m3tal up`. This command runs `docker compose up -d` for all compose files in `/docker/`.
```bash
docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-other-stack-compose.yml up -d
```
**Example Usage:**
```bash
# Manually bring up all core M3TAL and user-defined services
docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml up -d
```

### Stopping All Stacks Manually
Equivalent to `m3tal down`.
```bash
docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-other-stack-compose.yml down
```
**Example Usage:**
```bash
# Manually take down all core M3TAL and user-defined services
docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml down
```

### Managing the M3TAL Dashboard Manually
The dashboard requires specific compose overrides based on `DASHBOARD_EXPOSE_MODE`.

*   **For `DASHBOARD_EXPOSE_MODE=local`:**
    ```bash
    # Start dashboard in local mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard

    # Stop dashboard in local mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down m3tal-dashboard
    ```
*   **For `DASHBOARD_EXPOSE_MODE=traefik`:**
    ```bash
    # Start dashboard in Traefik mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d m3tal-dashboard

    # Stop dashboard in Traefik mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml down m3tal-dashboard
    ```

### Viewing Logs for a Specific Container
```bash
docker logs -f m3tal-dashboard
```
**Example Usage:**
```bash
# Stream logs for the Traefik container
docker logs -f traefik
```

### Checking Status of a Specific Container
```bash
docker ps -a --filter name=m3tal-dashboard
```
**Example Usage:**
```bash
# Check the status of the Cloudflared container
docker ps -a --filter name=cloudflared
```

---

## 9. Traefik Routing Architecture

Traefik acts as the reverse proxy, managing external access to your services. It's deployed via `routing-compose.yml` and listens on host port `80`.

*   **Entry Point:** Traefik binds host port `80` (HTTP) and optionally `443` (HTTPS) as its entry points.
*   **Service Discovery:** It automatically discovers services by reading Docker labels on containers (e.g., `m3tal-dashboard` in `traefik` mode).
*   **File Provider:** Traefik also loads dynamic configuration from `/docker/dynamic/` (specifically, `/etc/traefik/dynamic` within the container), which hot-reloads.
*   **API Routing:** M3TAL exposes its API daemon (running on host port `8080`) to Traefik via a dynamic configuration file (`/docker/dynamic/api.yml`). This routes `api.${DOMAIN}` to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` to the `m3tal-dashboard` container on its internal port `8082`, using labels defined in `m3tal-compose.traefik.yml`.

---

## 10. M3TAL Port Map

| Port | Service                     | Access                                     |
| :--- | :-------------------------- | :----------------------------------------- |
| 80   | Traefik HTTP entry point    | Public (if Traefik is running)             |
| 8080 | M3TAL API daemon (Go binary) | Host-local (accessible to containers via `host.docker.internal:8080`, Traefik routes `api.DOMAIN` here) |
| 8081 | Traefik dashboard           | Host-local only (`127.0.0.1:8081`)         |
| 8082 | M3TAL Dashboard (container) | Direct port (local mode: `HOST_IP:8082`) or via Traefik (traefik mode: `dash.DOMAIN`) |