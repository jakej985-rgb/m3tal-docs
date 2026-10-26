# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive command-line interface (CLI) cheat-sheet for managing your M3TAL self-hosting environment. The `m3tal` CLI, a single Go binary, is your primary interface for interacting with the M3TAL API daemon, configuring your system, and orchestrating your Docker-based services.

---

## 1. M3TAL Installation

To get started, ensure you have the `m3tal` CLI installed. Follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 2. Core M3TAL Concepts & Architecture

M3TAL is built on a robust architecture designed for flexibility and ease of use. Understanding these core components will empower you to effectively manage your self-hosted services.

### 2.1 M3TAL Ecosystem Overview

*   **CLI Binary (`/usr/bin/m3tal`)**: Your unified Go binary entry point for all operations. This document details its commands.
*   **API Daemon (`m3tal-api.service`)**: A Go binary running as a `systemd` service on port 8080. It's the brain, managing Docker interactions, the state database, and exposing API routes.
*   **M3TAL Dashboard (`m3tal-dashboard`)**: A Python/Flask Docker container (internal port 8082) providing a web-based user interface. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`)**: An optional Docker container acting as a reverse proxy, exposing services by domain name on port 80/443. It uses a file provider for dynamic configuration and Docker labels for service discovery.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container for secure, zero-config internet access to your services, bypassing direct port forwarding.

### 2.2 Filesystem Contract

M3TAL relies on a specific filesystem layout for configuration and data. It's crucial to understand these paths:

| Path                        | Purpose                                                                             |
| :-------------------------- | :---------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary configuration file**. Contains all environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Stores M3TAL's internal state. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains `*-compose.yml` files and Traefik dynamic config. |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | M3TAL Dashboard credential store. Managed by `m3tal dashpass`.                      |
| `/docker/dynamic/`          | Traefik dynamic configuration files (e.g., `api.yml`).                              |

### 2.3 M3TAL Dashboard Access Modes (Critical)

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: Local (Default)

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file, which adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container.
*   **Access**: Navigate to `http://YOUR_HOST_IP:8082` or `http://localhost:8082` in your web browser.
*   **Benefits**: No Traefik required. Works immediately on a home server. Ideal for LAN-only setups, first-time users, and local testing.

#### Mode 2: Traefik

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the dashboard container. Traefik (if running via `m3tal up`) then routes `dash.${DOMAIN}` to the dashboard on its internal port 8082.
*   **Access**: Navigate to `http://dash.YOUR_DOMAIN` in your web browser (e.g., `http://dash.myhomelab.com`).
*   **Requirements**: Traefik must be running as part of your `m3tal up` stack. Ensure `DOMAIN` is correctly set in `/etc/m3tal/.env`.
*   **Benefits**: Domain-based access, integrates seamlessly with other services behind Traefik, suitable for advanced networking and public exposure.

**To switch modes**:
1.  Edit `/etc/m3tal/.env` and change the `DASHBOARD_EXPOSE_MODE` value.
2.  Run `m3tal config wizard` to ensure the change is registered and all related variables are in place.
3.  Run `m3tal dash restart` to apply the new configuration.

### 2.4 Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as hard dependencies.

*   The `m3tal up` command orchestrates all `*-compose.yml` files found in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`).
*   The `m3tal dash up` command specifically manages the dashboard container:
    1.  It downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from GitHub.
    2.  It then starts the dashboard container using the appropriate override file based on `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
*   You can place your own custom `*-compose.yml` files directly into `/docker/` to integrate them into the M3TAL ecosystem.

---

## 3. M3TAL CLI Command Reference

This section provides a complete cheat-sheet for every `m3tal` command, along with real usage examples.

### 3.1. General Commands

#### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application. This command starts a graphical user interface for M3TAL if available on your system.
*   **Usage**: `m3tal`
*   **Example**:
    ```bash
    m3tal
    # Or explicitly:
    m3tal ui
    ```

#### `m3tal headless [port]`
Starts M3TAL in headless mode. This is ideal for home servers or environments without a graphical interface. It will expose the M3TAL API.
*   **Usage**: `m3tal headless [port]`
*   **Example**:
    ```bash
    # Starts the M3TAL API daemon listening on its default port (8080)
    m3tal headless

    # Starts the M3TAL API daemon listening on a custom port, e.g., 5000
    m3tal headless 5000
    ```

#### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Useful for a quick glance at system health.
*   **Usage**: `m3tal stats`
*   **Example**:
    ```bash
    m3tal stats
    ```

#### `m3tal doctor`
Performs a pre-flight health check of your M3TAL environment. This command verifies Docker connectivity, checks the validity of `/etc/m3tal/.env`, and ensures required ports are available.
*   **Usage**: `m3tal doctor`
*   **Example**:
    ```bash
    m3tal doctor
    ```

### 3.2. Configuration Management

M3TAL centralizes its configuration in `/etc/m3tal/.env`. These commands help you manage it.

