Greetings, Operator. I am DocSmith, your M3TAL Ecosystem Documentation Architect.

This document serves as your complete CLI cheat-sheet for the M3TAL ecosystem, detailing every command, its function, and practical usage. Master these commands, and you will command your digital infrastructure with the precision of a master craftsman.

---

## M3TAL Ecosystem: Command Reference

The `m3tal` CLI binary (`/usr/bin/m3tal`) is your singular interface to the M3TAL system. It orchestrates the M3TAL API daemon, manages Docker containers, configures your environment, and provides crucial insights into your system's health.

### 1. Installation

To begin your journey, install M3TAL using our APT repository:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### 2. Core M3TAL CLI Commands

The `m3tal` command is your primary tool. Here are its various subcommands:

#### `m3tal` / `m3tal ui`
Launches the M3TAL Linux Desktop Application, providing a graphical interface for interacting with your M3TAL instance.
```bash
m3tal
# OR
m3tal ui
```

#### `m3tal headless [port]`
Starts the M3TAL API daemon in headless mode, ideal for server environments without a graphical interface. It runs the API daemon directly, bypassing systemd. If `port` is omitted, it defaults to `8080`.
```bash
m3tal headless
# Starts the API daemon on default port 8080

m3tal headless 8081
# Starts the API daemon on port 8081
```

#### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. A quick glance at your server's pulse.
```bash
m3tal stats
```

#### `m3tal doctor`
Executes a pre-flight health check to diagnose common issues. It verifies Docker connectivity, validates the `/etc/m3tal/.env` configuration, and checks for port availability, among other critical system components.
```bash
m3tal doctor
```

#### `m3tal config wizard`
Initiates an interactive wizard to guide you through configuring or updating the essential variables within `/etc/m3tal/.env`. This is the recommended method for initial setup.
```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`
Sets a single environment variable within `/etc/m3tal/.env` to a specified value. This is useful for quick, targeted configuration changes.
```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
m3tal config set DOMAIN myhomelab.net
```

#### `m3tal config get KEY`
Retrieves and displays the current value of a specific environment variable from `/etc/m3tal/.env`.
```bash
m3tal config get PUID
# Expected output: 1000
```

#### `m3tal config scan`
Scans and lists all known M3TAL environment variables, including their default values and current values where set. This provides a comprehensive overview of configurable parameters across all stacks.
```bash
m3tal config scan
```

#### `m3tal config list`
Displays the entire contents of the primary configuration file, `/etc/m3tal/.env`, directly in your terminal.
```bash
m3tal config list
```

#### `m3tal dashpass [username] [password]`
Updates the password for a M3TAL Dashboard user. If `username` and `password` are omitted, the command will prompt you interactively. User credentials are stored in `/docker/users.json`.
```bash
m3tal dashpass docsmith_admin securePassword123!
# Sets the password for 'docsmith_admin' to 'securePassword123!'

m3tal dashpass
# Prompts interactively for username and new password
```

#### `m3tal dash up`
Pulls the latest dashboard compose configuration files from GitHub, then starts the `m3tal-dashboard` container. This command intelligently selects the appropriate Docker Compose override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.
```bash
m3tal dash up
```

#### `m3tal dash down`
Stops and removes the `m3tal-dashboard` container.
```bash
m3tal dash down
```

#### `m3tal dash restart`
Restarts the `m3tal-dashboard` container.
```bash
m3tal dash restart
```

#### `m3tal dash logs`
Streams the real-time logs from the `m3tal-dashboard` container to your terminal. Useful for debugging dashboard issues.
```bash
m3tal dash logs
```

#### `m3tal dash status`
Displays the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).
```bash
m3tal dash status
```

