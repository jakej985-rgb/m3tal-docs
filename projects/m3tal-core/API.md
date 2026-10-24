Greetings, fellow architects of the digital domain. I am DocSmith, your M3TAL Ecosystem Documentation Architect, and I am here to forge for you the ultimate guide to mastering the M3TAL CLI. This document, `docs/COMMAND_REFERENCE.md`, is your complete cheat-sheet, ensuring you can navigate, configure, and manage your M3TAL environment with precision and power.

---

# M3TAL CLI Command Reference

The M3TAL CLI (`/usr/bin/m3tal`) is the single entry point for all interactions with your M3TAL ecosystem. From launching the UI to orchestrating your Docker stacks, `m3tal` provides a unified and intuitive interface.

## Core M3TAL Commands

These commands manage the core M3TAL application and its underlying infrastructure.

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application, providing a graphical interface for managing your system.

**Usage Example:**

```bash
m3tal
# Or explicitly:
m3tal ui
```

### `m3tal headless [port]`

Starts the M3TAL backend API daemon in headless mode, ideal for servers without a GUI. This will run the API directly in your terminal, making it accessible via its configured port (default 8080).

**Usage Example:**

```bash
m3tal headless
# To run on a specific port:
m3tal headless 5050
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. A quick snapshot of your system's health.

**Usage Example:**

```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check on your M3TAL installation. This includes verifying Docker connectivity, validating the `/etc/m3tal/.env` file, and checking for port availability. Crucial for troubleshooting.

**Usage Example:**

```bash
m3tal doctor
```

## Configuration Management (`m3tal config`)

The `m3tal config` sub-commands allow you to manage the primary configuration file located at `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended way to set up your environment.

**Usage Example:**

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env`.

**Usage Example:**

```bash
m3tal config set DOMAIN myhomelab.com
m3tal config set PUID 1000
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**

```bash
m3tal config get PUID
```

### `m3tal config scan`

Lists all environment variables that M3TAL recognizes, including those used by various stacks and their default values. This provides a comprehensive overview of configurable parameters.

**Usage Example:**

```bash
m3tal config scan
```

### `m3tal config list`

Displays the current contents of your `/etc/m3tal/.env` file.

**Usage Example:**

```bash
m3tal config list
```

## Dashboard Management (`m3tal dash`)

These commands are specifically designed to manage the M3TAL Dashboard container.

### `m3tal dashpass [username] [password]`

Updates a user's password for the M3TAL Dashboard. If `username` and `password` are omitted, the command will prompt you interactively. Manages `/docker/users.json`.

**Usage Examples:**

```bash
# Interactive mode:
m3tal dashpass

# Direct mode (e.g., for 'admin' user):
m3tal dashpass admin newStrongDashboardPassword
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the M3TAL Dashboard container using the appropriate override based on `DASHBOARD_EXPOSE_MODE` in your `.env`.

**Usage Example:**

```bash
m3tal dash up
```

### `m3tal dash down`

Stops and removes the M3TAL Dashboard container.

**Usage Example:**

```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the M3TAL Dashboard container. Useful after configuration changes.

**Usage Example:**

```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams real-time logs from the M3TAL Dashboard container to your terminal. Essential for debugging.

**Usage Example:**

```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current operational status of the M3TAL Dashboard container.

**Usage Example:**

```bash
m3tal dash status
```

## Docker Stack Management (`m3tal up`, `m3tal down`, `m3tal logs`)

These commands interact with all Docker Compose stacks defined in your `/docker/` directory.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This brings up all your configured services and stacks.

**Usage Example:**

```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This stops and removes all services and networks associated with your M3TAL stacks.

**Usage Example:**

```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running M3TAL Docker containers. This provides a consolidated view of your ecosystem's activity.

**Usage Example:**

```bash
m3tal logs
```

---

# M3TAL System Architecture: The Ground Truth

Understanding the underlying architecture is key to effective management. M3TAL is built as a modular ecosystem of services.

### Components

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary, installed via APT, serving as the user's primary interface.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port `8080` (host-local). It manages Docker interactions, maintains the SQLite state database, and exposes the API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within Docker on internal port `8082`. It communicates with the host's API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services via domain names on host port `80` and uses a file provider for dynamic routing based on `/docker/dynamic/` configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container providing secure Cloudflare tunnels for zero-config internet access to your services.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract to ensure predictability and ease of management.

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.               |

### Dashboard Access — Two Modes

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. This determines how the dashboard container's port is exposed.

**Mode 1: `local` (Default)**

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the Docker Compose override file: `m3tal-compose.local.yml`.
*   Directly binds the dashboard's internal port (`8082`) to a host port (default `${DASHBOARD_PORT:-8082}`).
*   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **No Traefik required.** This mode works out of the box on a home server and is ideal for LAN-only setups, first-time users, and local testing.

```yaml
# m3tal-compose.local.yml (applied when DASHBOARD_EXPOSE_MODE=local)
services:
  m3tal-dashboard:
    ports:
      - "${DASHBOARD_PORT:-8082}:8082" # Direct port binding
```

**Mode 2: `traefik`**

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the Docker Compose override file: `m3tal-compose.traefik.yml`.
*   Adds specific Traefik labels to the dashboard service, allowing Traefik to automatically discover and route traffic to it.
*   **Access via:** `http://dash.DOMAIN` (assuming Traefik is running and configured, e.g., via `m3tal up`).
*   **Requires Traefik:** Best suited for domain-based setups and environments with multiple services behind a reverse proxy.

