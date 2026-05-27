# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive command-line interface (CLI) cheat-sheet for managing your M3TAL server. This document provides a comprehensive overview of every `m3tal` command, complete with real-world usage examples, and details the underlying architecture to empower your Day 2 operations.

## Introduction to M3TAL

M3TAL is a unified server management platform built around Docker and Docker Compose. It orchestrates services, manages configuration, and provides a sleek web dashboard, all powered by a Go API daemon. The `m3tal` CLI binary acts as your primary interface for interacting with the entire ecosystem.

## Installation

To install the `m3tal` CLI and its associated systemd service, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## M3TAL Filesystem Contract

The M3TAL ecosystem adheres to a strict filesystem contract to ensure consistent operation and simplify management.

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. All environment variables for Docker Compose and the API are sourced from here. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Stores internal M3TAL API data, such as discovered services. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains core M3TAL Compose files (`m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration. |
| `/docker`                | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path where all Docker Compose stack files (`*-compose.yml`) should reside. |
| `/docker/users.json`     | Dashboard credential store. Contains usernames and hashed passwords for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Dashboard Access Modes

The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. It's crucial to understand these modes for proper dashboard accessibility.

### Mode 1: `local` (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds port `${DASHBOARD_PORT:-8082}` from the host to the dashboard container. No Traefik required.
*   **Access:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Use Case:** Ideal for LAN-only setups, initial configuration, and local development. It works out-of-the-box without needing a domain or Traefik.

### Mode 2: `traefik`

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik (running via `routing-compose.yml`) then routes `dash.${DOMAIN}` to the dashboard container's internal port 8082. Requires the Traefik stack to be running (`m3tal up`).
*   **Access:** `http://dash.YOUR_DOMAIN` (e.g., `http://dash.example.com`).
*   **Use Case:** Best for domain-based access, production environments, and when integrating with other services behind a reverse proxy.

---

## M3TAL CLI Command Reference

### Core M3TAL Commands

#### `sudo m3tal`

Opens the interactive Text User Interface (TUI) Control Center. This provides a menu-driven interface for common operations. Requires `sudo` as it performs privileged operations (e.g., Docker commands).

**Usage Example:**

```bash
sudo m3tal
```

#### `m3tal init`

Generates the primary configuration file, `/etc/m3tal/.env`, from system defaults. This command is essential on first installation or if the `.env` file is missing.

**Usage Example:**

```bash
m3tal init
```

#### `m3tal doctor`

Performs a pre-flight health check of the M3TAL system. It verifies Docker connectivity, checks the validity of `/etc/m3tal/.env`, and ensures required ports (like 8080 for the API, 8082 for the dashboard) are available.

**Usage Example:**

```bash
m3tal doctor
```

### Configuration Management (`m3tal config`)

These commands manage the `/etc/m3tal/.env` configuration file.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. It prompts for critical environment variables.

**Usage Example:**

```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`. If the key exists, its value is updated; otherwise, a new key-value pair is added.

**Usage Examples:**

```bash
m3tal config set DOMAIN myhome.lan
m3tal config set DASHBOARD_EXPOSE_MODE traefik
m3tal config set PUID 1001
```

#### `m3tal config get KEY`

Retrieves and displays the value of a specific environment variable from `/etc/m3tal/.env`.

**Usage Examples:**

```bash
m3tal config get DOMAIN
m3tal config get DASHBOARD_PORT
```

#### `m3tal config scan`

Lists all environment variables known to the M3TAL system across all configured stacks. This includes default variables and those defined in `/etc/m3tal/.env`.

**Usage Example:**

```bash
m3tal config scan
```

#### `m3tal config list`

Displays the current contents of the `/etc/m3tal/.env` file.

**Usage Example:**

```bash
m3tal config list
```

### Dashboard Management (`m3tal dash`)

These commands specifically manage the M3TAL Dashboard container and its access.

#### `m3tal dashpass [username] [password]`

Updates the password for a specified dashboard user in `/docker/users.json`. If `username` and `password` are omitted, the command becomes interactive, prompting for the user and new password.

**Usage Examples:**

```bash
# Interactive mode (prompts for username and password)
m3tal dashpass

# Set password for 'admin' user directly
m3tal dashpass admin MyNewSecureP@ssword!
```

#### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and then starts the dashboard container using the appropriate override.

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

Streams aggregated logs from the `m3tal-dashboard` container. Press `Ctrl+C` to stop streaming.

**Usage Example:**

```bash
m3tal dash logs
```

#### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., `running`, `exited`).

**Usage Example:**

```bash
m3tal dash status
```

### Stack Management

These commands manage all Docker Compose stacks defined by `*-compose.yml` files in the `/docker/` directory.

#### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all defined services in detached mode.

**Usage Example:**

```bash
m3tal up
```

#### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command stops and removes all containers, networks, and volumes defined by these stacks.

**Usage Example:**

```bash
m3tal down
```

#### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. Press `Ctrl+C` to stop streaming.

**Usage Example:**

```bash
m3tal logs
```

## Systemd Service Management

The M3TAL API daemon runs as a systemd service (`m3tal-api.service`). You can manage this service using standard `systemctl` commands.

#### Check API daemon status

```bash
systemctl status m3tal-api
```

#### Stream API daemon logs

```bash
journalctl -u m3tal-api -f
```

#### Restart the API daemon

```bash
sudo systemctl restart m3tal-api
```

## Direct Docker Commands (Fallback)

M3TAL uses **Docker Engine** and **Docker Compose V2** under the hood. While the `m3tal` CLI abstracts most Docker operations, you can always use direct `docker` and `docker compose` commands as a fallback or for advanced debugging. The M3TAL CLI passes environment variables from `/etc/m3tal/.env` to `docker compose`.

**Filesystem Context:**
Remember that all M3TAL-managed Docker Compose files are located in `/docker/` (which is a symlink to `/opt/m3tal/stack/`). When running direct `docker compose` commands, navigate to this directory or specify the compose file paths.

#### Start all M3TAL stacks directly

```bash
cd /docker/
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f your-other-stack-compose.yml up -d
```
*Note: You would list all `*-compose.yml` files present in `/docker/`.*

#### Stop a specific stack directly (e.g., routing stack)

```bash
cd /docker/
sudo docker compose -f routing-compose.yml down
```

#### View logs for the `m3tal-dashboard` container directly

```bash
sudo docker logs -f m3tal-dashboard
```

#### Inspect the `m3tal-dashboard` container

```bash
sudo docker inspect m3tal-dashboard
```

#### Pull latest images for all M3TAL stacks directly

```bash
cd /docker/
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml pull
```

## M3TAL Runtime Architecture

### Docker / Compose Runtime

*   M3TAL is entirely dependent on **Docker Engine** and **Docker Compose V2**. Ensure these are installed and running on your system.
*   The `m3tal up` command iterates through all `*.yml` files in the `/docker/` directory and executes `docker compose up -d` for them.
*   The `m3tal dash up` command is special: it first downloads the latest dashboard compose files and then starts the dashboard container with the appropriate `DASHBOARD_EXPOSE_MODE` override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).
*   To add a new service/stack, simply place its Docker Compose file (e.g., `my-app-compose.yml`) in `/docker/`, ensure required environment variables are set in `/etc/m3tal/.env`, and run `m3tal up`.

