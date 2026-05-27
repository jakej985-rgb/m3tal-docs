# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for the M3TAL Command Line Interface (CLI), detailing every command and its usage.

---

## 🚀 Get Started: M3TAL Installation

The M3TAL CLI and API daemon are installed via an APT package.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 🧠 M3TAL System Architecture Overview

The M3TAL ecosystem is comprised of several key components working in concert:

*   **CLI binary** (`/usr/bin/m3tal`): Your primary interface for interacting with M3TAL.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service (port 8080), the backend for all operations, managing Docker and the state DB.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application providing a web UI, communicating with the API daemon.
*   **Traefik gateway** (`routing-compose.yml`): The reverse proxy, exposing services on port 80, handling domain-based routing.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel for secure, zero-config internet access to your services.

---

## 🗃️ M3TAL Filesystem Contract

M3TAL relies on a specific filesystem structure for its operation and configuration.

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file, containing environment variables for M3TAL.  |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's internal state.                        |
| `/opt/m3tal/stack/`      | Canonical directory for Docker Compose files and Traefik configuration. |
| `/docker`                | **Symlink** to `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store, managed by `m3tal dashpass`.               |
| `/docker/dynamic/`       | Directory for Traefik's dynamic configuration files (e.g., API routing). |

---

## 🚀 Core M3TAL Commands

### `sudo m3tal`

Opens the interactive Text User Interface (TUI) Control Center. This provides a menu-driven interface for common operations. Requires `sudo` for full access to system and Docker resources.

```bash
sudo m3tal
```

### `m3tal init`

Generates the primary M3TAL configuration file, `/etc/m3tal/.env`, from default values. This command should be run on first installation to set up the necessary environment variables.

```bash
m3tal init
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL environment. This includes verifying Docker connectivity, checking the validity of `/etc/m3tal/.env`, and ensuring required ports are available. It's an essential first step for troubleshooting.

```bash
m3tal doctor
```

---

## ⚙️ Configuration Management (`m3tal config`)

These commands allow you to manage the `/etc/m3tal/.env` configuration file.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended way to manage your M3TAL settings.

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a specific environment variable (`KEY`) to a new `VALUE` in `/etc/m3tal/.env`. This command is useful for quick, non-interactive configuration changes.

```bash
# Example: Set the primary domain for your services
m3tal config set DOMAIN myhomelab.com

# Example: Change the timezone
m3tal config set TZ America/New_York
```

### `m3tal config get KEY`

Retrieves and displays the current value of a specified environment variable (`KEY`) from `/etc/m3tal/.env`.

```bash
# Example: Get the currently configured domain
m3tal config get DOMAIN

# Example: Check the dashboard exposure mode
m3tal config get DASHBOARD_EXPOSE_MODE
```

### `m3tal config scan`

Lists all environment variables that M3TAL recognizes and their current values, aggregated across all relevant stacks and configuration files. This helps understand the complete active configuration.

```bash
m3tal config scan
```

### `m3tal config list`

Displays the raw contents of the current `/etc/m3tal/.env` file. This is useful for reviewing all defined variables directly.

```bash
m3tal config list
```

---

## 📊 Dashboard Management (`m3tal dash`)

These commands are specifically for managing the M3TAL Dashboard container.

### `m3tal dashpass [username] [password]`

Updates the password for a specified dashboard user. If `username` and `password` are omitted, the command will prompt you interactively. User credentials are stored in `/docker/users.json`.

```bash
# Example: Set password for 'admin' user interactively
m3tal dashpass admin

# Example: Set password for 'admin' user directly
m3tal dashpass admin MySuperSecurePassword123!
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the `m3tal-dashboard` container. This command intelligently selects the correct override file (`.local` or `.traefik`) based on the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

```bash
# Start the M3TAL Dashboard
m3tal dash up
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

```bash
# Stop the M3TAL Dashboard
m3tal dash down
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. This is useful after making configuration changes that affect the dashboard.

```bash
# Restart the M3TAL Dashboard
m3tal dash restart
```

### `m3tal dash logs`

Streams the aggregated logs from the `m3tal-dashboard` container to your terminal, providing real-time insights into its operation.

```bash
# View dashboard logs in real-time
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container, including its running state, uptime, and port mappings.

```bash
# Check the dashboard container's status
m3tal dash status
```

### 🌐 Dashboard Access Modes (Critical)

The M3TAL Dashboard can be accessed in two primary ways, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### **Mode 1: `local` (Default)**

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file, which directly binds the dashboard container's port 8082 to the host machine (e.g., `${DASHBOARD_PORT:-8082}:8082`). Traefik is **not** involved in this mode.
*   **Access URL**: `http://HOST_IP:8082` or `http://localhost:8082` (where `HOST_IP` is the IP address of your M3TAL server).
*   **Use Cases**: Ideal for initial setup, LAN-only access, local testing, or scenarios where a reverse proxy is not desired.

```bash
# Ensure DASHBOARD_EXPOSE_MODE is set to local
m3tal config set DASHBOARD_EXPOSE_MODE local

# Start the dashboard
m3tal dash up

echo "Access the dashboard at http://$(hostname -I | awk '{print $1}'):8082 or http://localhost:8082"
```

