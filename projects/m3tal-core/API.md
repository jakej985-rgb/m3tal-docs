# M3TAL CLI Command Reference

Welcome to the M3TAL Ecosystem Command Line Interface (CLI) documentation. This guide provides a comprehensive cheat-sheet for managing your M3TAL instance, from initial setup to day-to-day operations and advanced configuration.

M3TAL is powered by a unified Go binary (`/usr/bin/m3tal`) that acts as the single entrypoint for all interactions, abstracting away complex Docker Compose commands and system daemon management.

---

## M3TAL CLI Commands

### Core Operations

#### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application. This command initiates the GUI experience, ideal for desktop environments.

**Usage Example:**
```bash
m3tal
```

#### `m3tal headless [port]`
Starts M3TAL in headless mode. This is the recommended mode for home servers, cloud instances, or any environment without a graphical desktop. The M3TAL API daemon will listen on the specified port (default: 8080).

**Usage Examples:**
```bash
m3tal headless         # Starts headless mode on the default port (8080)
m3tal headless 9000    # Starts headless mode, making the API daemon available on port 9000
```

#### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Useful for quick health checks and performance monitoring.

**Usage Example:**
```bash
m3tal stats
```

#### `m3tal doctor`
Performs a pre-flight health check of your M3TAL installation. This command verifies Docker connectivity, validates your `/etc/m3tal/.env` file, and checks for port availability, helping diagnose common setup issues.

**Usage Example:**
```bash
m3tal doctor
```

### Configuration Management (`m3tal config`)

M3TAL's primary configuration is stored in `/etc/m3tal/.env`. The `m3tal config` subcommand provides tools to manage this file.

#### `m3tal config wizard`
Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the easiest way to set up M3TAL after installation.

**Usage Example:**
```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`
Sets a single environment variable in `/etc/m3tal/.env`.

**Usage Examples:**
```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
m3tal config set DOMAIN myhomelab.com
```

#### `m3tal config get KEY`
Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config get DASHBOARD_PORT
```

#### `m3tal config scan`
Lists all known environment variables across all M3TAL stacks, including default values and those defined in your `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config scan
```

#### `m3tal config list`
Prints the current contents of your `/etc/m3tal/.env` file to the terminal.

**Usage Example:**
```bash
m3tal config list
```

### Dashboard Management (`m3tal dash`)

The `m3tal dash` subcommand specifically manages the M3TAL Dashboard container (`m3tal-dashboard`).

#### `m3tal dashpass [username] [password]`
Updates the password for a dashboard user. If `username` and `password` are omitted, it will launch an interactive prompt. The user credentials are stored in `/docker/users.json`.

**Usage Examples:**
```bash
m3tal dashpass admin newSecurePassword123 # Sets password for 'admin' user
m3tal dashpass                           # Launches interactive prompt to set user and password
```

#### `m3tal dash up`
Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub and then starts the `m3tal-dashboard` container with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal dash up
```

#### `m3tal dash down`
Stops and removes the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash down
```

#### `m3tal dash restart`
Restarts the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash restart
```

#### `m3tal dash logs`
Streams the logs from the `m3tal-dashboard` container to your terminal.

**Usage Example:**
```bash
m3tal dash logs
```

#### `m3tal dash status`
Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

**Usage Example:**
```bash
m3tal dash status
```

### Stack Management (`m3tal up`, `m3tal down`, `m3tal logs`)

These commands manage all Docker Compose stacks deployed in your `/docker/` directory.

#### `m3tal up`
Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all defined services in your M3TAL ecosystem.

**Usage Example:**
```bash
m3tal up           # Starts all services in the foreground
m3tal up -d        # Starts all services in detached mode (recommended for production)
```

#### `m3tal down`
Runs `docker compose down` across all stacks, stopping and removing all containers, networks, and volumes defined in your `*-compose.yml` files within `/docker/`.

**Usage Example:**
```bash
m3tal down
```

#### `m3tal logs`
Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a unified view of your entire ecosystem's output.

**Usage Example:**
```bash
m3tal logs         # Shows logs from all containers
m3tal logs -f      # Follows (streams) logs in real-time
```

---

## M3TAL System Architecture & Management

### M3TAL Ecosystem Components

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary, installed via APT, serving as the sole interface for M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on host port `8080`. It manages Docker interactions, maintains the SQLite state database at `/var/lib/m3tal/state.db`, and exposes RESTful API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container. It runs internally on port `8082` and communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services on host port `80` (and `443` for HTTPS, if configured) and uses a file provider for dynamic routing based on domain names.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for establishing secure, zero-config internet access to your M3TAL services.

