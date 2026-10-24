# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive guide to interacting with your M3TAL system. This document serves as a complete command-line interface (CLI) cheat-sheet, covering every facet of M3TAL management from initial setup to day-to-day operations and advanced troubleshooting.

## Introduction to M3TAL

M3TAL is a robust, self-hosted Linux server platform designed for seamless deployment and management of your personal cloud services, media stacks, and home automation systems. It abstracts away the complexity of Docker and Docker Compose, offering a unified CLI for configuration, deployment, monitoring, and maintenance.

At its core, M3TAL orchestrates Docker containers using Docker Compose V2, managing configuration via a central `.env` file and providing a Go-based API for advanced control and system introspection.

## Core Concepts

Understanding the foundational architecture of M3TAL is crucial for effective system management.

### M3TAL System Architecture

*   **CLI binary (`/usr/bin/m3tal`)**: The single entry point for all M3TAL operations, installed via APT. This Go binary acts as the user's interface to the entire ecosystem.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a systemd service on port `8080`. It's responsible for managing Docker, interacting with the SQLite state database (`/var/lib/m3tal/state.db`), and exposing API routes for the dashboard and other internal components.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask container running internally on port `8082`. It communicates with the `m3tal-api.service` at `http://host.docker.internal:8080` to display system status, manage stacks, and provide a user-friendly interface.
*   **Traefik gateway (`routing-compose.yml`)**: A reverse proxy container that exposes services by domain name on port `80` (HTTP) and `443` (HTTPS). It dynamically discovers services via Docker labels and loads additional configuration from file providers.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, enabling secure, zero-config internet access to your services without exposing ports directly.

### Filesystem Contract

The following paths are central to M3TAL's operation and configuration:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical stack directory, containing compose files and Traefik config. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.           |

### Dashboard Access Modes

The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: M3TAL uses the `m3tal-compose.local.yml` override file, which directly binds the dashboard container's internal port `8082` to a host port (default `8082`).
*   **Access**: Navigate to `http://YOUR_HOST_IP:8082` or `http://localhost:8082` in your web browser.
*   **Requirements**: No Traefik instance is required. This mode works out-of-the-box on a home server.
*   **Best for**: LAN-only setups, first-time users, initial configuration, and local testing.

**Example `m3tal-compose.local.yml` snippet:**

```yaml
services:
  m3tal-dashboard:
    ports:
      - "${DASHBOARD_PORT:-8082}:8082"
```

#### Mode 2: `traefik`

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: M3TAL uses the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the dashboard container. Traefik (running via `routing-compose.yml`) then routes `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access**: Navigate to `http://dash.YOUR_DOMAIN` in your web browser (e.g., `http://dash.myhomelab.com`). Traefik must be running (`m3tal up`) for this mode to function.
*   **Requirements**: A running Traefik instance (`routing-compose.yml` stack) is essential. The `DOMAIN` variable must be correctly configured in `/etc/m3tal/.env`.
*   **Best for**: Domain-based setups, systems with multiple services behind a reverse proxy, and integration with dynamic DNS or Cloudflare.

**Example `m3tal-compose.traefik.yml` snippet:**

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

**Base `m3tal-compose.yml` (common to both modes):**

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
      - TZ=America/Denver
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

### Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its underlying containerization platform. These are hard dependencies for M3TAL's operation.

*   The `m3tal up` command orchestrates all Docker Compose stacks by running `docker compose up` across every `*-compose.yml` file found in the `/docker/` directory.
*   The `m3tal dash up` command is a specialized operation that:
    1.  Downloads the latest dashboard compose configuration (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from the official GitHub repository.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the base compose file and the appropriate override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).
*   User-defined Docker Compose stacks should be placed in the `/docker/` directory. Any `*-compose.yml` file in this directory will be automatically included in `m3tal up` operations.

### Deployment Lifecycle — Day 2 Operations

Installing and managing new Docker Compose stacks with M3TAL is streamlined:

1.  **Place your compose file**: Copy your custom Docker Compose file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
2.  **Configure environment variables**: Ensure all necessary environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for specific variables.
3.  **Start all stacks**: Run `m3tal up -d` to bring up all defined services, including your new stack, in the background.

### Traefik Routing Architecture

Traefik acts as M3TAL's intelligent reverse proxy, managing external access to your services. It is deployed as a Docker container via `routing-compose.yml`.

