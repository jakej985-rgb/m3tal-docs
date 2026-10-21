# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, my mission is to provide you with the definitive guide to harnessing the power of M3TAL. This document serves as a comprehensive command-line interface (CLI) cheat-sheet, detailing every `m3tal` command, its purpose, and practical usage examples.

Dive deep into the core of your M3TAL system, understand its architecture, and master its management with confidence.

---

## 1. M3TAL Installation

The M3TAL CLI binary (`/usr/bin/m3tal`) is the single entry point for all operations. Install it via APT:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 2. Core CLI Commands

### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application, providing a graphical interface for managing your ecosystem.
**Usage Example:**
```bash
m3tal
# OR
m3tal ui
```

### `m3tal headless [port]`
Starts the M3TAL API daemon (`m3tal-api.service`) in headless mode, ideal for home servers or environments without a desktop GUI. If a port is specified, the API will listen on that port; otherwise, it defaults to 8080.
**Usage Example (Default Port):**
```bash
m3tal headless
```
**Usage Example (Custom Port):**
```bash
m3tal headless 9000
```

### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal, offering a quick snapshot of your server's performance.
**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`
Executes a pre-flight health check, verifying critical system components such as Docker connectivity, the validity of your `/etc/m3tal/.env` configuration, and the availability of required ports.
**Usage Example:**
```bash
m3tal doctor
```

## 3. Configuration Management

M3TAL's primary configuration is stored in `/etc/m3tal/.env`. These commands help you manage it.

### `m3tal config wizard`
Launches an interactive wizard to guide you through the configuration of your `/etc/m3tal/.env` file, setting up essential variables for your M3TAL deployment.
**Usage Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`
Sets a single environment variable in `/etc/m3tal/.env` to a specified value. This is useful for precise adjustments.
**Usage Example:**
```bash
m3tal config set DOMAIN myhomelab.local
```

### `m3tal config get KEY`
Reads and prints the current value of a specific environment variable from `/etc/m3tal/.env`.
**Usage Example:**
```bash
m3tal config get DASHBOARD_EXPOSE_MODE
```

### `m3tal config scan`
Lists all environment variables known across all M3TAL stacks and their current values, providing a comprehensive overview of your configuration.
**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`
Displays the current contents of your primary `/etc/m3tal/.env` file, allowing you to review your configuration.
**Usage Example:**
```bash
m3tal config list
```

## 4. Dashboard Management

These commands are specifically designed for managing the M3TAL Dashboard container (`m3tal-dashboard`).

### `m3tal dashpass [username] [password]`
Updates the password for a specified dashboard user. If username and password are omitted, an interactive prompt will guide you. Dashboard credentials are stored in `/docker/users.json`.
**Usage Example (Interactive):**
```bash
m3tal dashpass
```
**Usage Example (Direct):**
```bash
m3tal dashpass admin strongsecurepassword123
```

### `m3tal dash up`
Pulls the latest dashboard Docker Compose configuration from GitHub, then starts the `m3tal-dashboard` container with the appropriate override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.
**Usage Example:**
```bash
m3tal dash up
```

### `m3tal dash down`
Stops and removes the `m3tal-dashboard` container.
**Usage Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`
Restarts the `m3tal-dashboard` container, useful after configuration changes or for basic troubleshooting.
**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`
Streams the logs from the `m3tal-dashboard` container to your terminal, invaluable for debugging.
**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`
Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).
**Usage Example:**
```bash
m3tal dash status
```

## 5. Stack Management

These commands operate on all Docker Compose stacks defined in `/docker/`.

### `m3tal up`
Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`, ensuring all your M3TAL services and user-defined stacks are running in detached mode.
**Usage Example:**
```bash
m3tal up
```

### `m3tal down`
Runs `docker compose down` across all `*-compose.yml` files in `/docker/`, stopping and removing all containers, networks, and volumes defined by your M3TAL stacks.
**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`
Streams aggregated logs from all currently running M3TAL Docker containers, providing a unified view for monitoring.
**Usage Example:**
```bash
m3tal logs
```

## 6. M3TAL System Architecture Deep Dive

Understanding the underlying architecture is key to mastering M3TAL.

### Components Overview
-   **CLI binary** (`/usr/bin/m3tal`): Your primary interface.
-   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port 8080. It manages Docker operations, maintains a state SQLite database, and exposes API routes.
-   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running inside Docker on port 8082, communicating with the API daemon at `http://host.docker.internal:8080`.
-   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services on port 80. It dynamically routes traffic based on Docker labels and file provider configurations.
-   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing secure, zero-config internet access to your services.

### Filesystem Contract
M3TAL adheres to a strict filesystem contract:

