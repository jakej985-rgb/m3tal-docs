# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, my mission is to illuminate every facet of your M3TAL experience. This document serves as the definitive command-line interface (CLI) cheat sheet, guiding you through the powerful `m3tal` binary and its interactions with the underlying M3TAL architecture.

## Installation

The `m3tal` CLI is installed as a Go binary via a dedicated APT repository, ensuring seamless updates and system integration.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## M3TAL Ecosystem Overview

The M3TAL ecosystem is designed for robust and flexible self-hosting. It comprises several key components working in concert:

*   **CLI Binary (`/usr/bin/m3tal`)**: Your primary interface, a unified Go binary for all operations.
*   **API Daemon (`m3tal-api.service`)**: A systemd-managed Go binary running on port 8080. It orchestrates Docker operations, manages the internal state database, and exposes API routes.
*   **Dashboard Container (`m3tal-dashboard`)**: A Python/Flask application running within Docker on port 8082, communicating with the local API daemon.
*   **Traefik Gateway (`routing-compose.yml`)**: An optional but recommended reverse proxy container that exposes services by domain name on port 80/443, using dynamic configuration.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container for secure, zero-config internet access without port forwarding.

M3TAL relies on **Docker Engine** and **Docker Compose V2** as hard dependencies for container orchestration.

## M3TAL Filesystem Contract

Understanding the filesystem layout is crucial for advanced configuration and troubleshooting:

