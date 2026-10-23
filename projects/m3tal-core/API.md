# M3TAL CLI Command Reference

Welcome, Architect. This document serves as the definitive command-line interface (CLI) cheat-sheet for the M3TAL Ecosystem, codenamed DocSmith. It provides a comprehensive guide to every `m3tal` command, complete with real-world usage examples, system architecture insights, and essential operational procedures.

The M3TAL CLI, powered by a unified Go binary installed at `/usr/bin/m3tal`, is your single point of entry for managing your entire home server infrastructure. From system diagnostics to Docker stack orchestration and configuration management, M3TAL streamlines complex operations into intuitive commands.

---

## M3TAL CLI Commands

### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application. This command initiates the graphical user interface for M3TAL, providing a visual dashboard and management tools for your ecosystem directly on your desktop environment.

**Usage Example:**
```bash
m3tal
# Or explicitly:
m3tal ui
```

### `m3tal headless [port]`
Starts M3TAL in headless mode, ideal for home servers and remote deployments without a graphical desktop environment. This launches the core API daemon and exposes it for remote management. If no port is specified, it will default to port 8080 as per the systemd service configuration for `m3tal-api.service`.

**Usage Example:**
```bash
# Start in headless mode on default port 8080
m3tal headless

# Start in headless mode, explicitly on port 8000 (if API daemon configured to listen on it)
m3tal headless 8000
```

### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. This provides a quick overview of your system's health and performance.

**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`
Performs a pre-flight health check of the M3TAL ecosystem. This command verifies critical dependencies and configurations, including Docker connectivity, `/etc/m3tal/.env` validity, and port availability.

**Usage Example:**
```bash
m3tal doctor
```

### `m3tal config wizard`
Launches an interactive wizard to configure the primary M3TAL environment variables stored in `/etc/m3tal/.env`. This is the recommended way to set up or modify core system settings.

**Usage Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`
Sets a single environment variable within `/etc/m3tal/.env`. This allows for direct, non-interactive modification of specific configuration parameters.

**Usage Example:**
```bash
# Set the primary domain for Traefik routing
m3tal config set DOMAIN myhomelab.local

# Configure the dashboard to use Traefik for exposure
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`
Reads and prints the value of a single environment variable from `/etc/m3tal/.env` to the terminal.

**Usage Example:**
```bash
# Retrieve the configured dashboard exposure mode
m3tal config get DASHBOARD_EXPOSE_MODE
```

### `m3tal config scan`
Lists all environment variables that M3TAL uses, including defaults and those from various Docker stacks, providing a comprehensive overview of your ecosystem's configurable parameters.

**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`
Displays the current contents of your primary configuration file, `/etc/m3tal/.env`, showing all set environment variables and their values.

**Usage Example:**
```bash
m3tal config list
```

### `m3tal dashpass [username] [password]`
Updates the M3TAL Dashboard user password. If `username` and `password` arguments are omitted, the command will launch an interactive prompt to guide you through the password update process. Dashboard credentials are stored in `/docker/users.json`.

**Usage Example:**
```bash
# Update password for 'admin' user interactively
m3tal dashpass

# Directly set password for user 'admin' to 'SuperSecurePassword123!'
m3tal dashpass admin SuperSecurePassword123!
```

### `m3tal dash up`
Manages the M3TAL Dashboard container. This command first pulls the latest dashboard compose configuration (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the dashboard container with the appropriate override based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal dash up
```

### `m3tal dash down`
Stops and removes the M3TAL Dashboard container and its associated network resources.

**Usage Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`
Restarts the M3TAL Dashboard container. This is useful for applying configuration changes or resolving minor issues.

**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`
Streams real-time logs from the M3TAL Dashboard container to your terminal, aiding in debugging and monitoring.

**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`
Shows the current operational status of the M3TAL Dashboard container, including its state, uptime, and resource usage.

**Usage Example:**
```bash
m3tal dash status
```

### `m3tal up`
Executes `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all defined M3TAL services and user-defined stacks in detached mode.

**Usage Example:**
```bash
m3tal up
```