*   **Entry Points**: Traefik binds to port `80` (HTTP) and `443` (HTTPS) on the host system, serving as the primary entry point for web traffic.
*   **Service Discovery**: It automatically discovers and routes traffic to Docker containers based on labels defined in their compose files (Docker provider).
*   **Dynamic Configuration**: Traefik loads additional routing rules from static YAML files in `/etc/traefik/dynamic` (file provider), which hot-reload on changes.
*   **API Daemon Routing**: It routes `api.${DOMAIN}` (e.g., `api.myhomelab.com`) to the M3TAL API daemon running on the host at `http://host.docker.internal:8080` via a configuration file in `dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE` is `traefik`, it routes `dash.${DOMAIN}` to the `m3tal-dashboard` container, leveraging Traefik labels within `m3tal-compose.traefik.yml`.

**Traefik static configuration (`traefik.yml`) example:**

```yaml
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

**Dynamic routing example for API (`dynamic/api.yml`):**

```yaml
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
          - url: "http://host.docker.internal:8080"
```

### Service Management (Systemd)

The core M3TAL API daemon runs as a dedicated systemd service, `m3tal-api.service`. This service is automatically started on boot and managed by systemd.

*   **Status Check**: `systemctl status m3tal-api`
*   **Restart**: `sudo systemctl restart m3tal-api`
*   **Log Streaming**: `sudo journalctl -u m3tal-api -f`

### Port Map

| Port | Service                            | Access                                      |
| :--- | :--------------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point           | Public (when Traefik is running)            |
| 8080 | M3TAL API daemon (Go)              | Host-local (internal communication)         |
| 8081 | Traefik dashboard                  | Host-local only (`127.0.0.1:8081`)          |
| 8082 | M3TAL Dashboard container (Python) | Direct port (local mode) or via Traefik (traefik mode) |

---

## M3TAL Installation

The M3TAL CLI binary is installed via a standard APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## M3TAL CLI Commands

This section provides a comprehensive cheat-sheet for every M3TAL command, complete with descriptions and real-world usage examples.

### Core Application Management

#### `m3tal` or `m3tal ui`

Launches the M3TAL Linux Desktop Application. This command is typically used when M3TAL is installed on a desktop environment, providing a graphical user interface (GUI).

*   **Usage**: `m3tal` or `m3tal ui`
*   **Example**:
    ```bash
    m3tal
    ```

#### `m3tal headless [port]`

Starts the M3TAL API daemon in headless mode, ideal for server environments without a graphical interface. It runs the API service directly in the terminal, binding to a specified port.

*   **Usage**: `m3tal headless [port]`
*   **Example**:
    ```bash
    m3tal headless # Starts the API on default port 8080
    m3tal headless 9000 # Starts the API on port 9000
    ```

#### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to the terminal. Provides a quick snapshot of system health and performance.

*   **Usage**: `m3tal stats`
*   **Example**:
    ```bash
    m3tal stats
    ```

#### `m3tal doctor`

Performs a pre-flight health check of the M3TAL environment. This command verifies critical dependencies and configurations, such as Docker connectivity, `/etc/m3tal/.env` validity, and port availability.

*   **Usage**: `m3tal doctor`
*   **Example**:
    ```bash
    m3tal doctor
    ```

### Configuration Management

M3TAL's configuration is primarily managed through the `/etc/m3tal/.env` file. These commands facilitate its management.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended method for initial setup and major configuration changes.

*   **Usage**: `m3tal config wizard`
*   **Example**:
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`

Sets or updates a single environment variable within `/etc/m3tal/.env`. This command is useful for quick, targeted configuration changes.