| Path                        | Purpose                                                                                                                                                                                                               |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary Configuration File.** Contains all critical environment variables for M3TAL components. Managed by `m3tal config wizard`.                                                                                   |
| `/var/lib/m3tal/state.db`   | **SQLite State Database.** Auto-created by the `m3tal-api` daemon. Stores internal state, user data, and service configurations.                                                                                      |
| `/opt/m3tal/stack/`         | **Canonical Stack Directory.** The foundational location for all M3TAL-managed Docker Compose files (e.g., `m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration.                            |
| `/docker`                   | **User-Facing Stack Symlink.** This is a symlink to `/opt/m3tal/stack/`. Users should place their custom `*-compose.yml` files here for M3TAL to discover and manage.                                               |
| `/docker/users.json`        | **Dashboard Credential Store.** Stores encrypted credentials for the M3TAL Dashboard. Automatically created and managed by `m3tal dashpass`.                                                                         |
| `/etc/traefik/dynamic/`     | **Traefik Dynamic Configuration.** Traefik loads additional routing rules from `.yml` files in this directory. For example, `dynamic/api.yml` routes `api.DOMAIN` to the M3TAL API daemon.                            |
| `host.docker.internal:8080` | **API Daemon Access for Containers.** Inside Docker containers, this special hostname and port combination resolves to the M3TAL API daemon running directly on the host machine, making it accessible to services like the dashboard. |

## M3TAL CLI Commands

The `m3tal` binary provides a comprehensive set of commands for managing your M3TAL ecosystem.

### `m3tal` or `m3tal ui` — Launch Desktop Application

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a graphical interface is available.

**Usage Example:**

```bash
m3tal
# OR
m3tal ui
```

### `m3tal headless [port]` — Start M3TAL in Headless Mode

Starts the M3TAL API daemon in headless mode, ideal for server environments without a GUI. By default, it runs on port 8080.

**Usage Example (Default Port):**

```bash
m3tal headless
```

**Usage Example (Custom Port):**

```bash
m3tal headless 8085
```

### `m3tal stats` — Print System Metrics

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Useful for quick health checks.

**Usage Example:**

```bash
m3tal stats
```

**Example Output:**

```
CPU Usage: 15.2%
Memory Usage: 4.5 GB / 16 GB (28%)
GPU: NVIDIA GeForce RTX 3080
  Temperature: 55°C
  Usage: 32%
  Memory: 2.1 GB / 10 GB
Disk Usage (/): 150 GB / 500 GB (30%)
Network (eth0): Rx 1.2 MB/s, Tx 500 KB/s
```

### `m3tal doctor` — Pre-flight Health Check

Performs a comprehensive health check, verifying Docker connectivity, `.env` file validity, port availability, and other critical system parameters.

**Usage Example:**

```bash
m3tal doctor
```

**Example Output:**

```
[PASS] Docker daemon is running and accessible.
[PASS] /etc/m3tal/.env file exists and is readable.
[PASS] Required environment variables are set.
[PASS] Port 8080 (M3TAL API) is available.
[PASS] Port 8082 (M3TAL Dashboard) is available.
[PASS] Docker network 'proxy' exists.
[INFO] All checks passed. Your M3TAL system is ready.
```

### `m3tal config wizard` — Interactive Configuration Wizard

Launches an interactive wizard to guide you through configuring or updating variables in `/etc/m3tal/.env`. This is the recommended way to manage your primary M3TAL configuration.

**Usage Example:**

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE` — Set Environment Variable

Sets a single environment variable in `/etc/m3tal/.env`. This is useful for scripting or quick, direct changes.

**Usage Example (Set Dashboard Mode to Traefik):**

```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

**Usage Example (Set Domain Name):**

```bash
m3tal config set DOMAIN myhomelab.local
```

### `m3tal config get KEY` — Read Environment Variable

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**

```bash
m3tal config get DASHBOARD_EXPOSE_MODE
```

**Example Output:**

```
traefik
```

### `m3tal config scan` — List All Environment Variables

Lists all known environment variables across all M3TAL stacks, including their default values, to help you understand what's configurable.

**Usage Example:**

```bash
m3tal config scan
```

### `m3tal config list` — List Current .env File Contents

Prints the current contents of the `/etc/m3tal/.env` file.

**Usage Example:**

```bash
m3tal config list
```

### `m3tal dashpass [username] [password]` — Update Dashboard User Password

Updates a user's password for the M3TAL Dashboard, storing credentials securely in `/docker/users.json`. If no arguments are provided, it runs in interactive mode.

**Usage Example (Interactive):**

```bash
m3tal dashpass
```

**Usage Example (Directly Set for 'admin'):**

```bash
m3tal dashpass admin SuperSecureP@ssw0rd!
```

### `m3tal dash up` — Start M3TAL Dashboard

Pulls the latest dashboard Docker Compose configuration from GitHub, then starts the `m3tal-dashboard` container with the appropriate override based on `DASHBOARD_EXPOSE_MODE`.

**Usage Example:**

```bash
m3tal dash up
```

### `m3tal dash down` — Stop M3TAL Dashboard

Stops and removes the `m3tal-dashboard` container.

**Usage Example:**

```bash
m3tal dash down
```

### `m3tal dash restart` — Restart M3TAL Dashboard

Restarts the `m3tal-dashboard` container.

**Usage Example:**

```bash
m3tal dash restart
```

### `m3tal dash logs` — Stream Dashboard Logs

Streams real-time logs from the `m3tal-dashboard` container to your terminal.

**Usage Example:**

```bash
m3tal dash logs
```

### `m3tal dash status` — Show Dashboard Status

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, unhealthy).

**Usage Example:**

```bash
m3tal dash status
```

### `m3tal up` — Start All Stacks

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts all M3TAL core services (including dashboard and routing) and any custom user stacks.

**Usage Example:**

```bash
m3tal up
```

### `m3tal down` — Stop All Stacks

Runs `docker compose down` across all `*-compose.yml` files found in the `/docker/` directory, gracefully stopping and removing all associated containers, networks, and volumes.

**Usage Example:**

```bash
m3tal down
```

### `m3tal logs` — Stream Aggregated Logs

Streams aggregated logs from all running Docker containers managed by M3TAL. This is incredibly useful for observing the overall health and activity of your ecosystem.

**Usage Example:**

```bash
m3tal logs
```

## M3TAL Dashboard Access & Configuration

The M3TAL Dashboard provides a web-based interface for managing your system. Its accessibility is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### `DASHBOARD_EXPOSE_MODE=local` (Default Mode)

This mode directly exposes the dashboard container on a host port, typically `8082`. It's ideal for initial setup, local network access, or environments where a reverse proxy like Traefik isn't desired.

*   **Configuration**:
    ```bash
    m3tal config set DASHBOARD_EXPOSE_MODE local
    ```
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file to add a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access URL**:
    *   `http://YOUR_HOST_IP:8082`
    *   `http://localhost:8082` (if accessed from the host machine itself)
*   **Requirements**: No Traefik required.
*   **Best for**: LAN-only setups, first-time users, local testing.

### `DASHBOARD_EXPOSE_MODE=traefik` (Domain-based Access)

This mode configures the dashboard to be routed via Traefik, allowing access through a subdomain (e.g., `dash.yourdomain.com`). This is the preferred method for domain-based setups and when running multiple services behind a reverse proxy.

*   **Configuration**:
    ```bash
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    m3tal config set DOMAIN yourdomain.com # Ensure your domain is set
    ```
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file to add Traefik labels. Traefik (if running) will then route `dash.${DOMAIN}` to the dashboard container on its internal port 8082.
*   **Access URL**: `http://dash.yourdomain.com`
*   **Requirements**:
    *   Traefik must be running (started via `m3tal up` as part of `routing-compose.yml`).
    *   Your `DOMAIN` environment variable in `/etc/m3tal/.env` must be correctly set.
    *   DNS records must point `dash.yourdomain.com` to your M3TAL host's IP address.
*   **Best for**: Domain-based setups, multiple services behind a reverse proxy, robust internet access.

Remember to run `m3tal dash up` after changing `DASHBOARD_EXPOSE_MODE` to apply the new configuration. If you change the `DOMAIN` variable, a full `m3tal down` then `m3tal up` might be required to ensure Traefik reloads its configuration correctly, though in most cases Traefik's dynamic file provider handles updates automatically.

## Managing M3TAL API Daemon (systemd)

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service, providing the backbone for the CLI and dashboard's operations.

*   **Check API Daemon Status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart API Daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream API Daemon Logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```

## Docker & Docker Compose (Fallback & Advanced Usage)

M3TAL uses **Docker Engine** and **Docker Compose V2** to manage all containerized services. The `m3tal` CLI abstracts many `docker compose` commands for convenience, but you can always interact directly with Docker Compose for advanced scenarios or troubleshooting.

All M3TAL's primary Docker Compose files reside in `/opt/m3tal/stack/`, which is symlinked to the user-facing `/docker/` directory.

### Core Compose Files

*   `/docker/m3tal-compose.yml`: Base configuration for the M3TAL dashboard.
*   `/docker/m3tal-compose.local.yml`: Override for local dashboard exposure.
*   `/docker/m3tal-compose.traefik.yml`: Override for Traefik-routed dashboard exposure.
*   `/docker/routing-compose.yml`: Manages Traefik and Cloudflared containers.

### How `m3tal up` Works

When you run `m3tal up`, the CLI essentially executes `docker compose` against all `*-compose.yml` files found in `/docker/`. It intelligently includes the correct dashboard override based on `DASHBOARD_EXPOSE_MODE`.

**Equivalent Direct `docker compose` Command (for reference):**

If `DASHBOARD_EXPOSE_MODE` is `local` and `DOMAIN` is `myhome.com`:
```bash
# This is an illustrative example of what m3tal up might run, it's not a direct command replacement for all cases.
# m3tal up handles the selection of overrides and all compose files.
docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml -f /docker/routing-compose.yml up -d
```
If `DASHBOARD_EXPOSE_MODE` is `traefik`:
```bash
# Again, illustrative. m3tal up is the canonical command.
docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml -f /docker/routing-compose.yml up -d
```

### Direct Docker Compose Fallback Commands

You can interact with individual stacks directly using `docker compose` if needed. Remember to include the `-f` flag for each compose file relevant to the stack you're managing.

*   **To start only the Dashboard (using `local` mode):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
    ```
*   **To stop only the Dashboard:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down
    ```
*   **To view logs for the `m3tal-dashboard` container:**
    ```bash
    docker logs -f m3tal-dashboard
    ```
*   **To restart the `traefik` container:**
    ```bash
    docker restart traefik
    ```
*   **To check the status of all M3TAL-related containers:**
    ```bash
    docker ps -a --filter label="m3tal.stack"
    ```

### Adding New User Stacks

To deploy your own Docker Compose applications with M3TAL:

1.  **Place your compose file:** Create a new `*-compose.yml` file (e.g., `/docker/my-app-compose.yml`) in the `/docker/` directory.
2.  **Configure environment variables:** If your stack requires custom environment variables, define them in `/etc/m3tal/.env` using `m3tal config set KEY VALUE` or `m3tal config wizard`.
3.  **Run `m3tal up`:** This will discover your new compose file and start your application alongside M3TAL's core services.

**Example: `/docker/my-app-compose.yml`**

```yaml
services:
  whoami:
    image: traefik/whoami
    container_name: my-whoami-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.${DOMAIN:-localhost}`)"
      - "traefik.http.routers.whoami.entrypoints=web"
      - "traefik.http.services.whoami.loadbalancer.server.port=80"
      - "traefik.docker.network=proxy"
    networks:
      - proxy

networks:
  proxy:
    external: true
```
After saving this file, run `m3tal up`. If `DOMAIN` is set to `myhome.com`, you could then access this service at `http://whoami.myhome.com` (assuming Traefik is running and configured).

## Port Map

| Port  | Service                     | Access (from outside host)                                                                                                                                                                                                                                                                                                                                                            |
| :---- | :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 80    | Traefik HTTP entry point    | **Public (traefik mode):** If Traefik is running and `DASHBOARD_EXPOSE_MODE=traefik`, or for any services routed by Traefik. Requires `routing-compose.yml` to be active.                                                                                                                                                                                                             |
| 8080  | M3TAL API daemon (Go)       | **Host-local:** Accessible from the host machine (e.g., by the `m3tal` CLI, the M3TAL Dashboard container via `host.docker.internal:8080`, or other local services). Not typically exposed directly to the public internet, though Traefik routes `api.${DOMAIN}` to it for external access.                                                                                             |
| 8081  | Traefik dashboard           | **Host-local only:** The Traefik management dashboard is intentionally bound to `127.0.0.1:8081` to prevent public exposure. Access via `http://localhost:8081` from the host.                                                                                                                                                                                                           |
| 8082  | M3TAL Dashboard             | **Direct port (local mode):** Accessible via `http://YOUR_HOST_IP:8082` when `DASHBOARD_EXPOSE_MODE=local`. <br>**Via Traefik (traefik mode):** Not directly exposed to the host network; accessible via `http://dash.${DOMAIN}` when `DASHBOARD_EXPOSE_MODE=traefik` and Traefik is routing traffic. The port mapping is internal to Traefik. |

---

This documentation should provide you with a comprehensive understanding of the M3TAL CLI and its underlying architecture. Should you encounter any challenges, remember to leverage `m3tal doctor` and review your `.env` configuration.

DocSmith, M3TAL Ecosystem Documentation Architect.