### `m3tal down`
Executes `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes defined by your M3TAL services and user-defined stacks.

**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`
Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of activity across your entire ecosystem.

**Usage Example:**
```bash
m3tal logs
```

---

## M3TAL System Architecture

M3TAL operates as a cohesive ecosystem built around Docker, systemd, and a unified Go CLI.

### Components
-   **CLI binary** (`/usr/bin/m3tal`): The central entry point for all M3TAL operations, installed via APT.
-   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, primarily on **port 8080**. It manages Docker interactions, maintains the state database, and provides API routes for the Dashboard and CLI.
-   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within Docker, internally on port 8082. It communicates with the API daemon at `http://host.docker.internal:8080`.
-   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services on host port 80. It uses Docker labels and a file provider for dynamic routing.
-   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing secure, zero-config internet access to your services.

### Filesystem Contract

| Path                          | Purpose                                                         |
| :---------------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`             | Primary configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`     | SQLite state database. Auto-created by the API daemon.          |
| `/opt/m3tal/stack/`           | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                     | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`          | Dashboard credential store. Managed by `m3tal dashpass`.        |

### Dashboard Access — Two Modes

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)
-   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
-   **Mechanism:** M3TAL uses the `m3tal-compose.local.yml` override, which adds a direct port binding to the host: `${DASHBOARD_PORT:-8082}:8082`.
-   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
-   **Requirements:** No Traefik is required. This mode works out-of-the-box, ideal for initial setup, LAN-only environments, or local development.

#### Mode 2: `traefik`
-   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
-   **Mechanism:** M3TAL uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik then routes `dash.${DOMAIN}` to the dashboard on its internal port 8082. **Traefik must be running via `m3tal up` for this mode to function.**
-   **Access:** The dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.myhomelab.local`), where `DOMAIN` is configured in `/etc/m3tal/.env`.
-   **Requirements:** Requires Traefik to be deployed and running via `routing-compose.yml` (part of `m3tal up`) and a configured `DOMAIN` variable. Best for domain-based setups and integrating with other services behind Traefik.

**Dashboard Compose Configuration Snippets:**

```yaml
# m3tal-compose.yml (Base Dashboard Configuration)
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
      - TZ=America/Denver
      - DASHBOARD_SECRET=${DASHBOARD_SECRET:-}
      - STATE_DIR=/docker/state
      - USERS_FILE=/docker/state/config/users.json
      - DASHBOARD_HOST=0.0.0.0
      - DASHBOARD_PORT=8082
      - PYTHONUNBUFFERED=1
      - GO_API_URL=${GO_API_URL:-http://host.docker.internal:8080} # Ensure this targets the API daemon's actual port
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

```yaml
# m3tal-compose.local.yml (Local Mode Override)
services:
  m3tal-dashboard:
    ports:
      - "${DASHBOARD_PORT:-8082}:8082"
```

```yaml
# m3tal-compose.traefik.yml (Traefik Mode Override)
services:
  m3tal-dashboard:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`dash.${DOMAIN:-localhost}`)"
      - "traefik.http.routers.dashboard.entrypoints=web"
      - "traefik.http.services.dashboard.loadbalancer.server.port=8082"
      - "traefik.docker.network=proxy"
