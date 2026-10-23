# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the comprehensive command-line interface (CLI) cheat-sheet for the M3TAL system. This document serves as your definitive guide to managing your M3TAL environment, from initial setup to day-to-day operations.

The M3TAL CLI, provided by the `/usr/bin/m3tal` binary, is your unified entry point for interacting with the M3TAL API daemon, Docker services, and system configuration.

## 1. M3TAL System Architecture Overview

The M3TAL ecosystem is built upon several key components:

*   **CLI binary** (`/usr/bin/m3tal`): Your primary interface for all operations, installed via APT.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port 8080. It orchestrates Docker, manages state in `/var/lib/m3tal/state.db`, and exposes API routes for the dashboard and other tools.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running in Docker, communicating with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on port 80/443, using a file provider for dynamic routing and Docker labels for service discovery.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare Tunnel container providing secure, zero-config internet access to your services without opening firewall ports.

## 2. APT Installation

Before using the CLI, ensure M3TAL is installed:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Core M3TAL Commands

These commands provide fundamental interactions with the M3TAL system.

### `m3tal` or `m3tal ui`
Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a graphical interface is available.

**Usage Example:**
```bash
m3tal
```

### `m3tal headless [port]`
Starts M3TAL in headless mode, ideal for servers without a graphical interface. This runs the M3TAL API daemon directly. If no port is specified, it defaults to `8080`.

**Usage Examples:**
```bash
# Start M3TAL in headless mode on the default port (8080)
m3tal headless

# Start M3TAL in headless mode on a custom port (e.g., 9000)
m3tal headless 9000
```

### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. This provides a quick snapshot of your system's health and performance.

**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`
Performs a pre-flight health check of your M3TAL installation. This includes verifying Docker connectivity, validating the contents of `/etc/m3tal/.env`, and checking for port availability (e.g., 8080 for the API, 8082 for the dashboard).

**Usage Example:**
```bash
m3tal doctor
```

## 4. Configuration Management (`m3tal config`)

M3TAL's primary configuration is stored in `/etc/m3tal/.env`. The `m3tal config` commands help you manage these environment variables.

### `m3tal config wizard`
Launches an interactive wizard to guide you through configuring essential environment variables in `/etc/m3tal/.env`. This is highly recommended for initial setup.

**Usage Example:**
```bash
sudo m3tal config wizard
```

### `m3tal config set KEY VALUE`
Sets a single environment variable in `/etc/m3tal/.env` to the specified value. Requires `sudo` as it modifies a system-wide configuration file.

**Usage Examples:**
```bash
# Set the primary domain for your services
sudo m3tal config set DOMAIN myhomelab.com

# Configure the dashboard to be exposed via Traefik
sudo m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`
Retrieves and prints the current value of a specified environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
# Get the currently configured domain
m3tal config get DOMAIN
```

### `m3tal config scan`
Scans all M3TAL stacks and configuration files to list all environment variables, their current values (if set in `/etc/m3tal/.env`), and their default values. This is useful for auditing and understanding your full configuration landscape.

**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`
Prints the raw contents of the primary M3TAL configuration file, `/etc/m3tal/.env`, to the terminal.

**Usage Example:**
```bash
m3tal config list
```

## 5. Dashboard Management (`m3tal dash`)

The `m3tal dash` commands specifically manage the M3TAL Dashboard container, including its deployment, access, and credentials.

### `m3tal dashpass [username] [password]`
Updates or sets the password for a user accessing the M3TAL Dashboard. User credentials are stored in `/docker/users.json`. If `username` and `password` are omitted, the command will run interactively, prompting you for the necessary information. Requires `sudo`.

**Usage Examples:**
```bash
# Update password for 'admin' user interactively
sudo m3tal dashpass

# Set password for 'admin' user directly (non-interactive)
sudo m3tal dashpass admin MyS3cur3P@ssw0rd!
```

### `m3tal dash up`
Downloads the latest M3TAL Dashboard Docker Compose configuration files from GitHub, then starts the `m3tal-dashboard` container. It automatically selects the appropriate compose override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal dash up
```

### `m3tal dash down`
Stops and removes the `m3tal-dashboard` container, along with its associated network configuration.

**Usage Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`
Restarts the `m3tal-dashboard` container. This is useful after making configuration changes or to resolve minor issues.

**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`
Streams real-time logs from the `m3tal-dashboard` container to your terminal. This is invaluable for debugging issues with the dashboard.

