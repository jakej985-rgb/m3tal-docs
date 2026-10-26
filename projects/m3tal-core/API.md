# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, my purpose is to provide you with the most precise and actionable command reference for the M3TAL CLI. This document serves as your definitive cheat-sheet for managing your M3TAL instance, from initial configuration to advanced stack orchestration.

The M3TAL CLI (`/usr/bin/m3tal`) is your single point of interaction for the entire M3TAL ecosystem. It's a unified Go binary that orchestrates the underlying Docker Engine, Docker Compose V2, and communicates with the `m3tal-api` daemon.

---

## 1. Installation

The M3TAL CLI and its associated daemon are distributed via an APT repository for easy installation and updates.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 2. Core Concepts & Architecture

Understanding M3TAL's fundamental components and design principles is crucial for effective management.

### 2.1. M3TAL System Architecture Overview

The M3TAL ecosystem is composed of several tightly integrated components:

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary you interact with, acting as the primary entry point for all operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port `8080`. It handles Docker orchestration, manages the state database, and serves API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running in a Docker container (internally on port `8082`). It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services by domain name on port `80`. It uses a file provider for dynamic routing and automatically discovers services via Docker labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing zero-config internet access for your services.

### 2.2. Filesystem Contract

M3TAL relies on a strict filesystem layout for its configuration and data:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary configuration file.** Managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files and Traefik config. |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |
| `/etc/traefik/dynamic/`     | Traefik's dynamic configuration directory (e.g., `api.yml`).           |

### 2.3. Dashboard Access — Two Modes

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. This choice dictates how the dashboard container is exposed and whether Traefik is involved.

#### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard container's internal port `8082` to the host's `DASHBOARD_PORT` (default `8082`).
*   **Access**: Via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik required. Works immediately on a home server.
*   **Best for**: LAN-only setups, first-time users, local testing, or environments without a domain.

#### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik (if running via `routing-compose.yml`) then routes `dash.${DOMAIN}` to the dashboard container's internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (e.g., `http://dash.yourdomain.com`).
*   **Requirements**: Traefik must be running (`m3tal up` must have started `routing-compose.yml`). The `DOMAIN` environment variable must be correctly configured in `/etc/m3tal/.env`.
*   **Best for**: Domain-based setups, environments with multiple services behind a reverse proxy, and advanced network configurations.

### 2.4. Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its container orchestration backbone. These are hard dependencies for M3TAL's operation.

*   The `m3tal up` command executes `docker compose up -d` across *all* `*-compose.yml` files found in the `/docker/` directory.
*   The `m3tal dash up` command is specialized for the M3TAL dashboard. It intelligently:
    1.  Downloads the latest dashboard compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the appropriate compose override file based on the configured `DASHBOARD_EXPOSE_MODE`.
*   You can extend your M3TAL setup by placing any custom `*-compose.yml` files directly into `/docker/`.

### 2.5. Deployment Lifecycle — Day 2 Operations

Integrating new Docker Compose-based services into your M3TAL ecosystem is straightforward:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) into the `/docker/` directory.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific variables.
3.  Run `m3tal up` to start all services, including your new stack.

### 2.6. Traefik Routing Architecture

Traefik acts as the central reverse proxy for domain-based access. It is deployed as a container via `routing-compose.yml` within `/docker/`.

*   **Entry Point**: Traefik binds port `80` (and typically `443` for HTTPS, though not explicitly shown in basic config) on the host system as its HTTP entry point.
*   **Service Discovery**: It automatically discovers services by reading Docker labels on running containers (e.g., `m3tal-compose.traefik.yml` adds labels for the dashboard).
*   **Dynamic Configuration**: Traefik loads additional dynamic routing rules from `/etc/traefik/dynamic/` (which can include custom `.yml` files), allowing for hot-reloads without restarting Traefik.
*   **API Routing**: It routes `api.${DOMAIN}` to the M3TAL API daemon (`http://host.docker.internal:8080`) via a dynamic configuration file like `/etc/traefik/dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` to the dashboard container on port `8082` using Docker labels.

### 2.7. Port Map

| Port | Service                               | Access                                                     |
| :--- | :------------------------------------ | :--------------------------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (when Traefik is enabled)                           |
| 8080 | M3TAL API daemon (Go)                 | Host-local only; `http://host.docker.internal:8080` for containers |
| 8081 | Traefik dashboard                     | Host-local only (`127.0.0.1:8081`)                         |
| 8082 | M3TAL Dashboard (internal container) | Direct port (local mode) or via Traefik (traefik mode)     |

---

## 3. M3TAL CLI Commands

This section provides a complete reference for all M3TAL CLI commands, including real usage examples.

### `m3tal` or `m3tal ui`

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment to bring up the graphical user interface.

**Example Usage:**

