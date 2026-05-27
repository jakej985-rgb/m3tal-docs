# M3TAL CLI Command Reference

Welcome to the M3TAL Ecosystem Documentation, fellow architect! This document serves as a comprehensive command-line interface (CLI) cheat-sheet for managing your M3TAL instance. The `m3tal` CLI binary, installed via APT to `/usr/bin/m3tal`, is your unified gateway to controlling the M3TAL API daemon, Docker services, and system configuration.

## 🚀 Getting Started: APT Installation

For first-time setup or upgrading your M3TAL CLI, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 🗂️ Filesystem Contract

The M3TAL ecosystem relies on a strict filesystem contract for its operations. Understanding these paths is crucial for advanced configuration and troubleshooting:

| Path                        | Purpose                                                                                                                                              |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. This file contains environment variables that dictate the behavior of M3TAL and its Docker containers. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. This database is auto-created and managed by the M3TAL API daemon, storing internal state and service information.            |
| `/opt/m3tal/stack/`         | Canonical stack directory. This directory is where all Docker Compose YAML files (`*-compose.yml`) and Traefik configuration are stored.               |
| `/docker`                   | **Symlink** → `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. When you interact with `/docker/`, you are operating within `/opt/m3tal/stack/`. |
| `/docker/users.json`        | Dashboard credential store. This JSON file securely holds the usernames and hashed passwords for accessing the M3TAL dashboard. Managed by `m3tal dashpass`. |
| `/docker/dynamic/`          | Directory for Traefik's dynamic configuration files (e.g., `api.yml`). These files allow hot-reloading of routing rules.                             |

## 🏛️ M3TAL System Architecture Overview

The M3TAL ecosystem is built upon several core components:

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary for all interactions.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port `8080` (host-local). It manages Docker, the internal state DB, and serves API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services by domain name on port `80` (HTTP) and `443` (HTTPS, if configured). It uses a file provider for dynamic routing and Docker labels for service discovery.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing secure, zero-config internet access to your services.

M3TAL uses **Docker Engine + Docker Compose V2** as its runtime. These are hard dependencies. The `m3tal` CLI commands abstract these underlying Docker operations.

## 🌐 Dashboard Access — Two Modes

Accessing the M3TAL dashboard is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. Understanding these modes is critical for proper setup.

### Mode 1: `local` (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the Docker Compose override file: `m3tal-compose.local.yml`.
*   **Direct Port Binding**: The dashboard container's internal port `8082` is directly mapped to `DASHBOARD_PORT` (default `8082`) on the host system.
*   **Access via**: `http://HOST_IP:8082` or `http://localhost:8082`.
*   **No Traefik Required**: Works out of the box without the need for the Traefik proxy.
*   **Best for**: LAN-only setups, first-time users, local development, or simple home server environments.

### Mode 2: `traefik`

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the Docker Compose override file: `m3tal-compose.traefik.yml`.
*   **Traefik Routing**: Adds Docker labels to the dashboard container, allowing Traefik to route requests for `dash.${DOMAIN}` to the dashboard on its internal port `8082`.
*   **Access via**: `http://dash.YOUR_DOMAIN_HERE` (e.g., `http://dash.m3tal.local`).
*   **Traefik Requirement**: Traefik must be running via `m3tal up` for this mode to function.
*   **Best for**: Domain-based setups, exposing services via a reverse proxy, or integrating with other domain-bound services.

**Example Dashboard Configuration for `m3tal-dashboard` container:**

*   **Base Compose (`m3tal-compose.yml`)**:
    ```yaml
    services:
      m3tal-dashboard:
        image: ghcr.io/jakej985-rgb/m3tal-godash:debug
        container_name: m3tal-dashboard
        user: "${PUID:-1000}:${PGID:-1000}"
        volumes:
          - ${CONFIG_PATH:-/mnt/config}/m3tal/state:/docker/state
          - ${BASE_STORAGE_PATH:-/mnt}:/mnt
          - .:/docker/state/config:rw
        command: python3 server.py
        restart: unless-stopped
        environment:
          - TZ=${TZ:-America/Denver}
          - DASHBOARD_SECRET=${DASHBOARD_SECRET:-}
          - STATE_DIR=/docker/state
          - USERS_FILE=/docker/state/config/users.json
          - DASHBOARD_HOST=0.0.0.0
          - DASHBOARD_PORT=8082
          - PYTHONUNBUFFERED=1
          - GO_API_URL=${GO_API_URL:-http://host.docker.internal:5050}
        extra_hosts:
          - "host.docker.internal:host-gateway"
        labels:
          - "m3tal.stack=control-plane"
        networks:
          - proxy
        deploy:
          resources:
            limits:
              cpus: '0.5'
              memory: 250M
    networks:
      proxy:
        external: true
    ```