| Path | Purpose |
| :------------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env` | The primary configuration file, managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | The SQLite state database, automatically created and managed by the `m3tal-api.service` daemon. |
| `/opt/m3tal/stack/` | The canonical directory for all M3TAL and user-defined Docker Compose files (`*-compose.yml`). |
| `/docker` | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | The credential store for the M3TAL Dashboard, managed by `m3tal dashpass`. |

### Dashboard Access — TWO MODES

The M3TAL Dashboard has two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)
-   **`DASHBOARD_EXPOSE_MODE=local`**
-   Uses the `m3tal-compose.local.yml` override file.
-   Directly binds the dashboard container's internal port (`8082`) to the host's `DASHBOARD_PORT` (defaulting to `8082`).
-   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`. For example, if your server's IP is `192.168.1.100`, you would access it at `http://192.168.1.100:8082`.
-   **Best for:** LAN-only setups, initial configuration, or environments where Traefik is not desired. No Traefik configuration is required.

#### Mode 2: `traefik`
-   **`DASHBOARD_EXPOSE_MODE=traefik`**
-   Uses the `m3tal-compose.traefik.yml` override file.
-   Adds Traefik labels to the dashboard container, instructing Traefik to route traffic for `dash.${DOMAIN}` to the dashboard's internal port (`8082`).
-   **Access via:** `http://dash.YOUR_DOMAIN` (e.g., if `DOMAIN` is set to `myhomelab.local`, access is `http://dash.myhomelab.local`).
-   **Requires:** Traefik to be running via `m3tal up` (`routing-compose.yml`).
-   **Best for:** Domain-based access, integrating into a reverse proxy setup, or when exposing services publicly via Traefik.

### Docker / Compose Runtime
M3TAL leverages **Docker Engine** and **Docker Compose V2** for container orchestration.
-   The `m3tal up` command executes `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory.
-   The `m3tal dash up` command is special: it first downloads the latest dashboard compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the dashboard using the appropriate override based on your `DASHBOARD_EXPOSE_MODE`.
-   **To deploy a new stack:** Simply place your `my-stack-compose.yml` file into `/docker/` and ensure any required variables are set in `/etc/m3tal/.env`. Then run `m3tal up`.

### Deployment Lifecycle — Day 2 Operations
**Installing a new Docker Compose stack:**
1.  Place your `docker-compose.yml` file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  Ensure any environment variables required by your new stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start your new stack alongside all existing M3TAL services.

### Traefik Routing Architecture
Traefik runs as a container (`traefik` in `routing-compose.yml`) and acts as the central reverse proxy.
-   It binds to port 80 (and 443 if configured for HTTPS) on the host as its HTTP entry point.
-   Services are automatically discovered via Docker labels (e.g., the dashboard in `traefik` mode).
-   Dynamic routing rules are loaded from `/docker/dynamic/` (using a file provider, hot-reloading on changes).
    -   **API Routing:** Traefik routes `api.${DOMAIN}` (e.g., `api.myhomelab.local`) directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080` using a dynamic rule in `/docker/dynamic/api.yml`.
    -   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` (e.g., `dash.myhomelab.local`) to the `m3tal-dashboard` container (port 8082) via labels defined in `m3tal-compose.traefik.yml`.

### M3TAL Service Management (systemd)
The M3TAL API daemon, `m3tal-api`, runs as a systemd service.
-   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
-   **Restart the API daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
-   **Stream API daemon logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```

### M3TAL Port Map
| Port | Service | Access |
| :--- | :-------------------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public (if Traefik is running and exposed) |
| 8080 | M3TAL API daemon (Go) | Host-local (accessed by dashboard via `host.docker.internal`, and by Traefik via `dynamic/api.yml`) |
| 8081 | Traefik dashboard | Host-local only (for Traefik management interface) |
| 8082 | M3TAL Dashboard | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## 7. Direct Docker Compose Fallback

While `m3tal` commands streamline operations, you can always interact directly with Docker Compose for individual stacks or advanced debugging.

To manage a specific compose file in `/docker/`, you would use the `docker compose` command with the `-f` flag.

**Example: Starting a specific stack (e.g., `ollama-compose.yml`):**
```bash
docker compose -f /docker/ollama-compose.yml up -d
```

**Example: Stopping the main M3TAL dashboard (if `m3tal dash down` fails):**
```bash
# First, identify the exact compose files used by the dashboard
# (base + override, e.g., m3tal-compose.yml and m3tal-compose.local.yml)
docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down
# Or, if in traefik mode:
# docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml down
```

**Example: Streaming logs for a specific container (e.g., `m3tal-dashboard`):**
```bash
docker logs -f m3tal-dashboard
```

**Example: Listing all running containers:**
```bash
docker ps
```

---

This comprehensive guide should equip you with all the knowledge to navigate and manage your M3TAL ecosystem effectively. Should you encounter any issues, remember the `m3tal doctor` command and the power of `m3tal logs` for quick diagnostics.

*DocSmith, M3TAL Ecosystem Documentation Architect*