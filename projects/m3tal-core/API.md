# M3TAL CLI Command Reference

Welcome to the M3TAL Ecosystem Command Reference. As DocSmith, your dedicated M3TAL Ecosystem Documentation Architect, I've compiled this comprehensive guide to empower you with full control over your M3TAL environment. This document details every command of the `m3tal` CLI binary, providing practical usage examples and critical insights into the system's architecture.

## M3TAL System Architecture Overview

The M3TAL ecosystem is designed for robustness and ease of management, built upon a foundation of Docker and systemd. Understanding its core components and their interactions is key to leveraging its full potential.

### Components

*   **CLI binary (`/usr/bin/m3tal`)**: Your primary interface. This unified Go binary, installed via APT, acts as the single entrypoint for all M3TAL operations.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a systemd service, listening on port `8080`. It's the central brain, managing Docker interactions, maintaining the SQLite state database (`/var/lib/m3tal/state.db`), and exposing the API routes that the dashboard and other services communicate with.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask application container. It runs internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`)**: A powerful reverse proxy container. It exposes services via domain names on port `80` (and `443` if TLS is configured). Traefik uses a file provider for static routing (e.g., to the M3TAL API) and dynamic Docker labels for automatic service discovery.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, providing zero-config internet access to your services by securely connecting your M3TAL instance to the Cloudflare edge network without requiring open ports.

### Filesystem Contract

M3TAL establishes a clear filesystem contract to ensure predictable operation and easy management.

| Path                         | Purpose                                                                                                                                                                                                   |
| :--------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | **Primary configuration file**. This file holds all environment variables that configure your M3TAL deployment. It is primarily managed by `m3tal config wizard` and `m3tal config set`.                 |
| `/var/lib/m3tal/state.db`    | **SQLite state database**. Automatically created and managed by the M3TAL API daemon. Stores internal state, metrics, and other operational data.                                                           |
| `/opt/m3tal/stack/`          | **Canonical stack directory**. This is where M3TAL's core Docker Compose files (`m3tal-compose.yml`, `routing-compose.yml`, etc.) and Traefik dynamic configuration files reside.                         |
| `/docker`                    | **Symlink to `/opt/m3tal/stack/`**. This is the user-facing path for all Docker Compose stack operations. Any custom `*-compose.yml` files placed here will be managed by `m3tal up` and `m3tal down`. |
| `/docker/users.json`         | **Dashboard credential store**. This JSON file securely stores dashboard user accounts and their hashed passwords. It is managed exclusively by the `m3tal dashpass` command.                              |
| `/docker/dynamic/`           | **Traefik dynamic configuration directory**. Contains `.yml` files for services that need explicit routing rules, such as `api.DOMAIN` to the M3TAL API daemon. Hot-reloads changes.                    |

### Dashboard Access Modes

The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. Understanding these modes is critical for configuring your M3TAL instance correctly.

#### Mode 1: `local` (Default)

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik required. Works out of the box.
*   **Best for**: LAN-only setups, first-time users, quick local testing, environments without a domain name.

#### Mode 2: `traefik`

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik (running via `routing-compose.yml`) then routes `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (e.g., `http://dash.myhome.com`).
*   **Requirements**: Traefik must be running via `m3tal up`. You must have a `DOMAIN` configured in `/etc/m3tal/.env` and correct DNS records pointing to your host.
*   **Best for**: Domain-based setups, exposing services behind a central reverse proxy, integrating with other domain-routed services.

### Docker / Compose Runtime

M3TAL relies heavily on **Docker Engine** and **Docker Compose V2** for container orchestration. These are fundamental hard dependencies for the ecosystem.

*   The `m3tal up` command orchestrates all `*-compose.yml` files located in the `/docker/` directory, treating them as a single logical application.
*   The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It performs the following intelligent steps:
    1.  Downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from the official GitHub repository.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the appropriate compose override file based on the configured `DASHBOARD_EXPOSE_MODE`.
*   **User Stacks**: Your custom Docker Compose stacks should be placed as `*-compose.yml` files directly into the `/docker/` directory. M3TAL will automatically pick them up when you run `m3tal up`.

### Deployment Lifecycle — Day 2 Operations

