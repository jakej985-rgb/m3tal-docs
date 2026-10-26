As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive CLI cheat-sheet for the M3TAL platform. This reference guide will equip you with all the essential commands to manage your M3TAL instance, from initial setup to day-to-day operations and advanced diagnostics.

---

# M3TAL CLI Command Reference

## Table of Contents

1.  [M3TAL Installation](#m3tal-installation)
2.  [M3TAL Core Commands](#m3tal-core-commands)
3.  [Configuration Management](#configuration-management)
4.  [Dashboard Management](#dashboard-management)
5.  [Stack Management](#stack-management)
6.  [System Diagnostics & Utilities](#system-diagnostics--utilities)
7.  [M3TAL System Architecture Overview](#m3tal-system-architecture-overview)
    *   [Components](#components)
    *   [Filesystem Contract](#filesystem-contract)
    *   [Dashboard Access Modes](#dashboard-access-modes)
    *   [Docker / Compose Runtime](#docker--compose-runtime)
    *   [Deployment Lifecycle – Day 2 Operations](#deployment-lifecycle--day-2-operations)
    *   [Traefik Routing Architecture](#traefik-routing-architecture)
    *   [Port Map](#port-map)
8.  [Systemd Service Management](#systemd-service-management)
9.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## 1. M3TAL Installation

The M3TAL CLI binary is installed via APT for seamless integration with your Linux system.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 2. M3TAL Core Commands

These commands initiate the core M3TAL experience or start the platform in a server-friendly mode.

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. On most systems, this will open your default web browser to the M3TAL Dashboard, providing a graphical user interface for managing your ecosystem.

**Usage Example:**

```bash
m3tal
```

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for home servers or environments without a desktop GUI. This command ensures the M3TAL API daemon and all configured Docker stacks are running in the background. If the M3TAL Dashboard is configured for `local` exposure (default), it will be accessible on `http://HOST_IP:8082`. The optional `[port]` argument is generally not required, as the M3TAL API runs on port `8080` internally and the dashboard on `8082`.

**Usage Example:**

```bash
m3tal headless
```

## 3. Configuration Management

M3TAL's configuration is primarily driven by `/etc/m3tal/.env`. These commands provide robust tools for managing your environment variables.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the essential variables within your `/etc/m3tal/.env` file. This is the recommended method for initial setup.

**Usage Example:**

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env` to the specified value. This command allows for precise control over individual settings without entering the full wizard.

**Usage Example:**

To configure the dashboard to be exposed via Traefik:

```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`

Retrieves and prints the current value of a specified environment variable from `/etc/m3tal/.env`.

**Usage Example:**

To check the currently configured domain:

```bash
m3tal config get DOMAIN
```

### `m3tal config scan`

Scans all active Docker Compose stacks within `/docker/` and lists all environment variables referenced, along with their default values if available. This provides a comprehensive overview of all configurable parameters across your M3TAL ecosystem.

**Usage Example:**

```bash
m3tal config scan
```

### `m3tal config list`

Displays the entire content of the current `/etc/m3tal/.env` file, showing all configured environment variables and their values.

**Usage Example:**

```bash
m3tal config list
```

## 4. Dashboard Management

These commands focus on controlling the `m3tal-dashboard` container, which provides the web-based UI. The dashboard's accessibility depends on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

### `m3tal dashpass [username] [password]`

Updates or sets the login credentials for a M3TAL Dashboard user. If `[username]` and `[password]` are omitted, an interactive prompt will guide you through the process. This modifies the `/docker/users.json` file.

**Usage Example (interactive):**

```bash
m3tal dashpass
```

**Usage Example (direct):**

```bash
m3tal dashpass admin StrongAndSecurePassword123!
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`), then starts the `m3tal-dashboard` container. It automatically applies the correct override (`.local.yml` or `.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

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

Restarts the `m3tal-dashboard` container. This is useful after making configuration changes that require the dashboard to reload.

**Usage Example:**

```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the `m3tal-dashboard` container to your terminal, providing real-time insight into its operations.

**Usage Example:**

```bash
m3tal dash logs
```

### `m3tal dash status`

Displays the current status of the `m3tal-dashboard` container (e.g., `running`, `exited`, `restarting`).

**Usage Example:**

```bash
m3tal dash status
```

## 5. Stack Management

These commands provide high-level control over all your Docker Compose stacks defined in the `/docker/` directory.

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command brings up all your configured M3TAL services and user-defined stacks in detached mode.

**Usage Example:**

```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory, gracefully stopping and removing all services and associated networks.

**Usage Example:**

```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running M3TAL Docker containers (including core services and user stacks). This provides a consolidated view of system activity.

**Usage Example:**

```bash
m3tal logs
```

## 6. System Diagnostics & Utilities

These tools help you monitor your system's health and troubleshoot issues within your M3TAL environment.

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. This provides a quick snapshot of your system's performance and resource utilization.

**Usage Example:**

```bash
m3tal stats
```

### `m3tal doctor`

Performs a comprehensive pre-flight health check of your M3TAL system. This includes verifying Docker connectivity, validating the `/etc/m3tal/.env` file, and checking for port availability, helping you identify and resolve common setup issues.

**Usage Example:**

```bash
m3tal doctor
```

## 7. M3TAL System Architecture Overview

Understanding the underlying architecture is crucial for effective management of your M3TAL ecosystem.

### Components

*   **CLI binary** (`/usr/bin/m3tal`): Your single entry point for all M3TAL operations, installed directly on your Linux host.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host port `8080`. It serves as the backbone, managing Docker interactions, maintaining the state database, and exposing the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services (like the API or Dashboard) by domain name on host port `80` (and `443` if configured for HTTPS). Traefik uses a file provider for dynamic routing configuration and discovers Docker services via labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container that establishes secure, outbound-only Cloudflare tunnels, providing zero-config internet access to your services without exposing ports directly.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract for predictable operations:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file, managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db`   | The SQLite state database, automatically created by the API daemon.  |
| `/opt/m3tal/stack/`         | The canonical directory for all Docker Compose files and Traefik config. |
| `/docker`                   | A symbolic link to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Stores dashboard user credentials, managed by `m3tal dashpass`.      |

### Dashboard Access Modes

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

**Mode 1: `local` (Default)**

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard container's port to your host.
*   **Access:** Via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik required. Works out of the box.
*   **Best for:** LAN-only setups, first-time users, or local development and testing.

**Mode 2: `traefik`**

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik (running via `routing-compose.yml`) then routes `dash.${DOMAIN}` to the dashboard on its internal port `8082`.
*   **Access:** Via `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with your configured `DOMAIN` variable).
*   **Requirements:** Traefik must be running via `m3tal up`.
*   **Best for:** Domain-based setups, exposing services behind a central reverse proxy, and advanced network configurations.

### Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as hard dependencies for container orchestration.

*   The `m3tal up` command orchestrates all Docker Compose files ending with `*-compose.yml` found within the `/docker/` directory.
*   The `m3tal dash up` command specifically manages the dashboard container:
    1.  It downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from GitHub.
    2.  It reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  It then starts the dashboard container using the appropriate compose override file.
*   User-defined stacks are placed directly in `/docker/`. Any valid `*-compose.yml` file in this directory will be managed by `m3tal up`/`down`.

### Deployment Lifecycle – Day 2 Operations

To integrate a new Docker stack into your M3TAL ecosystem:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  Ensure all required environment variables for your new stack are defined in `/etc/m3tal/.env`. You can use `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific variables.
3.  Run `m3tal up` to start your newly added stack along with all other M3TAL services.

### Traefik Routing Architecture

Traefik acts as the central ingress for your M3TAL services when domain-based access is desired.

*   Traefik is deployed as a container via `routing-compose.yml`.
*   It binds to port `80` (and `443` for HTTPS) on the host, serving as the primary HTTP entry point.
*   Services are automatically discovered via Docker labels (e.g., for the dashboard in `traefik` mode).
*   Dynamic routing configurations are loaded from files in `/docker/dynamic/` (e.g., `dynamic/api.yml`), enabling hot-reloading without restarting Traefik.
*   Example: `api.YOUR_DOMAIN` is routed to the M3TAL API daemon (running on `http://host.docker.internal:8080`) via `dynamic/api.yml`.
*   Example: `dash.YOUR_DOMAIN` is routed to the dashboard container (internal port `8082`) via Traefik labels defined in `m3tal-compose.traefik.yml` when `DASHBOARD_EXPOSE_MODE=traefik`.

### Port Map

| Port | Service                               | Access                                    |
| :--- | :------------------------------------ | :---------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (when Traefik is enabled)          |
| 8080 | M3TAL API daemon (Go)                 | Host-local only (internal communication)  |
| 8081 | Traefik Dashboard                     | Host-local only (e.g., `http://localhost:8081`) |
| 8082 | M3TAL Dashboard (container's internal port) | Direct port (local mode) or via Traefik (traefik mode) |

## 8. Systemd Service Management

The core M3TAL API daemon runs as a systemd service (`m3tal-api.service`). You can manage its lifecycle using standard `systemctl` commands.

**Check the status of the M3TAL API daemon:**

```bash
sudo systemctl status m3tal-api
```

**Restart the M3TAL API daemon:**

```bash
sudo systemctl restart m3tal-api
```

**Stream real-time logs from the M3TAL API daemon:**

```bash
sudo journalctl -u m3tal-api -f
```

## 9. Direct Docker Compose Fallback

While the `m3tal` CLI abstracts many Docker Compose operations, you can always fall back to direct `docker compose` commands if needed. All M3TAL Docker Compose files reside in the `/docker/` directory.

**Navigate to the stacks directory:**

```bash
cd /docker/
```

**List all services and their statuses:**

```bash
docker compose ps
```

**Start all services in detached mode:**

```bash
docker compose up -d
```

**Stop all services:**

```bash
docker compose down
```

**Stream logs from all services:**

```bash
docker compose logs -f
```

**Pull latest images for all services:**

```bash
docker compose pull
```

---

This comprehensive command reference provides you with the knowledge to navigate and master the M3TAL ecosystem. Should you encounter any issues or require further clarification, consult the official M3TAL documentation or community channels.

**DocSmith, M3TAL Ecosystem Documentation Architect**