### Filesystem Contract

M3TAL adheres to a strict filesystem layout for robust operation.

| Path                         | Purpose                                                                                                                                                                                                                                                                             |
| :--------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | The primary configuration file for your M3TAL instance. All core environment variables are defined here. This file is managed by `m3tal config wizard` and `m3tal config set`.                                                                                                       |
| `/var/lib/m3tal/state.db`    | The SQLite state database used by the M3TAL API daemon to persist application state, metrics, and other internal data. This file is automatically created and managed by the API daemon.                                                                                                |
| `/opt/m3tal/stack/`          | The canonical directory containing M3TAL's core Docker Compose files (`m3tal-compose.yml`, `routing-compose.yml`, etc.) and Traefik configuration. **Users typically interact with this via the `/docker` symlink.**                                                              |
| `/docker`                    | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path where all Docker Compose files (`*-compose.yml`) for M3TAL's core services and any user-added stacks should reside. `m3tal up` and `m3tal down` operate on files within this directory.                    |
| `/docker/users.json`         | The credential store for the M3TAL Dashboard. This JSON file contains encrypted user accounts and passwords. It is managed exclusively by the `m3tal dashpass` command. **Do not edit manually.**                                                                                   |
| `/docker/dynamic/`           | Directory for Traefik's dynamic configuration files. `routing-compose.yml` automatically mounts this into the Traefik container. Custom `*.yml` files here can configure additional Traefik routes.                                                                                  |
| `/docker/dynamic/api.yml`    | A dynamic Traefik configuration file that routes `api.${DOMAIN}` requests to the M3TAL API daemon (Go binary) running on `http://host.docker.internal:8080`. This is part of the `routing-compose.yml` stack.                                                                      |

### Dashboard Access Modes (Critical)

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration:** Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** This mode utilizes the `m3tal-compose.local.yml` override file, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik instance is required for this mode. It works out-of-the-box on a home server.
*   **Best for:** LAN-only setups, first-time users, and local development or testing environments.

**Example `m3tal-compose.local.yml` snippet:**
```yaml
services:
  m3tal-dashboard:
    ports:
      - "${DASHBOARD_PORT:-8082}:8082"
```

#### Mode 2: `traefik`

*   **Configuration:** Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** This mode utilizes the `m3tal-compose.traefik.yml` override file. This file adds specific Traefik labels to the `m3tal-dashboard` service, enabling Traefik to automatically discover and route requests for `dash.${DOMAIN}` to the dashboard container's internal port `8082`.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.myhomelab.com`). For this to work, the Traefik gateway (`routing-compose.yml`) must be running via `m3tal up`.
*   **Requirements:** Requires Traefik to be active and properly configured. `DOMAIN` must be set in `/etc/m3tal/.env`.
*   **Best for:** Domain-based setups, environments with multiple services behind a reverse proxy, and internet-facing deployments.

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

### Docker / Compose Runtime Explained

M3TAL relies heavily on **Docker Engine** and **Docker Compose V2** as hard dependencies.

*   The `m3tal up` command intelligently scans the `/docker/` directory and executes `docker compose up -d` (or similar) across all `*-compose.yml` files found. This includes M3TAL's core stacks (`m3tal-compose.yml`, `routing-compose.yml`) and any user-defined stacks.
*   The `m3tal dash up` command is a specialized operation. It ensures that the latest dashboard Docker Compose configurations are downloaded from GitHub, then starts the dashboard container using the appropriate override (`.local.yml` or `.traefik.yml`) based on `DASHBOARD_EXPOSE_MODE`.
*   **User Stacks:** To add new services to your M3TAL ecosystem, simply place your Docker Compose file (e.g., `my-service-compose.yml`) into the `/docker/` directory. M3TAL will automatically pick it up on the next `m3tal up` command.

### Deployment Lifecycle — Day 2 Operations

Installing and managing new services with M3TAL is streamlined:

1.  **Place your compose file:** Copy your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  **Configure environment variables:** Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive experience or `m3tal config set KEY value` for specific variables.
3.  **Start all stacks:** Run `m3tal up -d` to start all services, including your newly added stack, in detached mode.

### Traefik Routing Architecture

Traefik acts as M3TAL's intelligent reverse proxy, simplifying service exposure.