Installing and managing new services or "stacks" within M3TAL is streamlined:

1.  **Place your compose file**: Copy your `docker-compose.yml` (or any `*-compose.yml` file) into the `/docker/` directory. For example: `/docker/my-stack-compose.yml`.
2.  **Configure environment variables**: Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`. You can use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for specific variables.
3.  **Start all stacks**: Run `m3tal up` to launch your new stack along with all other configured M3TAL services.

### Traefik Routing Architecture

Traefik acts as the intelligent ingress for your M3TAL services. It's deployed as a container via `routing-compose.yml` and is central to domain-based access.

*   **Entry Points**: Traefik binds port `80` on the host as its primary HTTP entry point (and `443` for HTTPS if configured).
*   **Service Discovery**: It automatically discovers and routes traffic to services by inspecting Docker labels on containers within the `proxy` network.
*   **Dynamic Configuration**: Traefik loads dynamic routing configuration from files within `/docker/dynamic/`. These files are watched for changes, enabling hot-reloads of routing rules.
    *   Example: `dynamic/api.yml` ensures that `api.DOMAIN` is routed directly to the M3TAL API daemon (running on `http://host.docker.internal:8080`).
    *   Example: When `DASHBOARD_EXPOSE_MODE=traefik`, the Traefik labels within `m3tal-compose.traefik.yml` instruct Traefik to route `dash.DOMAIN` to the dashboard container's internal port `8082`.

### Port Map

This table outlines the default ports used by M3TAL components and their accessibility.

| Port | Service                    | Access Mode                                         |
| :--- | :------------------------- | :-------------------------------------------------- |
| `80` | Traefik HTTP entry point   | Public (when Traefik is active and configured)      |
| `8080` | M3TAL API daemon (Go)      | Host-local only (accessed by dashboard and Traefik) |
| `8081` | Traefik dashboard          | Host-local only (e.g., `http://127.0.0.1:8081`)     |
| `8082` | M3TAL Dashboard container  | Direct port (in `local` mode) or via Traefik (`traefik` mode) |

## Installation

The M3TAL CLI binary is distributed via an APT repository, ensuring easy installation and updates on Debian/Ubuntu-based systems.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## CLI Command Reference

This section details every command available through the `m3tal` CLI.

### Core Application Commands

These commands launch or provide insights into the main M3TAL application.

#### `m3tal [ui]`

Launches the M3TAL Linux Desktop Application. This command starts the integrated graphical user interface (GUI) for local desktop environments.

*   **Usage**: `m3tal`
*   **Example**:
    ```bash
    m3tal
    # Or explicitly:
    m3tal ui
    ```

#### `m3tal headless [port]`

Starts M3TAL in headless mode, meaning the API daemon runs without a GUI. This is ideal for home servers or environments where direct GUI interaction is not needed. If `port` is omitted, it defaults to `8080`.

*   **Usage**: `m3tal headless [port]`
*   **Example**:
    ```bash
    m3tal headless
    # Starts the API daemon on its default port 8080.

    m3tal headless 9000
    # Starts the API daemon on port 9000.
    ```

#### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to the terminal. Provides a quick snapshot of your system's health and performance.

*   **Usage**: `m3tal stats`
*   **Example**:
    ```bash
    m3tal stats
    ```

### System Health and Diagnostics

These commands help you diagnose and ensure the healthy operation of your M3TAL instance.

#### `m3tal doctor`

Performs a pre-flight health check, verifying critical components such as Docker connectivity, the validity of your `/etc/m3tal/.env` configuration, and the availability of essential ports.

*   **Usage**: `m3tal doctor`
*   **Example**:
    ```bash
    m3tal doctor
    ```

### Configuration Management

M3TAL's configuration is primarily managed through `/etc/m3tal/.env`. These commands provide convenient ways to interact with this file.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through the configuration of your `/etc/m3tal/.env` file. This is recommended for first-time setups or significant configuration changes.

*   **Usage**: `m3tal config wizard`
*   **Example**:
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env` to a specified value. If the key already exists, its value will be updated.

*   **Usage**: `m3tal config set KEY VALUE`
*   **Example**:
    ```bash
    m3tal config set DOMAIN myhome.com
    m3tal config set PUID 1000
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