#### **Mode 2: `traefik`**

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file, which adds Traefik-specific labels to the dashboard container. Traefik (which must be running via `m3tal up`) then automatically discovers and routes traffic to the dashboard based on a domain rule.
*   **Access URL**: `http://dash.YOUR_DOMAIN` (e.g., `http://dash.myhomelab.com`), where `YOUR_DOMAIN` is set via `m3tal config set DOMAIN YOUR_DOMAIN`.
*   **Use Cases**: Recommended for domain-based setups, exposing services via a reverse proxy, and integration into a larger Traefik-managed environment.

```bash
# Set your domain and enable Traefik exposure
m3tal config set DOMAIN myhomelab.com
m3tal config set DASHBOARD_EXPOSE_MODE traefik

# Start the dashboard
m3tal dash up

# Ensure all M3TAL stacks, including Traefik, are running
m3tal up

echo "Access the dashboard at http://dash.myhomelab.com"
```

---

## 🐳 Stack Management Commands

These commands manage all Docker Compose-based services (stacks) defined in the `/docker/` directory.

### `m3tal up`

Runs `docker compose up -d` across **all** `*-compose.yml` files found within the `/docker/` directory (which is symlinked to `/opt/m3tal/stack/`). This command starts or updates all your configured Docker stacks, including `routing-compose.yml` (Traefik, Cloudflared) and any user-defined stacks.

```bash
# Start all M3TAL services and user-defined stacks
m3tal up
```

### `m3tal down`

Runs `docker compose down` across **all** Docker Compose files in `/docker/`. This command stops and removes all containers, networks, and volumes defined in these stacks.

```bash
# Stop and remove all M3TAL services and user-defined stacks
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a unified view for monitoring the health and activity of your entire M3TAL ecosystem.

```bash
# View aggregated logs from all running M3TAL containers
m3tal logs
```

---

## 🛠️ Systemd Service Management

The M3TAL API daemon runs as a systemd service, `m3tal-api.service`. You can use standard `systemctl` and `journalctl` commands to manage and monitor it.

*   **Check API daemon status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart the API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream API daemon logs**:
    ```bash
    sudo journalctl -u m3tal-api -f
    ```
*   **Enable API daemon to start on boot**:
    ```bash
    sudo systemctl enable m3tal-api
    ```
*   **Disable API daemon from starting on boot**:
    ```bash
    sudo systemctl disable m3tal-api
    ```

---

## 📦 Docker Direct Commands (Fallback)

M3TAL leverages Docker Engine and Docker Compose V2. While the `m3tal` CLI abstracts many Docker commands, you can always use direct `docker compose` commands as a fallback or for advanced debugging. The M3TAL CLI primarily operates on files within `/docker/` (which is `/opt/m3tal/stack/`).

To interact with a specific stack, navigate to the `/docker/` directory or use the `-f` flag with `docker compose`.

```bash
# Navigate to the M3TAL stack directory
cd /docker/

# Start all stacks defined in the directory (same as m3tal up)
# Note: Replace `my-stack-compose.yml` with your actual user-defined stack files.
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f my-stack-compose.yml up -d

# Stop all stacks defined in the directory (same as m3tal down)
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f my-stack-compose.yml down

# View logs for a specific stack (e.g., routing)
sudo docker compose -f routing-compose.yml logs -f

# View logs for a specific container (e.g., m3tal-dashboard)
sudo docker logs -f m3tal-dashboard

# Check the status of all running containers
sudo docker ps

# Pull the latest images for all defined services in m3tal-compose.yml (e.g., dashboard)
sudo docker compose -f m3tal-compose.yml pull

# Rebuild and restart a specific service if you've modified its Dockerfile or context (less common for M3TAL)
sudo docker compose -f m3tal-compose.yml up -d --build m3tal-dashboard
```

**Note**: When using direct `docker compose` commands, ensure you specify all relevant compose files if you intend to manage multiple stacks simultaneously, or target specific files as needed. The `m3tal up` and `m3tal down` commands automatically discover and apply to all `*-compose.yml` files in `/docker/`.

---

## 🛣️ Traefik Routing Architecture

Traefik acts as the central reverse proxy for M3TAL, deployed via `routing-compose.yml`.

*   **Entry Points**: Binds to host port 80 (and 443 if HTTPS is configured) as its primary entry points.
*   **Service Discovery**: Automatically discovers Docker containers with Traefik labels, routing traffic to them based on defined rules (e.g., `Host(\`dash.${DOMAIN}\`)`).
*   **File Provider**: Loads additional dynamic configuration from `/docker/dynamic/`, allowing for custom routing rules (e.g., `api.yml` for the M3TAL API daemon).
*   **M3TAL API Routing**: The M3TAL API daemon (running on host port 8080) is exposed via Traefik through a file provider config in `/docker/dynamic/api.yml`, which routes `api.YOUR_DOMAIN` to `http://host.docker.internal:8080`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.YOUR_DOMAIN` to the `m3tal-dashboard` container's internal port 8082 based on container labels.

---

## 🔗 M3TAL Port Map

| Port | Service                                | Access                 |
| :--- | :------------------------------------- | :--------------------- |
| 80   | Traefik HTTP entry point               | Public (traefik mode)  |
| 443  | Traefik HTTPS entry point              | Public (traefik mode)  |
| 8080 | M3TAL API daemon (Go binary)           | Host-local             |
| 8081 | Traefik dashboard                      | Host-local only        |
| 8082 | M3TAL Dashboard (Python/Flask container)| Direct port (local mode) or via Traefik (traefik mode) |