### Traefik Routing Architecture

Traefik acts as the central reverse proxy for M3TAL, deployed via `routing-compose.yml`.

*   **Entry Point:** Traefik listens on port 80 (and 443 if configured) on the host.
*   **Service Discovery:** It automatically discovers services by reading Docker labels (e.g., on the `m3tal-dashboard` container when `DASHBOARD_EXPOSE_MODE=traefik`).
*   **File Provider:** It also loads dynamic routing configurations from `/docker/dynamic/` (e.g., `dynamic/api.yml` for the M3TAL API daemon).
*   **API Routing:** `api.${DOMAIN}` is routed to the M3TAL API daemon at `http://host.docker.internal:8080` via a static file provider configuration in `/docker/dynamic/api.yml`.
*   **Dashboard Routing:** `dash.${DOMAIN}` is routed to the M3TAL Dashboard container via Docker labels attached to the dashboard container (only when `DASHBOARD_EXPOSE_MODE=traefik`).

### Port Map

| Port | Service                                | Access                 |
| :--- | :------------------------------------- | :--------------------- |
| 80   | Traefik HTTP entry point               | Public (traefik mode)  |
| 443  | Traefik HTTPS entry point              | Public (traefik mode)  |
| 8080 | M3TAL API daemon (Go)                  | Host-local             |
| 8081 | Traefik dashboard (admin interface)    | Host-local only        |
| 8082 | M3TAL Dashboard (Python/Flask)         | Direct port (local mode) or via Traefik (traefik mode) |