*   **Local Mode Override (`m3tal-compose.local.yml`)**:
    ```yaml
    services:
      m3tal-dashboard:
        ports:
          - "${DASHBOARD_PORT:-8082}:8082"
    ```
*   **Traefik Mode Override (`m3tal-compose.traefik.yml`)**:
    ```yaml
    services:
      m3tal-dashboard:
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.dashboard.rule=Host(`dash.${DOMAIN:-localhost}`)"
          - "traefik.http.routers.dashboard.entrypoints=web"
          - "traefik.http.services.dashboard.loadbalancer.server.port=8082"
          - "traefik.docker.network=proxy"
    ```

## ⚙️ Core M3TAL Commands

These commands provide fundamental control over the M3TAL system.

### `sudo m3tal`

Opens the interactive Text-based User Interface (TUI) Control Center. This provides a numbered menu for common operations and system status.
*The M3TAL API daemon must be running (`systemctl status m3tal-api`) for the TUI to fully function.*

**Usage Example:**
```bash
sudo m3tal
# This will launch the interactive menu:
#
#    M3TAL Control Center
#    --------------------
#    1. System Status
#    2. Manage Stacks
#    3. Configure M3TAL
#    4. Dashboard Access
#    ...
```

### `m3tal init`

Generates the primary configuration file, `/etc/m3tal/.env`, from internal defaults. This command is essential for the first installation of M3TAL and sets up the foundational environment variables. It will not overwrite an existing `.env` file unless forced.

**Usage Example:**
```bash
m3tal init
# Output:
# Creating default /etc/m3tal/.env... Done.
# Please review and configure your environment variables using 'm3tal config wizard'.
```

### `m3tal doctor`

Performs a pre-flight health check of the M3TAL system. This includes verifying Docker connectivity, checking the validity of `/etc/m3tal/.env`, and ensuring required network ports are available. Run this to quickly diagnose common issues.

**Usage Example:**
```bash
m3tal doctor
# Output:
# Docker connectivity: OK
# /etc/m3tal/.env validation: OK
# Port 8080 (M3TAL API): Available
# Port 8082 (M3TAL Dashboard): Available
# All systems nominal.
```

## 📝 Configuration Management (`m3tal config`)

These commands manage the `/etc/m3tal/.env` configuration file, which defines crucial environment variables for M3TAL and its Docker containers.

### `m3tal config wizard`

Launches an interactive wizard to configure or update variables in `/etc/m3tal/.env`. This guided process makes it easy to set up your M3TAL instance.

**Usage Example:**
```bash
m3tal config wizard
# This will prompt you for configuration details:
#
#    M3TAL Configuration Wizard
#    ---------------------------
#    Current value for DOMAIN [localhost]: myhome.m3tal.dev
#    Set DASHBOARD_EXPOSE_MODE (local/traefik) [local]: traefik
#    ...
#    Configuration saved to /etc/m3tal/.env
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env`. This command is useful for quick, non-interactive adjustments.

**Usage Example:**
```bash
m3tal config set DOMAIN myhomelab.dev
# Output:
# Set DOMAIN=myhomelab.dev in /etc/m3tal/.env.
```

### `m3tal config get KEY`

Reads and displays the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config get DASHBOARD_EXPOSE_MODE
# Output:
# traefik
```

### `m3tal config scan`

Lists all known environment variables across all configured Docker stacks (based on `*-compose.yml` files in `/docker/`) and their default values, if applicable. This helps you understand all possible configuration options.

**Usage Example:**
```bash
m3tal config scan
# Output:
# DASHBOARD_PORT (default: 8082)
# DASHBOARD_EXPOSE_MODE (default: local)
# DOMAIN (default: localhost)
# PUID (default: 1000)
# PGID (default: 1000)
# ...
```

### `m3tal config list`

Displays the current contents of the `/etc/m3tal/.env` file. This provides a direct view of your active configuration.

**Usage Example:**
```bash
m3tal config list
# Output:
# DASHBOARD_PORT=8082
# DASHBOARD_EXPOSE_MODE=traefik
# DOMAIN=myhomelab.dev
# PUID=1000
# PGID=1000
# ...
```

## 📊 Dashboard Management (`m3tal dash`)

These commands specifically control the `m3tal-dashboard` container and its credentials.

### `m3tal dashpass [username] [password]`

Updates or sets user passwords for the M3TAL dashboard, stored in `/docker/users.json`. If no arguments are provided, it runs interactively, prompting for a username and password.

**Usage Example (Interactive):**
```bash
m3tal dashpass
# Output:
# Enter username: admin
# Enter new password:
# Confirm new password:
# Password for 'admin' updated successfully in /docker/users.json.
```

**Usage Example (Non-Interactive):**
```bash
m3tal dashpass admin newSecurePassword123
# Output:
# Password for 'admin' updated successfully in /docker/users.json.
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the `m3tal-dashboard` container with the appropriate override based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal dash up
# Output:
# Pulling latest dashboard compose files... Done.
# Using DASHBOARD_EXPOSE_MODE: traefik
# Starting m3tal-dashboard...
# Container m3tal-dashboard started.
# Access the dashboard at http://dash.myhomelab.dev
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash down
# Output:
# Stopping m3tal-dashboard...
# Container m3tal-dashboard stopped and removed.
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. This is useful after making configuration changes to `/etc/m3tal/.env` that specifically affect the dashboard.