**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`
Displays the current operational status of the `m3tal-dashboard` container (e.g., `running`, `exited`, `restarting`).

**Usage Example:**
```bash
m3tal dash status
```

---

### Dashboard Access Modes

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port (`8082`) to a host port (defaulting to `8082`).
*   **Access URL:** `http://HOST_IP:8082` or `http://localhost:8082`
*   **Best For:** LAN-only setups, first-time users, local testing, or environments where Traefik is not yet configured or desired. No additional services like Traefik are required for access.

    **Example:** After running `m3tal dash up` with `DASHBOARD_EXPOSE_MODE=local`, access your dashboard via `http://192.168.1.100:8082` (replace with your server's IP).

#### Mode 2: `traefik`
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which adds specific Traefik labels to the dashboard container. Traefik (running via `routing-compose.yml`) then detects these labels and automatically routes `dash.${DOMAIN}` to the dashboard's internal port `8082`.
*   **Prerequisite:** Traefik must be running and properly configured (e.g., via `m3tal up`).
*   **Access URL:** `http://dash.DOMAIN` (e.g., `http://dash.myhomelab.com`)
*   **Best For:** Domain-based setups, exposing services behind a central reverse proxy, and environments with multiple services managed by Traefik.

    **Example:** After setting `DASHBOARD_EXPOSE_MODE=traefik` and `DOMAIN=myhomelab.com` in `/etc/m3tal/.env`, then running `m3tal up` and `m3tal dash up`, access your dashboard via `http://dash.myhomelab.com`.

---

## 6. Stack Management

M3TAL uses Docker Compose to manage collections of containers, referred to as "stacks." These commands manage all Docker Compose files located in `/docker/`.

### `m3tal up`
Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/` (which is a symlink to `/opt/m3tal/stack/`). This command brings up all your configured M3TAL services and user-defined Docker stacks in detached mode.

**Usage Example:**
```bash
m3tal up
```

### `m3tal down`
Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command stops and removes all associated containers, networks, and volumes (unless explicitly defined as external).

**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`
Streams aggregated, real-time logs from *all* currently running Docker containers managed by M3TAL. This provides a consolidated view for comprehensive monitoring and debugging.

**Usage Example:**
```bash
m3tal logs
```

---

### Deployment Lifecycle — Day 2 Operations

To add a new Docker Compose stack to your M3TAL environment:

1.  **Place your compose file:** Create or copy your `docker-compose.yml` file (e.g., `my-stack-compose.yml`) into the `/docker/` directory. For example, to add an Ollama stack:
    ```bash
    # Example: Create an Ollama compose file
    sudo nano /docker/ollama-compose.yml
    ```
    (Paste your compose configuration here, e.g., the Ollama snippet provided in the ground truth)

2.  **Configure environment variables:** Ensure any required variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set` for this. For Ollama, you might need `PUID` and `PGID`.
    ```bash
    sudo m3tal config set PUID 1000
    sudo m3tal config set PGID 1000
    ```
3.  **Start all stacks:** Run `m3tal up` to deploy your new stack along with all other M3TAL services.
    ```bash
    m3tal up
    ```

---

## 7. Filesystem Contract

The M3TAL ecosystem relies on a specific filesystem layout for its operation:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database for the API daemon. Auto-created.                |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik config. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path.          |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |

## 8. Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage this service using standard `systemctl` commands.

*   **Check status:** View the current status of the M3TAL API daemon.
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart service:** Restart the M3TAL API daemon (useful after `.env` changes).
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream logs:** Follow real-time logs from the M3TAL API daemon.
    ```bash
    journalctl -u m3tal-api -f
    ```

## 9. Docker Compose Fallback / Direct Management

M3TAL heavily relies on **Docker Engine** and **Docker Compose V2**. While the `m3tal` CLI abstracts most Docker operations, you can always fall back to direct `docker compose` commands for advanced debugging or specific scenarios.

The M3TAL CLI's `m3tal up` and `m3tal down` commands essentially execute `docker compose` operations on all `*-compose.yml` files found in the `/docker/` directory.

**Example: Starting a specific stack directly (e.g., Traefik & Cloudflared):**
```bash
cd /docker/
sudo docker compose -f routing-compose.yml up -d
```

**Example: Stopping a specific stack directly:**
```bash
cd /docker/
sudo docker compose -f ollama-compose.yml down
```

**Example: Viewing logs for a specific service:**
```bash
sudo docker logs m3tal-dashboard -f
```

**Example: Listing all M3TAL-managed containers:**
```bash
sudo docker ps --filter "label=m3tal.stack"
```

## 10. Port Map

Understanding the port mapping is crucial for network configuration and troubleshooting.

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public (if Traefik is running and configured) |
| 8080 | M3TAL API daemon (Go)       | Host-local                                  |
| 8081 | Traefik dashboard           | Host-local only (for Traefik's own UI)      |
| 8082 | M3TAL Dashboard             | Direct port (if `local` mode) or via Traefik (if `traefik` mode) |

---
*DocSmith, M3TAL Ecosystem Documentation Architect*
*End of Document*