```

### Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its core container orchestration tools. These are hard dependencies for the system.

-   The `m3tal up` command abstracts `docker compose up -d`, running it across all `*-compose.yml` files located in the `/docker/` directory. This includes core M3TAL services and any user-defined stacks.
-   The `m3tal dash up` command performs a specialized sequence:
    1.  It first downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from GitHub to ensure your dashboard configuration is up-to-date.
    2.  It then reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Finally, it starts the `m3tal-dashboard` container, applying the appropriate Compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the configured mode.
-   Users can extend M3TAL's functionality by placing their own Docker Compose files (e.g., `my-app-compose.yml`) directly into the `/docker/` directory. These will be automatically included in `m3tal up` and `m3tal down` operations.

### Deployment Lifecycle — Day 2 Operations

To install and integrate a new Docker stack into your M3TAL ecosystem:

1.  **Place Compose File:** Copy your `docker-compose.yml` (or similar, e.g., `my-stack-compose.yml`) into the `/docker/` directory.
    ```bash
    sudo cp ~/my-stack-compose.yml /docker/
    ```
2.  **Configure Environment Variables:** Ensure all necessary environment variables for your new stack are defined in `/etc/m3tal/.env`. Use the interactive wizard or direct set command:
    ```bash
    m3tal config wizard
    # Or
    m3tal config set MY_APP_PORT 9000
    ```
3.  **Start All Stacks:** Run `m3tal up` to deploy your new stack alongside all other M3TAL services.
    ```bash
    m3tal up
    ```

### Traefik Routing Architecture

Traefik acts as the central reverse proxy for M3TAL, deployed as a container via `routing-compose.yml`.

-   **Entry Points:** Traefik binds host port 80 as the primary HTTP entry point (`web`).
-   **Service Discovery:** It automatically discovers Docker services and applies routing rules based on Docker labels on containers (e.g., `m3tal-dashboard` when in `traefik` mode).
-   **Dynamic Configuration:** Traefik also utilizes a file provider, loading dynamic routing configurations from `/docker/dynamic/`. These files are hot-reloaded for immediate application of changes.
-   **API Routing:** Traefik routes `api.DOMAIN` to the M3TAL API daemon running at `http://host.docker.internal:8080` (the `m3tal-api.service`). This is configured via `dynamic/api.yml`.
-   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the `m3tal-dashboard` container based on the labels defined in `m3tal-compose.traefik.yml`.

**Traefik Configuration Snippets:**

```yaml
# traefik.yml (Static Traefik Configuration)
entryPoints:
  web:
    address: ":80"

providers:
  docker:
    exposedByDefault: false
    network: proxy
  file:
    directory: /etc/traefik/dynamic
    watch: true
```

```yaml
# dynamic/api.yml (Dynamic Routing for M3TAL API)
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)"
      service: api
      entryPoints:
        - web

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080" # Routes to the API daemon
```

---

## Service Management — systemd

The core M3TAL API daemon is managed by `systemd` as the `m3tal-api.service`. This allows for robust control over its lifecycle.

-   **Check Status:** Verify if the API daemon is running and healthy.
    ```bash
    sudo systemctl status m3tal-api
    ```
-   **Restart Service:** Restart the API daemon, for example, after updating `/etc/m3tal/.env` or system binaries.
    ```bash
    sudo systemctl restart m3tal-api
    ```
-   **Stream Logs:** View real-time logs from the API daemon for debugging purposes.
    ```bash
    sudo journalctl -u m3tal-api -f
    ```

---

## Port Map

| Port | Service               | Access                                   |
| :--- | :-------------------- | :--------------------------------------- |
| 80   | Traefik HTTP entry point | Public (if `traefik` mode is active)     |
| 8080 | M3TAL API daemon (Go) | Host-local (exposed via Traefik for `api.DOMAIN`) |
| 8081 | Traefik dashboard     | Host-local only (`127.0.0.1:8081`)     |
| 8082 | M3TAL Dashboard       | Direct port (if `local` mode) or via Traefik (if `traefik` mode) |

---

## Fallback: Direct Docker Compose Commands

While the `m3tal` CLI provides a convenient abstraction, you can always interact directly with Docker Compose for more granular control or troubleshooting. These commands operate on the `*-compose.yml` files found in the `/docker/` directory.

-   **Start all stacks in detached mode:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/ollama-compose.yml up -d
    ```
    *(Note: You'll need to list all relevant compose files manually)*

-   **Stop and remove all stacks:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/ollama-compose.yml down
    ```

-   **View logs for all running services:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/ollama-compose.yml logs -f
    ```

-   **List all running Docker containers and their status:**
    ```bash
    sudo docker ps
    ```

-   **List all Docker Compose projects and services:**
    ```bash
    sudo docker compose ls
    sudo docker compose ps
    ```

---

## APT Installation

To install the `m3tal` CLI binary on your Debian/Ubuntu-based system:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```