**Usage Example:**
```bash
m3tal dash restart
# Output:
# Restarting m3tal-dashboard...
# Container m3tal-dashboard restarted.
```

### `m3tal dash logs`

Streams the logs from the `m3tal-dashboard` container, useful for debugging dashboard-specific issues.

**Usage Example:**
```bash
m3tal dash logs
# Output:
# Attaching to m3tal-dashboard
# m3tal-dashboard | 2023-10-27 10:30:05 INFO: Dashboard server started on 0.0.0.0:8082
# m3tal-dashboard | 2023-10-27 10:30:05 INFO: Connecting to API at http://host.docker.internal:8080
# ... (Ctrl+C to exit)
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

**Usage Example:**
```bash
m3tal dash status
# Output:
# m3tal-dashboard: Up (running)
```

## 🐳 Stack Management (`m3tal`)

These commands manage all Docker Compose-defined stacks within the `/docker/` directory. This is where you deploy your applications.

### Deployment Lifecycle — Day 2 Operations

To deploy a new application or service (a "stack"):
1.  **Place your compose file**: Create or copy your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory (which is symlinked to `/opt/m3tal/stack/`).
2.  **Configure environment variables**: Ensure all necessary environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value`.
3.  **Start all stacks**: Run `m3tal up` to bring up your new stack along with all others.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or updates all defined services, including M3TAL's routing infrastructure (`routing-compose.yml`) and any user-defined applications.

**Usage Example:**
```bash
m3tal up
# Output:
# Bringing up routing-traefik ... done
# Bringing up ollama ... done
# Bringing up my-app ... done
# All stacks deployed and running.
```

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes defined by your compose files.

**Usage Example:**
```bash
m3tal down
# Output:
# Stopping routing-traefik ... done
# Stopping ollama ... done
# Stopping my-app ... done
# All stacks stopped and removed.
```

### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view for overall system monitoring.

**Usage Example:**
```bash
m3tal logs
# Output:
# Attaching to traefik, ollama, m3tal-dashboard
# traefik      | time="2023-10-27T10:35:01Z" level=info msg="Configuration loaded from file."
# ollama       | time="2023-10-27T10:35:02Z" level=info msg="Ollama server is listening on 0.0.0.0:11434"
# m3tal-dashboard | 2023-10-27 10:35:03 INFO: Dashboard server heartbeat.
# ... (Ctrl+C to exit)
```

## 🛠️ Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. This daemon is crucial for the `m3tal` CLI to interact with Docker and the internal state.

### `systemctl status m3tal-api`

Checks the current status of the M3TAL API daemon.

**Usage Example:**
```bash
systemctl status m3tal-api
# Output:
# ● m3tal-api.service - M3TAL API Daemon
#      Loaded: loaded (/etc/systemd/system/m3tal-api.service; enabled; vendor preset: enabled)
#      Active: active (running) since Fri 2023-10-27 10:00:00 UTC; 1h ago
#    Main PID: 1234 (m3tal-api)
#       Tasks: 8 (limit: 4615)
#      Memory: 25.0M
#         CPU: 1.2s
#      CGroup: /system.slice/m3tal-api.service
#              └─1234 /usr/bin/m3tal-api
# Oct 27 10:00:00 m3talhost systemd[1]: Started M3TAL API Daemon.
# Oct 27 10:00:05 m3talhost m3tal-api[1234]: API daemon listening on :8080
```

### `systemctl restart m3tal-api`

Restarts the M3TAL API daemon. Use this after making manual changes to `/etc/m3tal/.env` that affect the API daemon (though `m3tal config` commands often handle this automatically).