#### `m3tal config wizard`
Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended way to set up your environment variables.
*   **Usage**: `m3tal config wizard`
*   **Example**:
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`
Sets a single environment variable in `/etc/m3tal/.env`.
*   **Usage**: `m3tal config set KEY VALUE`
*   **Example**:
    ```bash
    # Set your primary domain name for Traefik routing
    m3tal config set DOMAIN myhomelab.com

    # Switch the dashboard to use Traefik for access
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`
Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.
*   **Usage**: `m3tal config get KEY`
*   **Example**:
    ```bash
    # Retrieve the configured PUID (User ID)
    m3tal config get PUID

    # Check the current dashboard exposure mode
    m3tal config get DASHBOARD_EXPOSE_MODE
    ```

#### `m3tal config scan`
Lists all environment variables known to M3TAL, including defaults and those defined in your `/etc/m3tal/.env`, across all active stacks.
*   **Usage**: `m3tal config scan`
*   **Example**:
    ```bash
    m3tal config scan
    ```

#### `m3tal config list`
Displays the current contents of your `/etc/m3tal/.env` file.
*   **Usage**: `m3tal config list`
*   **Example**:
    ```bash
    m3tal config list
    ```

### 3.3. Dashboard Management

These commands specifically interact with the M3TAL Dashboard container and its credentials.

#### `m3tal dashpass [username] [password]`
Updates the M3TAL Dashboard user password. If `username` and `password` are omitted, it will launch an interactive prompt. The credentials are stored in `/docker/users.json`.
*   **Usage**: `m3tal dashpass [username] [password]`
*   **Example**:
    ```bash
    # Update the 'admin' user's password interactively
    m3tal dashpass

    # Set the password for 'admin' directly (use with caution in scripts)
    m3tal dashpass admin MySuperSecurePass123!
    ```

#### `m3tal dash up`
Pulls the latest dashboard Docker Compose configuration from GitHub and then starts the M3TAL Dashboard container with the appropriate `DASHBOARD_EXPOSE_MODE` override.
*   **Usage**: `m3tal dash up`
*   **Example**:
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`
Stops and removes the M3TAL Dashboard container.
*   **Usage**: `m3tal dash down`
*   **Example**:
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`
Restarts the M3TAL Dashboard container. Useful after changing dashboard configuration or credentials.
*   **Usage**: `m3tal dash restart`
*   **Example**:
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`
Streams the logs from the M3TAL Dashboard container to your terminal. This is invaluable for debugging dashboard issues.
*   **Usage**: `m3tal dash logs`
*   **Example**:
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`
Shows the current operational status of the M3TAL Dashboard container (e.g., running, exited, restarting).
*   **Usage**: `m3tal dash status`
*   **Example**:
    ```bash
    m3tal dash status
    ```

### 3.4. Stack Management

These commands manage all your Docker Compose stacks defined in `/docker/`.

#### `m3tal up`
Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This starts or recreates all your defined services in detached mode.
*   **Usage**: `m3tal up`
*   **Example**:
    ```bash
    # Start all M3TAL core services and any user-defined stacks
    m3tal up
    ```

#### `m3tal down`
Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This stops and removes all containers, networks, and volumes defined in those stacks.
*   **Usage**: `m3tal down`
*   **Example**:
    ```bash
    # Stop and remove all M3TAL services and user stacks
    m3tal down
    ```

#### `m3tal logs`
Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of activity across your entire ecosystem.
*   **Usage**: `m3tal logs`
*   **Example**:
    ```bash
    m3tal logs
    ```

---

## 4. Systemd Service Management

The M3TAL API daemon runs as a `systemd` service named `m3tal-api.service`. You can manage it using standard `systemctl` commands.

*   **Check API Daemon Status**:
    ```bash
    sudo systemctl status m3tal-api
    ```
*   **Restart API Daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **View Live API Daemon Logs**:
    ```bash
    sudo journalctl -u m3tal-api -f
    ```
*   **Stop API Daemon**:
    ```bash
    sudo systemctl stop m3tal-api
    ```
*   **Enable API Daemon to start on boot**:
    ```bash
    sudo systemctl enable m3tal-api
    ```

---

## 5. Direct Docker Compose Fallback

M3TAL uses **Docker Engine** and **Docker Compose V2** under the hood. While the `m3tal` CLI abstracts many Docker commands, you might occasionally need to use `docker compose` directly for advanced debugging or specific container operations not covered by the `m3tal` CLI.

All M3TAL's Docker Compose files, including your custom stacks, reside in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).

*   **Prerequisite**: Ensure you are in the `/docker/` directory to run `docker compose` commands against your M3TAL stacks.

*   **List all running containers in M3TAL stacks**:
    ```bash
    cd /docker/
    docker compose ps
    ```

*   **Start a specific stack (e.g., `routing-compose.yml`)**:
    ```bash
    cd /docker/
    docker compose -f routing-compose.yml up -d
    ```

*   **View logs for a specific service within a stack (e.g., `traefik` from `routing-compose.yml`)**:
    ```bash
    cd /docker/
    docker compose -f routing-compose.yml logs -f traefik
    ```

*   **Stop and remove a specific stack (e.g., a custom `my-stack-compose.yml`)**:
    ```bash
    cd /docker/
    docker compose -f my-stack-compose.yml down
    ```

*   **Pull latest images for all services in all stacks**:
    ```bash
    cd /docker/
    docker compose pull
    ```

---

## 6. Key Configuration Paths & Ports

### 6.1 Important File Paths Revisited

*   **Primary Configuration**: `/etc/m3tal/.env`
*   **API State Database**: `/var/lib/m3tal/state.db`
*   **Stack Definitions**: `/docker/` (symlink to `/opt/m3tal/stack/`)
*   **Dashboard Credentials**: `/docker/users.json`
*   **Traefik Dynamic Config**: `/docker/dynamic/`

### 6.2 M3TAL Port Map

Understanding the port usage is vital for network configuration and firewall rules.

| Port | Service                          | Access                                      |
| :--- | :------------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point         | Public (when Traefik is running)            |
| 8080 | M3TAL API daemon (Go binary)     | Host-local only (internal communication)    |
| 8081 | Traefik dashboard                | Host-local only (`127.0.0.1:8081:8080`)     |
| 8082 | M3TAL Dashboard container        | Direct port (local mode) or via Traefik (traefik mode) |