```bash
m3tal
# OR
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode. This command runs the M3TAL API daemon directly in the foreground, ideal for server environments where the systemd service (`m3tal-api.service`) might be temporarily stopped, or for debugging. If no port is specified, it defaults to `8080`.

**Example Usage:**

```bash
# Start M3TAL API in headless mode on default port 8080
m3tal headless

# Start M3TAL API in headless mode on a custom port 9000
m3tal headless 9000
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. Provides a quick snapshot of your system's health and performance.

**Example Usage:**

```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. This command verifies critical aspects like Docker connectivity, `.env` file validity, and port availability to ensure a smooth operation.

**Example Usage:**

```bash
m3tal doctor
```

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring the primary `/etc/m3tal/.env` file. This is the recommended way to set up or modify core M3TAL environment variables.

**Example Usage:**

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`. This command allows for direct, non-interactive modification of configuration values.

**Example Usage:**

```bash
# Set the dashboard secret
m3tal config set DASHBOARD_SECRET "my_super_secret_key_123"

# Change the dashboard exposure mode
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Example Usage:**

```bash
# Get the currently configured domain
m3tal config get DOMAIN

# Retrieve the PUID
m3tal config get PUID
```

### `m3tal config scan`

Lists all known environment variables across all configured M3TAL stacks, showing their current values (if set) and default values. This provides a comprehensive overview of your entire M3TAL configuration.

**Example Usage:**

```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of the primary configuration file, `/etc/m3tal/.env`.

**Example Usage:**

```bash
m3tal config list
```

### `m3tal dashpass [username] [password]`

Updates a user's password for the M3TAL dashboard. If the username and password are omitted, the command will prompt for them interactively. This manages the `/docker/users.json` file.

**Example Usage:**

```bash
# Update the 'admin' user's password interactively
m3tal dashpass

# Set a specific password for the 'admin' user non-interactively
m3tal dashpass admin SecurePass@2024!
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub and then starts the `m3tal-dashboard` container. This command intelligently applies the correct `local` or `traefik` override based on your `DASHBOARD_EXPOSE_MODE`.

**Example Usage:**

```bash
m3tal dash up
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

**Example Usage:**

```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful for applying new configuration changes or resolving minor issues.

**Example Usage:**

```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the aggregated logs from the `m3tal-dashboard` container to your terminal. Useful for monitoring its operation and debugging.

**Example Usage:**

```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

**Example Usage:**

```bash
m3tal dash status
```

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts all defined Docker stacks, including M3TAL's core components and any custom user stacks.

**Example Usage:**

```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command stops and removes all containers, networks, and volumes defined by your M3TAL stacks.

**Example Usage:**

```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from *all* currently running M3TAL Docker containers. This provides a consolidated view of activity across your entire M3TAL ecosystem.

**Example Usage:**

```bash
m3tal logs
```

---

## 4. Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage this service directly using `systemctl` and view its logs with `journalctl`.

### Checking the API Daemon Status

Verify if the M3TAL API daemon is running and healthy.

**Example Usage:**

```bash
sudo systemctl status m3tal-api
```

### Restarting the API Daemon

Apply changes to the `/etc/m3tal/.env` that affect the API (e.g., `API_PORT`, `LOG_LEVEL`, `API_TOKEN`) by restarting the service.

**Example Usage:**

```bash
sudo systemctl restart m3tal-api
```

### Streaming API Daemon Logs

View real-time logs from the API daemon for debugging or monitoring.

**Example Usage:**

```bash
sudo journalctl -u m3tal-api -f
```

---

## 5. Docker Fallback Commands

While the `m3tal` CLI abstracts away much of the Docker complexity, it's essential to know the underlying `docker compose` commands as a fallback, especially for advanced debugging or manual intervention. M3TAL utilizes **Docker Engine** and **Docker Compose V2**.

All Docker Compose files managed by M3TAL (and your custom stacks) reside in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).

### Starting All Stacks Manually

To start all services defined by `.yml` files in `/docker/`:

```bash
cd /docker
sudo docker compose -f *-compose.yml up -d
```

### Stopping All Stacks Manually

To stop and remove all services defined by `.yml` files in `/docker/`:

```bash
cd /docker
sudo docker compose -f *-compose.yml down
```

### Checking Status of Specific Services

To check the status of a specific service or stack, for example, the `m3tal-dashboard`:

```bash
cd /docker
sudo docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml ps m3tal-dashboard
# (Adjust -f flags based on your DASHBOARD_EXPOSE_MODE)
```

### Viewing Logs for a Specific Service

To view logs for a specific container, e.g., `traefik`:

```bash
sudo docker logs -f traefik
```

### Rebuilding a Specific Service

To rebuild and restart a service (useful after manual changes to a compose file or a container image):

```bash
cd /docker
sudo docker compose -f routing-compose.yml up -d --build traefik
```

---

This command reference provides the essential tools and knowledge to effectively manage your M3TAL ecosystem. Should you encounter further complexities, remember the architectural ground truth and fallback commands provided herein.