*   **Usage**: `m3tal config get KEY`
*   **Example**:
    ```bash
    m3tal config get PUID
    m3tal config get DASHBOARD_EXPOSE_MODE
    ```

#### `m3tal config scan`

Scans all M3TAL-managed Docker Compose files in `/docker/` and lists all environment variables referenced, including their default values. This helps identify all possible configuration options for your stacks.

*   **Usage**: `m3tal config scan`
*   **Example**:
    ```bash
    m3tal config scan
    ```

#### `m3tal config list`

Prints the entire contents of the current `/etc/m3tal/.env` file to the terminal, showing all configured environment variables.

*   **Usage**: `m3tal config list`
*   **Example**:
    ```bash
    m3tal config list
    ```

### Dashboard Management

These commands are specifically designed to manage the M3TAL Dashboard container and its user credentials.

#### `m3tal dashpass [username] [password]`

Updates a dashboard user's password. If `username` and `password` arguments are omitted, the command will launch an interactive prompt to guide you. This updates `/docker/users.json`.

*   **Usage**: `m3tal dashpass [username] [password]`
*   **Example**:
    ```bash
    m3tal dashpass admin newSecurePassword123
    # Sets the password for user 'admin' to 'newSecurePassword123'.

    m3tal dashpass
    # Launches an interactive prompt to set or update a password.
    ```

#### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files from GitHub and then starts the `m3tal-dashboard` container, applying the correct expose mode based on `/etc/m3tal/.env`.

*   **Usage**: `m3tal dash up`
*   **Example**:
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

*   **Usage**: `m3tal dash down`
*   **Example**:
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful for applying configuration changes or resolving minor issues.

*   **Usage**: `m3tal dash restart`
*   **Example**:
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`

Streams the aggregated logs from the `m3tal-dashboard` container to your terminal. Press `Ctrl+C` to stop streaming.

*   **Usage**: `m3tal dash logs`
*   **Example**:
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

*   **Usage**: `m3tal dash status`
*   **Example**:
    ```bash
    m3tal dash status
    ```

### Stack Management

These are the primary commands for orchestrating all Docker Compose stacks managed by M3TAL.

#### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all services defined in your M3TAL stacks, including the Traefik gateway if configured.

*   **Usage**: `m3tal up`
*   **Example**:
    ```bash
    m3tal up -d
    # Starts all services in detached mode (runs in the background).
    ```

#### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command gracefully stops and removes all containers, networks, and volumes associated with your M3TAL stacks.

*   **Usage**: `m3tal down`
*   **Example**:
    ```bash
    m3tal down
    ```

#### `m3tal logs`

Streams aggregated logs from all currently running M3TAL-managed Docker stacks to your terminal. This provides a unified view of activity across your entire ecosystem.

*   **Usage**: `m3tal logs`
*   **Example**:
    ```bash
    m3tal logs -f
    # Streams logs in real-time (follow mode).
    ```

### Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service. You can interact with it directly using standard `systemctl` commands.

*   **Check API daemon status**:
    ```bash
    systemctl status m3tal-api
    ```

*   **Restart API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```

*   **Stream API daemon logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```

### Direct Docker Compose Fallback

While `m3tal up` and `m3tal down` are the recommended ways to manage your stacks, you can always fall back to direct `docker compose` commands if needed. This is useful for advanced debugging or specific scenarios.

The M3TAL ecosystem's core compose files are located in `/docker/` (which is a symlink to `/opt/m3tal/stack/`).

*   **Start all M3TAL core services (including routing and dashboard in `local` mode):**
    ```bash
    # Ensure you are in the /docker/ directory or specify full paths
    cd /docker/
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml -f routing-compose.yml up -d
    ```
    *Note: The `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` should be selected based on your `DASHBOARD_EXPOSE_MODE` setting.*

*   **Stop all M3TAL core services:**
    ```bash
    cd /docker/
    docker compose -f m3tal-compose.yml -f routing-compose.yml down
    ```

*   **Stream logs from a specific service (e.g., Traefik):**
    ```bash
    docker compose -f /docker/routing-compose.yml logs -f traefik
    ```

*   **Inspect a specific service (e.g., m3tal-dashboard):**
    ```bash
    docker inspect m3tal-dashboard
    ```