#### `m3tal up`
Executes `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command brings up all your defined M3TAL services and user-defined stacks.
```bash
m3tal up
```

#### `m3tal down`
Executes `docker compose down` across all `*-compose.yml` files in the `/docker/` directory, gracefully stopping and removing all associated containers, networks, and volumes.
```bash
m3tal down
```

#### `m3tal logs`
Streams aggregated logs in real-time from all currently running Docker containers managed by M3TAL. This provides a unified view of all your services' output.
```bash
m3tal logs
```

---

### 3. M3TAL System Architecture

Understanding the underlying architecture empowers you to troubleshoot and extend your M3TAL ecosystem effectively.

#### Components
*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary, installed via APT, serving as the single entry point for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port 8080. It manages Docker interactions, maintains the M3TAL state database, and exposes RESTful API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running inside a Docker container, accessible internally on port 8082. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services by domain name on host port 80 (and 443 if configured). It uses a file provider for dynamic routing and Docker labels for service discovery.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing zero-configuration secure internet access to your M3TAL services without requiring open ports.

#### Filesystem Contract

| Path                     | Purpose                                                                                                                                                                                                                                                                                                                        |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | **Primary configuration file.** This file contains all critical environment variables for M3TAL operations. It is meticulously managed by `m3tal config wizard` and `m3tal config set`.                                                                                                                                          |
| `/var/lib/m3tal/state.db`| **SQLite state database.** This database is automatically created and managed by the M3TAL API daemon. It stores internal M3TAL state, including information about running containers, installed stacks, and user configurations. **Do not modify manually.**                                                               |
| `/opt/m3tal/stack/`      | **Canonical stack directory.** This is the internal, definitive location for all M3TAL Docker Compose files (`*-compose.yml`) and Traefik dynamic configuration files.                                                                                                                                                           |
| `/docker`                | **User-facing stack directory.** This is a symbolic link (`/docker -> /opt/m3tal/stack/`). All user-defined Docker Compose stacks and Traefik dynamic configuration should be placed here. This path is used by `m3tal up` and `m3tal down` to discover and manage your services.                                              |
| `/docker/users.json`     | **Dashboard credential store.** This JSON file holds the hashed credentials for users accessing the M3TAL Dashboard. It is managed exclusively by the `m3tal dashpass` command. **Do not modify manually.**                                                                                                                      |

#### Dashboard Access — Two Modes

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. This variable dictates which Docker Compose override file is used when starting the dashboard.

**Mode 1: `local` (Default for home servers)**
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port to the host: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**:
    *   **Via Host IP**: `http://YOUR_HOST_IP:8082` (e.g., `http://192.168.1.100:8082`)
    *   **Via Localhost**: `http://localhost:8082` (if accessing from the host machine itself)
*   **Best for**: LAN-only setups, first-time users, environments without a reverse proxy, and local testing. No Traefik configuration is required for this mode.

**Mode 2: `traefik` (For domain-based access)**
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override. This file adds specific Traefik labels to the `m3tal-dashboard` container, enabling Traefik to automatically discover and route traffic to it. Traefik will route `dash.${DOMAIN}` to the dashboard on port 8082.
*   **Access**: `http://dash.YOUR_DOMAIN` (e.g., `http://dash.myhomelab.net`). **Requires Traefik to be running via `m3tal up`**. Ensure `DOMAIN` is correctly set in `/etc/m3tal/.env`.
*   **Best for**: Setups utilizing domain names, environments with multiple services behind a reverse proxy, and scenarios requiring unified access via Traefik.

#### Docker / Compose Runtime

M3TAL heavily relies on **Docker Engine** and **Docker Compose V2**. These are fundamental dependencies for the entire ecosystem.

*   The `m3tal up` command orchestrates all your services by executing `docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml ... up -d` across *all* `*-compose.yml` files found within the `/docker/` directory.
*   The `m3tal dash up` command is specialized:
    1.  It first fetches the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from our GitHub repository.
    2.  It then reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Finally, it starts the `m3tal-dashboard` container using the base compose file and the appropriate override.
*   To add your own custom Docker services, simply place your `my-stack-compose.yml` file into the `/docker/` directory. M3TAL will automatically pick it up with `m3tal up`.