**Usage Example:**
```bash
sudo systemctl restart m3tal-api
# Output: (No direct output, check status or logs)
```

### `journalctl -u m3tal-api -f`

Streams logs from the M3TAL API daemon, providing real-time insights into its operations and any errors.

**Usage Example:**
```bash
sudo journalctl -u m3tal-api -f
# Output:
# -- Logs begin at ...
# Oct 27 10:00:05 m3talhost m3tal-api[1234]: API daemon listening on :8080
# Oct 27 11:15:20 m3talhost m3tal-api[1234]: INFO: Docker client connected.
# ... (Ctrl+C to exit)
```

## 🐳 Direct Docker / Compose Fallback

While `m3tal` commands abstract Docker Compose, you can always use direct `docker compose` commands as a fallback for advanced debugging or specific scenarios. All M3TAL Docker Compose files reside in the `/docker/` directory (symlinked from `/opt/m3tal/stack/`).

### View running containers for all stacks:

```bash
cd /docker/
sudo docker compose -f *-compose.yml ps
# Output:
# NAME                IMAGE                                COMMAND                  SERVICE             CREATED             STATUS              PORTS
# m3tal-dashboard     ghcr.io/jakej985-rgb/m3tal-godash ... python3 server.py        m3tal-dashboard     2 hours ago         Up 2 hours          8082/tcp
# ollama              ollama/ollama:latest                 /bin/ollama              ollama              2 hours ago         Up 2 hours          0.0.0.0:11434->11434/tcp
# routing-traefik-1   traefik:latest                       /entrypoint.sh traefik   traefik             2 hours ago         Up 2 hours          0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 127.0.0.1:8081->8080/tcp
```

### Bring up a specific stack (e.g., `routing-compose.yml`):

```bash
cd /docker/
sudo docker compose -f routing-compose.yml up -d
# Output:
# [+] Running 2/2
#  ✔ Container routing-traefik-1   Started
#  ✔ Container routing-cloudflared Started
```

### View logs for a specific container (e.g., `m3tal-dashboard`):

```bash
sudo docker logs m3tal-dashboard -f
# Output:
# 2023-10-27 10:30:05 INFO: Dashboard server started on 0.0.0.0:8082
# ...
```

## 🗺️ M3TAL Port Map

| Port | Service                                      | Access                                  |
| :--- | :------------------------------------------- | :-------------------------------------- |
| 80   | Traefik HTTP entry point                     | Public (via Traefik, if `routing-compose.yml` is active) |
| 443  | Traefik HTTPS entry point                    | Public (via Traefik, if `routing-compose.yml` is active and TLS is configured) |
| 8080 | M3TAL API daemon (Go)                        | Host-local (accessed by dashboard and Traefik via `host.docker.internal`) |
| 8081 | Traefik dashboard (admin UI for Traefik)     | Host-local only (`127.0.0.1:8081`)      |
| 8082 | M3TAL Dashboard                              | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## 🧭 Traefik Routing Architecture

Traefik, deployed via `routing-compose.yml`, acts as the central reverse proxy for M3TAL.

*   **Entry Points**: Binds to port 80 (HTTP) and optionally 443 (HTTPS) on the host.
*   **Service Discovery**: Automatically discovers Docker services by inspecting their labels (e.g., `m3tal-dashboard` when `DASHBOARD_EXPOSE_MODE=traefik`).
*   **File Provider**: Loads dynamic routing configurations from `/docker/dynamic/` which allows for hot-reloading rules without restarting Traefik.
*   **API Routing**: Routes `api.YOUR_DOMAIN` to the host-local M3TAL API daemon (`http://host.docker.internal:8080`) via `/docker/dynamic/api.yml`.

**Example Traefik Static Configuration (`traefik.yml` within its container):**

```yaml
entryPoints:
  web:
    address: ":80"
  websecure: # Example for HTTPS, needs cert resolver and router update
    address: ":443"

providers:
  docker:
    exposedByDefault: false # Only services with traefik.enable=true labels are exposed
    network: proxy # Connects to the 'proxy' network for service discovery
  file:
    directory: /etc/traefik/dynamic # Mapped from /docker/dynamic on the host
    watch: true # Reloads config when files change
```

**Example Dynamic API Routing (`/docker/dynamic/api.yml`):**

```yaml
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)" # Routes requests for api.YOUR_DOMAIN
      service: api
      entryPoints:
        - web # Use the 'web' entrypoint (port 80)

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080" # Targets the host-local M3TAL API daemon
```