```yaml
# m3tal-compose.traefik.yml (applied when DASHBOARD_EXPOSE_MODE=traefik)
services:
  m3tal-dashboard:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`dash.${DOMAIN:-localhost}`)" # Routes dash.DOMAIN to this service
      - "traefik.http.routers.dashboard.entrypoints=web"
      - "traefik.http.services.dashboard.loadbalancer.server.port=8082"
      - "traefik.docker.network=proxy"
```

The base `m3tal-compose.yml` provides the fundamental service definition for the dashboard:

```yaml
# m3tal-compose.yml (Base Dashboard Compose)
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

M3TAL is built on **Docker Engine** and **Docker Compose V2**. These are hard dependencies for the system to function.

*   The `m3tal up` command intelligently executes `docker compose up` across all `*-compose.yml` files it finds within the `/docker/` directory.
*   The `m3tal dash up` command specifically manages the dashboard container by downloading its latest compose configuration and starting it with the appropriate override based on your `DASHBOARD_EXPOSE_MODE`.
*   You can extend M3TAL by simply placing your own Docker Compose files (e.g., `my-app-compose.yml`) into the `/docker/` directory.

### Deployment Lifecycle — Day 2 Operations

Integrating new services into your M3TAL ecosystem is straightforward:

1.  **Place your compose file:** Copy your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  **Configure environment variables:** Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for direct assignment.
3.  **Start all stacks:** Run `m3tal up` to deploy your new service along with all other M3TAL-managed stacks.

### Traefik Routing Architecture

Traefik acts as the intelligent ingress for your M3TAL services. It's deployed as a container via `routing-compose.yml`.

*   **Entry Point:** Traefik binds host port `80` (and `443` if HTTPS is configured) as its primary HTTP entry point.
*   **Service Discovery:** It automatically discovers Docker services configured with appropriate labels.
*   **Dynamic Configuration:** Traefik loads additional, hot-reloading dynamic configuration from `/docker/dynamic/` via its file provider.
*   **API Routing:** `api.DOMAIN` is routed directly to the M3TAL API daemon (Go binary) on `http://host.docker.internal:8080` via a static configuration file (`/docker/dynamic/api.yml`).
*   **Dashboard Routing:** `dash.DOMAIN` is routed to the M3TAL Dashboard container using Traefik labels defined in `m3tal-compose.traefik.yml` (only when `DASHBOARD_EXPOSE_MODE=traefik`).

**Traefik Static Configuration (from `traefik.yml` within the Traefik container):**

```yaml
entryPoints:
  web:
    address: ":80" # Binds to host port 80

providers:
  docker:
    exposedByDefault: false # Only expose services with explicit Traefik labels
    network: proxy          # Operates on the 'proxy' Docker network
  file:
    directory: /etc/traefik/dynamic # Watches this directory for dynamic config
    watch: true                   # Hot-reloads changes
```

**Dynamic Routing Example (for the M3TAL API, from `/docker/dynamic/api.yml`):**

```yaml
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)" # Routes traffic for api.YOUR_DOMAIN
      service: api
      entryPoints:
        - web

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080" # Forwards to the M3TAL API daemon on the host
```

### Port Map

This table outlines the default port assignments within the M3TAL ecosystem.

| Port | Service                                | Access                                            |
| :--- | :------------------------------------- | :------------------------------------------------ |
| 80   | Traefik HTTP entry point               | Public (when `routing-compose.yml` is active)     |
| 8080 | M3TAL API daemon (Go)                  | Host-local only (internal communication)          |
| 8081 | Traefik dashboard (for Traefik itself) | Host-local only (e.g., `http://127.0.0.1:8081`)  |
| 8082 | M3TAL Dashboard                        | Direct port (local mode) or via Traefik (traefik mode) |

---

# Service Management with `systemd`

The core M3TAL API daemon runs as a `systemd` service, providing robustness and automatic startup.

### `m3tal-api.service`

This service manages the Go binary that powers the M3TAL API.

*   **Check Status:** See if the API is running and healthy.

    ```bash
    systemctl status m3tal-api
    ```

*   **Restart Service:** Apply configuration changes or recover from issues.

    ```bash
    sudo systemctl restart m3tal-api
    ```

*   **Follow Logs:** Stream real-time logs from the API daemon for debugging.

    ```bash
    journalctl -u m3tal-api -f
    ```

---

# Direct Docker & Compose Commands (Fallback)

While `m3tal` provides a convenient wrapper, you can always fall back to direct Docker and Docker Compose commands. This is useful for advanced debugging or specific scenarios.

**Important Note:** These commands operate directly on the Docker environment. Ensure you understand their implications before bypassing the `m3tal` CLI.

*   **Bring up all M3TAL stacks directly:**

    ```bash
    cd /docker/ # Navigate to the stacks directory
    docker compose -f m3tal-compose.yml -f routing-compose.yml -f other-stack-compose.yml up -d
    # Note: You'll need to manually list all compose files, and apply the correct dashboard override
    ```

*   **Bring up ONLY the dashboard in local mode:**

    ```bash
    cd /docker/ # Ensure you're in the /docker directory
    DASHBOARD_PORT=8082 docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d m3tal-dashboard
    ```

*   **Stop all M3TAL stacks directly:**

    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f routing-compose.yml -f other-stack-compose.yml down
    ```

*   **View logs for a specific container (e.g., m3tal-dashboard):**

    ```bash
    docker logs -f m3tal-dashboard
    ```

*   **Check status of all Docker containers:**

    ```bash
    docker ps -a
    ```

*   **Inspect the 'proxy' network:**

    ```bash
    docker network inspect proxy
    ```

---

# M3TAL APT Installation

For a seamless installation of the `m3tal` CLI and its core components, follow these steps.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```