#### Deployment Lifecycle — Day 2 Operations

Installing and managing new Docker-based stacks is streamlined:
1.  **Place your compose file**: Copy your `my-application-compose.yml` file into the `/docker/` directory (e.g., `sudo cp ./my-application-compose.yml /docker/`).
2.  **Configure environment variables**: Ensure any required environment variables for your new stack (and M3TAL itself) are properly set in `/etc/m3tal/.env`. Use `m3tal config wizard` for interactive setup, or `m3tal config set KEY value` for specific changes.
3.  **Start all stacks**: Run `m3tal up` to deploy your new stack alongside all other M3TAL services.

#### Traefik Routing Architecture

Traefik acts as the intelligent ingress for your M3TAL ecosystem. It is deployed as a container via `routing-compose.yml`.

*   **Entry Points**: Traefik binds port 80 (and optionally 443 for HTTPS) on the host machine, serving as the primary HTTP entry point (`web`).
*   **Service Discovery**: It automatically discovers services running in Docker containers by inspecting their Docker labels.
*   **Dynamic Configuration**: Traefik loads additional dynamic configuration from the `/docker/dynamic/` directory (which symlinks to `/opt/m3tal/stack/dynamic/`) using its file provider. Changes to these files are hot-reloaded.
*   **API Daemon Routing**: Traefik routes requests for `api.YOUR_DOMAIN` directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080`. This routing is defined in `/docker/dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.YOUR_DOMAIN` to the `m3tal-dashboard` container, leveraging the Traefik labels defined in `m3tal-compose.traefik.yml`.

#### Service Management (Systemd)

The core M3TAL API daemon is managed as a `systemd` service for robust background operation.

*   **Check Status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart Service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream Logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```

#### Port Map

| Port | Service                     | Access                                                                  |
| :--- | :-------------------------- | :---------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other Traefik services) |
| 8080 | M3TAL API daemon (Go)       | Host-local (accessed by dashboard/Traefik via `host.docker.internal`)   |
| 8081 | Traefik dashboard           | Host-local only (for Traefik's own UI/metrics)                          |
| 8082 | M3TAL Dashboard container   | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik         |

---

### 4. Direct Docker Commands (Fallback & Advanced Usage)

While the `m3tal` CLI provides a convenient abstraction, you may occasionally need to interact with Docker Compose directly for advanced debugging or specific operations. Remember that M3TAL uses Docker Compose V2.

To manage all M3TAL and user-defined stacks:

```bash
# Navigate to the M3TAL stack directory
cd /docker/

# List all compose files M3TAL would process
# You might need to manually specify them for direct Docker Compose
# Example: -f m3tal-compose.yml -f routing-compose.yml -f your-stack-compose.yml

# List all running containers (short form)
docker ps -a

# Start all stacks in detached mode
# Ensure you include all relevant compose files, and the correct override for the dashboard
docker compose \
  -f m3tal-compose.yml \
  -f m3tal-compose.local.yml \ # or m3tal-compose.traefik.yml based on your .env
  -f routing-compose.yml \
  -f your-custom-stack.yml \
  up -d

# Stop all stacks
docker compose \
  -f m3tal-compose.yml \
  -f routing-compose.yml \
  -f your-custom-stack.yml \
  down

# Stream logs from all services
docker compose \
  -f m3tal-compose.yml \
  -f routing-compose.yml \
  -f your-custom-stack.yml \
  logs -f

# To manage specific containers (e.g., m3tal-dashboard, traefik) individually:
# Check status of the dashboard container
docker inspect m3tal-dashboard --format "{{.State.Status}}"

# View dashboard logs
docker logs -f m3tal-dashboard

# Restart the dashboard container
docker restart m3tal-dashboard
```

By understanding these commands and the underlying architecture, you are now equipped to effectively manage your M3TAL ecosystem. Proceed with confidence.