*   **Usage**: `m3tal config set KEY VALUE`
*   **Example**:
    ```bash
    m3tal config set DOMAIN myhomelab.com
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

*   **Usage**: `m3tal config get KEY`
*   **Example**:
    ```bash
    m3tal config get PUID
    m3tal config get DASHBOARD_PORT
    ```

#### `m3tal config scan`

Scans all M3TAL-managed Docker Compose files and system components to list all recognized environment variables. It displays their current values from `/etc/m3tal/.env` and their default values.

*   **Usage**: `m3tal config scan`
*   **Example**:
    ```bash
    m3tal config scan
    ```

#### `m3tal config list`

Prints the entire content of the primary configuration file, `/etc/m3tal/.env`, to the terminal.

*   **Usage**: `m3tal config list`
*   **Example**:
    ```bash
    m3tal config list
    ```

### Dashboard Management

Commands specifically for controlling and configuring the M3TAL Dashboard container.

#### `m3tal dashpass [username] [password]`

Updates the password for a dashboard user. If `username` and `password` are omitted, the command will prompt for them interactively. This manages the `/docker/users.json` file.

*   **Usage**: `m3tal dashpass [username] [password]`
*   **Example**:
    ```bash
    m3tal dashpass # Interactive prompt for username and password
    m3tal dashpass admin MySecureNewPassword123! # Sets 'admin' user password
    ```

#### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files from GitHub and then starts the `m3tal-dashboard` container, applying the correct expose mode based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

*   **Usage**: `m3tal dash up`
*   **Example**:
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

*   **Usage**: `m3tal dash down`
*   **Example**:
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. This is useful after making configuration changes that require the dashboard to reinitialize.

*   **Usage**: `m3tal dash restart`
*   **Example**:
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`

Streams the real-time logs from the `m3tal-dashboard` container to your terminal. Useful for debugging dashboard issues.

*   **Usage**: `m3tal dash logs`
*   **Example**:
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`

Shows the current operational status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

*   **Usage**: `m3tal dash status`
*   **Example**:
    ```bash
    m3tal dash status
    ```

### Docker Stack Management

These commands manage all Docker Compose stacks defined in the `/docker/` directory.

#### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files located in `/docker/`. This command brings up or recreates all services defined across your M3TAL ecosystem. It's recommended to run it in detached mode (`-d`).

*   **Usage**: `m3tal up [OPTIONS]`
*   **Example**:
    ```bash
    m3tal up -d # Start all stacks in the background
    m3tal up --force-recreate # Force recreation of all containers
    ```

#### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command gracefully stops and removes all containers, networks, and volumes associated with your M3TAL stacks.

*   **Usage**: `m3tal down`
*   **Example**:
    ```bash
    m3tal down
    ```

#### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of activity across your entire stack.

*   **Usage**: `m3tal logs [OPTIONS]`
*   **Example**:
    ```bash
    m3tal logs -f # Follow (stream) logs in real-time
    m3tal logs --tail 100 # Show last 100 lines from all services
    ```

---

## Advanced Topics

### Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) is managed by `systemd`. You can interact with this service directly using `systemctl` and `journalctl`.

*   **Check the status of the M3TAL API service**:
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart the M3TAL API service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream real-time logs from the M3TAL API service**:
    ```bash
    sudo journalctl -u m3tal-api -f
    ```

### Direct Docker Compose Fallback

M3TAL's CLI is a wrapper around Docker Engine and Docker Compose V2. In scenarios where you need direct Docker interaction or want to troubleshoot specific services, you can use `docker` and `docker compose` commands directly. Remember that all M3TAL-managed compose files reside in the `/docker/` directory.

*   **List all Docker containers (including M3TAL's)**:
    ```bash
    sudo docker ps -a
    ```
*   **Inspect a specific container**:
    ```bash
    sudo docker inspect m3tal-dashboard
    ```
*   **Stream logs from a specific M3TAL-managed container**:
    ```bash
    sudo docker logs -f m3tal-dashboard
    sudo docker logs -f traefik
    ```
*   **Start a specific user-defined stack directly (if `m3tal up` encounters an issue)**:
    Assuming you have `/docker/my-app-compose.yml`:
    ```bash
    cd /docker/
    sudo docker compose -f my-app-compose.yml up -d
    ```
*   **Bring up M3TAL's core routing and dashboard components manually (e.g., for troubleshooting `m3tal up` itself)**:
    This example starts Traefik and the dashboard in `local` mode:
    ```bash
    cd /docker/
    # Ensure the 'proxy' network exists, or create it if missing:
    sudo docker network create proxy || true
    sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f m3tal-compose.local.yml up -d
    ```
    Or in `traefik` mode (requires `DASHBOARD_EXPOSE_MODE=traefik` in `.env` for labels to be active):
    ```bash
    cd /docker/
    sudo docker network create proxy || true
    sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f m3tal-compose.traefik.yml up -d
    ```
*   **Stop specific containers or stacks directly**:
    ```bash
    sudo docker stop m3tal-dashboard
    cd /docker/
    sudo docker compose -f my-app-compose.yml down
    ```

By understanding both the `m3tal` CLI and its underlying Docker components, you gain full control over your M3TAL ecosystem.