*   **Deployment:** Traefik is deployed as a Docker container via `routing-compose.yml`.
*   **Entry Points:** It binds host port `80` (and `443` for HTTPS, if configured) as its primary HTTP entry point (`web`).
*   **Service Discovery:** Traefik automatically discovers services by reading Docker labels on running containers (e.g., `m3tal-dashboard` when `DASHBOARD_EXPOSE_MODE=traefik`).
*   **Dynamic Configuration:** It loads additional routing rules from `/docker/dynamic/` using a file provider, with hot-reloading enabled.
*   **API Routing:** A specific dynamic configuration (`/docker/dynamic/api.yml`) routes `api.${DOMAIN}` to the M3TAL API daemon (Go binary) running on `http://host.docker.internal:8080`.
*   **Dashboard Routing:** If `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels on the `m3tal-dashboard` container configure routing for `dash.${DOMAIN}` to the dashboard's internal port `8082`.

**Example Traefik Static Configuration (`traefik.yml`):**
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

**Example Dynamic API Routing (`/docker/dynamic/api.yml`):**
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

### Service Management with `systemd`

The core M3TAL API daemon runs as a systemd service, `m3tal-api.service`. You can manage it using standard `systemctl` commands.

*   **Check Status:**
    ```bash
    sudo systemctl status m3tal-api
    ```
*   **Restart Service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Follow Logs:**
    ```bash
    sudo journalctl -u m3tal-api -f
    ```

### Port Map

Understanding the port usage is crucial for networking and troubleshooting.

| Port | Service                                | Access                                                               |
| :--- | :------------------------------------- | :------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point               | Public (when Traefik is running, `DASHBOARD_EXPOSE_MODE=traefik`)    |
| 8080 | M3TAL API daemon (Go binary)           | Host-local only (accessed by containers via `host.docker.internal`) |
| 8081 | Traefik dashboard (admin UI)           | Host-local only (access via `http://localhost:8081`)                 |
| 8082 | M3TAL Dashboard (Python/Flask)         | Direct port (when `DASHBOARD_EXPOSE_MODE=local`) or via Traefik    |

---

## Docker Fallback Commands

While the `m3tal` CLI abstracts Docker Compose, it's useful to know the underlying Docker commands for debugging or advanced scenarios. M3TAL utilizes **Docker Engine** and **Docker Compose V2**.

### Managing Individual Containers (e.g., M3TAL Dashboard)

Instead of `m3tal dash up` or `m3tal dash down`:

*   **Start Dashboard (local mode):**
    ```bash
    # Ensure /etc/m3tal/.env is configured with DASHBOARD_EXPOSE_MODE=local
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d m3tal-dashboard
    ```
*   **Start Dashboard (traefik mode):**
    ```bash
    # Ensure /etc/m3tal/.env is configured with DASHBOARD_EXPOSE_MODE=traefik
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```
*   **Stop Dashboard:**
    ```bash
    docker stop m3tal-dashboard
    docker rm m3tal-dashboard
    ```
    or more cleanly using compose:
    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml down m3tal-dashboard # or .traefik.yml
    ```
*   **View Dashboard Logs:**
    ```bash
    docker logs -f m3tal-dashboard
    ```
*   **Check Dashboard Status:**
    ```bash
    docker ps -f name=m3tal-dashboard
    ```

### Managing All Stacks

Instead of `m3tal up` or `m3tal down`:

*   **Start All Stacks:**
    ```bash
    # Note: You'd need to manually list all compose files that `m3tal up` manages.
    # This example includes core M3TAL and routing. Add your custom stack files as needed.
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml -f routing-compose.yml up -d
    ```
*   **Stop All Stacks:**
    ```bash
    # Note: Manually list all compose files.
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml -f routing-compose.yml down
    ```
*   **View Aggregated Logs:**
    ```bash
    docker logs -f $(docker ps -q) # Streams logs from all running containers
    ```

### General Docker Commands

*   **List all running containers:**
    ```bash
    docker ps
    ```
*   **List all containers (including stopped):**
    ```bash
    docker ps -a
    ```
*   **List all Docker networks:**
    ```bash
    docker network ls
    ```
*   **Inspect a container:**
    ```bash
    docker inspect m3tal-dashboard
    ```
*   **Pull a specific image:**
    ```bash
    docker pull ghcr.io/jakej985-rgb/m3tal-godash:debug
    ```

---

## M3TAL APT Installation

To install the M3TAL CLI and API daemon on Debian/Ubuntu-based systems:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```