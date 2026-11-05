# M3TAL CLI Command Reference

Welcome, Architect. This document serves as the definitive command-line interface (CLI) cheat-sheet for the M3TAL Ecosystem, detailing every available command, its purpose, and practical usage examples. M3TAL is designed for seamless management of your home server infrastructure, leveraging Docker and systemd for robust orchestration.

## Table of Contents

1.  [Installation](#1-installation)
2.  [Core CLI Commands](#2-core-cli-commands)
3.  [Configuration Management](#3-configuration-management)
4.  [Dashboard Management](#4-dashboard-management)
5.  [Stack Management](#5-stack-management)
6.  [M3TAL System Architecture](#6-m3tal-system-architecture)
    *   [Filesystem Contract](#filesystem-contract)
    *   [M3TAL API Daemon](#m3tal-api-daemon)
    *   [Docker / Compose Runtime](#docker--compose-runtime)
    *   [Dashboard Access Modes](#dashboard-access-modes)
    *   [Traefik Routing Architecture](#traefik-routing-architecture)
    *   [Port Map](#port-map)
7.  [Day 2 Operations: Deploying New Stacks](#7-day-2-operations-deploying-new-stacks)
8.  [Service Management with Systemd](#8-service-management-with-systemd)
9.  [Docker Fallback Commands](#9-docker-fallback-commands)

---

## 1. Installation

The M3TAL CLI binary (`/usr/bin/m3tal`) is distributed via an APT repository for easy installation and updates on Debian/Ubuntu-based systems.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 2. Core CLI Commands

These commands provide fundamental interactions with the M3TAL ecosystem.

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. This command is typically used when running M3TAL on a desktop environment with a graphical user interface.

**Usage Example:**
```bash
m3tal
# Or explicitly:
m3tal ui
```

### `m3tal headless [port]`

Starts the M3TAL API daemon in headless mode, ideal for servers without a GUI. The API daemon runs as a systemd service (`m3tal-api.service`) by default on port 8080. This command allows for foreground execution with an optional custom port.

**Usage Example:**
```bash
# Start in headless mode on default port 8080 (if not already running as systemd service)
m3tal headless

# Start in headless mode on a custom port 8000
m3tal headless 8000
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Provides a quick overview of your M3TAL server's health and performance.

**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL setup. This command verifies critical components such as Docker connectivity, the validity of your `/etc/m3tal/.env` configuration, and the availability of essential ports.

**Usage Example:**
```bash
m3tal doctor
```

---

## 3. Configuration Management

M3TAL uses `/etc/m3tal/.env` as its primary configuration file. These commands help you manage environment variables crucial for M3TAL's operation.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended method for initial setup and major configuration changes.

**Usage Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env`. This allows for precise, non-interactive configuration updates.

**Usage Example:**
```bash
# Set the domain for Traefik routing
m3tal config set DOMAIN "myhomeserver.com"

# Change the dashboard exposure mode to use Traefik
m3tal config set DASHBOARD_EXPOSE_MODE "traefik"
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
# Get the currently configured dashboard exposure mode
m3tal config get DASHBOARD_EXPOSE_MODE
```

### `m3tal config scan`

Lists all environment variables known across all M3TAL stacks and services, including defaults and overrides. This provides a comprehensive view of potential configuration points.

**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`

Displays the current contents of your primary configuration file, `/etc/m3tal/.env`, to the terminal.

**Usage Example:**
```bash
m3tal config list
```

---

## 4. Dashboard Management

The M3TAL Dashboard (`m3tal-dashboard` container) provides a web-based interface for managing your ecosystem. These commands control its lifecycle and access.

### `m3tal dashpass [username] [password]`

Updates a user's password for the M3TAL Dashboard. If `username` and `password` arguments are omitted, the command will launch an interactive prompt to securely set credentials. This command manages entries in `/docker/users.json`.

**Usage Example:**
```bash
# Interactively set or update the password for a user
m3tal dashpass

# Set the password for user 'admin' to 'SuperSecurePwd123!'
m3tal dashpass admin SuperSecurePwd123!
```

### `m3tal dash up`

Pulls the latest dashboard `docker-compose` configuration from GitHub and then starts the `m3tal-dashboard` container with the appropriate exposure mode (`local` or `traefik`) based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

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

Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.

**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams aggregated logs from the `m3tal-dashboard` container to your terminal, aiding in real-time monitoring and debugging.

**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container, including its state (running, stopped, etc.), uptime, and port mappings.

**Usage Example:**
```bash
m3tal dash status
```

---

## 5. Stack Management

These commands are used for managing all your Docker Compose stacks defined in `/docker/`.

### `m3tal up`

Executes `docker compose up -d` across all `*-compose.yml` files found within the `/docker/` directory. This command starts or recreates all services defined in your active stacks in detached mode.

**Usage Example:**
```bash
m3tal up
```

### `m3tal down`

Executes `docker compose down` across all `*-compose.yml` files found within the `/docker/` directory. This command stops and removes all containers, networks, and volumes associated with your M3TAL stacks.

**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running containers managed by M3TAL's Docker Compose stacks. This provides a consolidated view of all service activity.

**Usage Example:**
```bash
m3tal logs
```

---

## 6. M3TAL System Architecture

Understanding the underlying architecture is key to effectively managing your M3TAL ecosystem.

### Filesystem Contract

M3TAL establishes a strict filesystem contract for its operations:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.   |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                   | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.             |
| `/docker/dynamic/`          | Directory for Traefik's dynamic file provider configurations.        |

### M3TAL API Daemon

The M3TAL API daemon is a Go binary running as a systemd service (`m3tal-api.service`) typically on port `8080`. It serves as the central brain, managing Docker operations, maintaining the state database (`/var/lib/m3tal/state.db`), and exposing API routes for the CLI and Dashboard.

### Docker / Compose Runtime

M3TAL is built upon **Docker Engine** and **Docker Compose V2**. These are hard dependencies for the system to function.

*   The `m3tal up` command orchestrates all Docker Compose stacks by running `docker compose` commands across all `*-compose.yml` files present in the `/docker/` directory.
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container. It performs the following steps:
    1.  Downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from GitHub.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the dashboard container, applying the appropriate compose override file (`m3tal-compose.local.yml` for `local` mode or `m3tal-compose.traefik.yml` for `traefik` mode).
*   User-defined Docker Compose stacks should be placed in the `/docker/` directory. Any `*-compose.yml` file in this directory will be included in `m3tal up` and `m3tal down` operations.

### Dashboard Access Modes

The M3TAL Dashboard can be accessed in two primary modes, configured via the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the `m3tal-compose.local.yml` override, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Benefits:** No Traefik required. Works out-of-the-box on a home server. Ideal for LAN-only setups, first-time users, and local testing.

#### Mode 2: `traefik`

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the `m3tal-compose.traefik.yml` override. This file adds specific Traefik labels to the dashboard container, instructing Traefik to route `dash.${DOMAIN}` to the dashboard on its internal port 8082.
*   **Access via:** `http://dash.DOMAIN` (e.g., `http://dash.myhomeserver.com`).
*   **Prerequisite:** Traefik must be running (started via `m3tal up`).
*   **Benefits:** Best for domain-based setups, integrating with multiple services behind a central reverse proxy, and enabling Cloudflare tunnels for external access.

### Traefik Routing Architecture

Traefik acts as the central reverse proxy for M3TAL, deployed as a container via `routing-compose.yml`.

*   **Entry Point:** Binds to host port `80` (HTTP) as its primary entry point.
*   **Service Discovery:** Automatically discovers services via Docker labels on containers.
*   **Dynamic Configuration:** Loads additional dynamic routing rules from files in `/docker/dynamic/` (using the file provider for hot-reloading).
*   **API Daemon Routing:** Routes `api.DOMAIN` (if configured) to the M3TAL API daemon (running on `http://host.docker.internal:8080`) via a configuration file like `/docker/dynamic/api.yml`.
*   **Dashboard Routing:** Routes `dash.DOMAIN` to the `m3tal-dashboard` container when `DASHBOARD_EXPOSE_MODE=traefik` is set, leveraging Docker labels in `m3tal-compose.traefik.yml`.
*   **Cloudflared:** `routing-compose.yml` also includes an optional Cloudflare tunnel container (`cloudflared`) for zero-config internet access to your services.

### Port Map

Key ports utilized by the M3TAL ecosystem:

| Port | Service                          | Access                                                    |
| :--- | :------------------------------- | :-------------------------------------------------------- |
| 80   | Traefik HTTP entry point         | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)             |
| 8080 | M3TAL API daemon (Go binary)     | Host-local only (internal communication)                  |
| 8081 | Traefik dashboard (admin UI)     | Host-local only (for Traefik management)                  |
| 8082 | M3TAL Dashboard (Python/Flask) | Direct port binding (local mode) or via Traefik (traefik mode) |

---

## 7. Day 2 Operations: Deploying New Stacks

Adding new services or applications to your M3TAL-managed server is straightforward:

1.  **Place your compose file:** Create or copy your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  **Configure environment variables:** Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive experience, or `m3tal config set KEY value` for direct assignment.
3.  **Start all stacks:** Run `m3tal up` to initialize and start your new stack along with all other managed services.

---

## 8. Service Management with Systemd

The M3TAL API daemon runs as a systemd service, `m3tal-api.service`. You can manage its lifecycle using standard `systemctl` commands.

*   **Check Status:** View the current status of the M3TAL API daemon.
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart Service:** Restart the M3TAL API daemon (e.g., after updating configuration).
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stop Service:** Stop the M3TAL API daemon.
    ```bash
    sudo systemctl stop m3tal-api
    ```
*   **Start Service:** Start the M3TAL API daemon.
    ```bash
    sudo systemctl start m3tal-api
    ```
*   **Follow Logs:** Stream real-time logs from the M3TAL API daemon.
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## 9. Docker Fallback Commands

While `m3tal` provides a convenient wrapper, you can always interact directly with Docker Compose V2 if needed, especially for debugging or advanced scenarios. All M3TAL managed `docker-compose` files reside in `/docker/`.

*   **List all active services/containers managed by M3TAL:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-custom-stack-compose.yml ps
    # Note: You'll need to explicitly list all relevant compose files.
    ```
*   **Start all M3TAL-managed services in detached mode:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-custom-stack-compose.yml up -d
    ```
*   **Stop all M3TAL-managed services:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-custom-stack-compose.yml down
    ```
*   **Stream logs from a specific M3TAL-managed service (e.g., the dashboard):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml logs -f m3tal-dashboard
    ```
*   **Inspect a specific M3TAL-managed container:**
    ```bash
    docker inspect m3tal-dashboard-1
    ```

---