# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, my mission is to provide you with a comprehensive, crystal-clear guide to managing your M3TAL instance. This document serves as your definitive cheat-sheet for the `m3tal` command-line interface (CLI), covering every essential operation from system setup to daily stack management.

## Welcome to the M3TAL Ecosystem

M3TAL is a unified platform designed to simplify the deployment and management of your self-hosted services. At its core, M3TAL orchestrates Docker containers using Docker Compose V2, providing a robust API (the `m3tal-api` daemon) and an intuitive web dashboard for control. The `m3tal` CLI binary acts as your primary interface, abstracting complex Docker commands into simple, powerful actions.

### Core Components & Architecture

*   **CLI binary (`/usr/bin/m3tal`)**: Your entry point for all M3TAL operations, installed via APT.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a systemd service (port 8080), handling Docker interactions, state management (SQLite), and API requests from the dashboard or CLI.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask web application container (internal port 8082), communicating with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway (`routing-compose.yml`)**: An optional, yet highly recommended, reverse proxy container. It exposes services via domain names on port 80/443, using Docker labels and dynamic file providers for configuration.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare Tunnel container for secure, zero-config internet exposure of your services without opening firewall ports.

### M3TAL Filesystem Contract

M3TAL maintains a strict filesystem contract to ensure consistent operation. Understanding these paths is crucial:

| Path                        | Purpose                                                                                                                                                                                                    |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary Configuration File**. Stores all M3TAL environment variables. Managed by `m3tal config wizard` and `m3tal config set`.                                                                          |
| `/var/lib/m3tal/state.db`   | **SQLite State Database**. Automatically created and managed by the `m3tal-api` daemon. Stores internal state, user data, and service information.                                                          |
| `/opt/m3tal/stack/`         | **Canonical Stack Directory**. This is the authoritative location for M3TAL's internal Docker Compose files and Traefik configuration. **Do not modify directly.**                                        |
| `/docker`                   | **User-Facing Stack Directory (Symlink)**. A symlink to `/opt/m3tal/stack/`. This is where you, the user, should place all your custom `*-compose.yml` files and Traefik dynamic configuration files. |
| `/docker/users.json`        | **Dashboard Credential Store**. Stores encrypted username/password pairs for the M3TAL Dashboard. Managed by `m3tal dashpass`.                                                                             |
| `/docker/dynamic/`          | Directory within `/docker` for Traefik dynamic configuration files (e.g., `api.yml`). These files are watched by Traefik for hot-reloading.                                                              |
| `/var/log/m3tal-api/`       | Directory where `m3tal-api.service` stores its logs.                                                                                                                                                       |

### Dashboard Access Modes

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard container's port to the host machine.
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requirements**: No Traefik required. Works out-of-the-box.
*   **Best for**: LAN-only setups, first-time users, local testing, or scenarios where Traefik is not desired for dashboard access.

#### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard container. Traefik (if running) will then route requests for `dash.${DOMAIN}` to the dashboard.
*   **Access**: `http://dash.YOUR_DOMAIN` (e.g., `http://dash.homelab.local`)
*   **Requirements**: Traefik must be running (via `m3tal up`). Your `DOMAIN` variable in `/etc/m3tal/.env` must be correctly set. DNS resolution for `dash.YOUR_DOMAIN` must point to your M3TAL host.
*   **Best for**: Domain-based setups, integrating the dashboard with other services behind Traefik, using a single entry point for all web services.

### Docker / Compose Runtime Explained

M3TAL leverages **Docker Engine** and **Docker Compose V2** for all container orchestration. These are fundamental dependencies for M3TAL to function.

*   **`m3tal up`**: This command intelligently scans `/docker/` for all files ending in `-compose.yml` and executes `docker compose up -d` across all of them simultaneously. This is how you start *all* your managed services, including Traefik, Cloudflared, and any custom user stacks.
*   **`m3tal dash up`**: This command is specialized for the M3TAL Dashboard. It performs the following steps:
    1.  Pulls the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files directly from the M3TAL GitHub repository into `/docker/`.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the `m3tal-dashboard` container, applying the appropriate override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` setting.

### Deployment Lifecycle: Day 2 Operations

To add a new self-hosted stack to M3TAL's management:

1.  **Place your Compose file**: Copy your `docker-compose.yml` file into `/docker/` and rename it with the `*-compose.yml` suffix (e.g., `/docker/my-app-compose.yml`).
2.  **Configure Environment Variables**: Ensure all necessary environment variables for your new stack (and any M3TAL-wide variables) are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` for guided setup or `m3tal config set KEY VALUE` for direct edits.
3.  **Start the Stack**: Run `m3tal up`. M3TAL will detect your new compose file and bring up the services defined within it, alongside any other existing stacks.

### Traefik Routing Architecture

Traefik, if enabled, acts as your reverse proxy. It's deployed via `routing-compose.yml` and operates as follows:

*   **Entry Points**: Binds to host port `80` (and `443` if TLS is configured) for HTTP/HTTPS requests.
*   **Service Discovery**: Automatically discovers services running in Docker based on their labels.
*   **Dynamic Configuration**: Loads additional routing rules from `.yml` files placed in `/docker/dynamic/`. These files are hot-reloaded without needing a Traefik restart.
    *   **API Routing**: An example is routing `api.YOUR_DOMAIN` to the M3TAL API daemon (Go service) running on `http://host.docker.internal:8080` via a file like `/docker/dynamic/api.yml`.
    *   **Dashboard Routing**: If `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels on the dashboard container route `dash.YOUR_DOMAIN` to the dashboard.

### M3TAL Port Map

Understanding M3TAL's port usage is essential for network configuration:

| Port | Service                                | Access                                                                 |
| :--- | :------------------------------------- | :--------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point               | Public (when Traefik is active, routes services by domain)             |
| 443  | Traefik HTTPS entry point              | Public (when Traefik is active and TLS configured)                     |
| 8080 | M3TAL API daemon (Go)                  | Host-local only (or via Traefik routing `api.DOMAIN`)                  |
| 8081 | Traefik dashboard                      | Host-local only (`127.0.0.1:8081` by default), provides Traefik metrics |
| 8082 | M3TAL Dashboard (container internal) | Direct port (local mode) or via Traefik (traefik mode)                 |

---

## M3TAL Installation

The M3TAL CLI is distributed as a native package for Debian/Ubuntu systems.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## M3TAL CLI Command Reference

This section details every `m3tal` command, providing its purpose and a practical usage example.

### Core M3TAL Commands

#### `sudo m3tal`

Opens the interactive TUI Control Center. This provides a user-friendly, menu-driven interface for common M3TAL operations, often simplifying multi-step processes.

*   **Description**: Launches the M3TAL Text User Interface (TUI), offering a numbered menu to perform operations like starting/stopping services, managing configurations, and viewing status. Requires `sudo` for full functionality.
*   **Usage Example**:
    ```bash
    sudo m3tal
    ```
    (This will present a menu like: `1. Start M3TAL Stacks`, `2. Stop M3TAL Stacks`, etc.)

#### `m3tal init`

Generates the primary configuration file, `/etc/m3tal/.env`, from M3TAL's default settings. Essential for a first-time setup or recovering a missing `.env` file.

*   **Description**: Initializes the M3TAL system by creating a default `/etc/m3tal/.env` file. This file contains critical environment variables used by the `m3tal-api` daemon and all Docker Compose stacks.
*   **Usage Example**:
    ```bash
    m3tal init
    ```

#### `m3tal doctor`

Performs a pre-flight health check of your M3TAL system.

*   **Description**: Diagnoses common issues before starting services. It checks Docker daemon connectivity, validates the `/etc/m3tal/.env` file for syntax and required variables, and verifies that essential ports are not already in use by other processes.
*   **Usage Example**:
    ```bash
    m3tal doctor
    ```
    (Expected output might include: `[PASS] Docker daemon reachable`, `[PASS] /etc/m3tal/.env is valid`, `[WARN] Port 80 might be in use by Nginx.`)

### Configuration Management (`m3tal config`)

These commands manage the primary configuration file located at `/etc/m3tal/.env`.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring `/etc/m3tal/.env`.

*   **Description**: Provides a guided, step-by-step process to set or update environment variables in `/etc/m3tal/.env`. This is the recommended way to configure M3TAL after `m3tal init`. It prompts for values, explains their purpose, and validates input where possible.
*   **Usage Example**:
    ```bash
    m3tal config wizard
    ```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`.

*   **Description**: Directly updates or adds a specific key-value pair in your M3TAL configuration file. Useful for quick, targeted changes.
*   **Usage Example**:
    ```bash
    m3tal config set DOMAIN myhomelab.local
    m3tal config set DASHBOARD_EXPOSE_MODE traefik
    ```

#### `m3tal config get KEY`

Reads and displays the value of a single environment variable from `/etc/m3tal/.env`.

*   **Description**: Retrieves and prints the current value associated with a specified key from your M3TAL configuration.
*   **Usage Example**:
    ```bash
    m3tal config get DASHBOARD_PORT
    # Expected output: 8082
    ```

#### `m3tal config scan`

Lists all environment variables known to M3TAL, showing their current values across all stacks.

*   **Description**: Scans all `*-compose.yml` files in `/docker/` for declared environment variables, cross-referencing them with the values in `/etc/m3tal/.env`. It provides a comprehensive overview of all configuration variables in use by your M3TAL ecosystem.
*   **Usage Example**:
    ```bash
    m3tal config scan
    ```

#### `m3tal config list`

Displays the entire content of the current `/etc/m3tal/.env` file.

*   **Description**: Prints the raw contents of your primary M3TAL configuration file. This is useful for reviewing all settings at once.
*   **Usage Example**:
    ```bash
    m3tal config list
    ```

### Dashboard Management (`m3tal dash`)

These commands are specifically for managing the M3TAL Dashboard.

#### `m3tal dashpass [username] [password]`

Updates or creates a user password for the M3TAL Dashboard.

*   **Description**: Manages user credentials stored in `/docker/users.json`. If `username` and `password` are provided, it performs a non-interactive update. If omitted, it guides you through an interactive prompt. Passwords are securely hashed.
*   **Usage Example (Interactive)**:
    ```bash
    m3tal dashpass
    # Prompts for Username, then Password, then Confirm Password.
    ```
*   **Usage Example (Direct)**:
    ```bash
    m3tal dashpass admin SuperSecurePassword123
    ```

#### `m3tal dash up`

Pulls the latest dashboard compose configuration and starts the dashboard container.

*   **Description**: This is the command to bring up the M3TAL Dashboard. It first ensures you have the latest `m3tal-compose.yml` and its overrides (`local.yml`, `traefik.yml`) by pulling them from GitHub. It then starts the `m3tal-dashboard` container using the appropriate override based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.
*   **Usage Example**:
    ```bash
    m3tal dash up
    ```

#### `m3tal dash down`

Stops the M3TAL Dashboard container.

*   **Description**: Gracefully stops and removes the `m3tal-dashboard` container.
*   **Usage Example**:
    ```bash
    m3tal dash down
    ```

#### `m3tal dash restart`

Restarts the M3TAL Dashboard container.

*   **Description**: Stops and then immediately starts the `m3tal-dashboard` container. Useful after configuration changes or if the dashboard becomes unresponsive.
*   **Usage Example**:
    ```bash
    m3tal dash restart
    ```

#### `m3tal dash logs`

Streams logs from the M3TAL Dashboard container.

*   **Description**: Displays real-time log output from the `m3tal-dashboard` container, useful for debugging dashboard-specific issues.
*   **Usage Example**:
    ```bash
    m3tal dash logs
    ```

#### `m3tal dash status`

Shows the current status of the M3TAL Dashboard container.

*   **Description**: Reports whether the `m3tal-dashboard` container is running, stopped, or in another state, along with its uptime and other relevant Docker information.
*   **Usage Example**:
    ```bash
    m3tal dash status
    ```

### Stack Management (`m3tal up`, `m3tal down`, `m3tal logs`)

These commands manage all Docker Compose stacks defined by `*-compose.yml` files in `/docker/`.

#### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files in `/docker/`.

*   **Description**: This is the primary command to bring up all your M3TAL-managed services. It iterates through every file ending with `-compose.yml` in the `/docker/` directory (including `routing-compose.yml`, `m3tal-compose.yml`, and your custom stacks) and starts their respective containers in detached mode.
*   **Usage Example**:
    ```bash
    m3tal up
    ```
    (This will start Traefik, Cloudflared, and any other services you've placed in `/docker/`.)

#### `m3tal down`

Runs `docker compose down` across all stacks.

*   **Description**: Stops and removes all containers, networks, and volumes defined by all `*-compose.yml` files in `/docker/`. Use with caution as this will stop all M3TAL-managed services.
*   **Usage Example**:
    ```bash
    m3tal down
    ```

#### `m3tal logs`

Streams aggregated logs from all running M3TAL-managed stacks.

*   **Description**: Provides a consolidated, real-time view of logs from *all* active Docker containers managed by M3TAL. This is incredibly useful for monitoring the health and activity of your entire ecosystem.
*   **Usage Example**:
    ```bash
    m3tal logs
    ```

---

## Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service, providing the backend for the CLI and Dashboard. You can manage it directly using `systemctl` and `journalctl`.

*   **Check API Status**: Verify if the API daemon is running and healthy.
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart API Daemon**: Apply changes to `/etc/m3tal/.env` that require the API to reload, or troubleshoot issues.
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream API Logs**: View real-time logs from the `m3tal-api` daemon.
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Enable/Disable API at Boot**:
    ```bash
    sudo systemctl enable m3tal-api
    sudo systemctl disable m3tal-api
    ```

---

## Direct Docker Compose Commands (Fallback / Advanced)

While `m3tal` abstracts many Docker commands, understanding the underlying `docker compose` operations can be useful for advanced debugging or when M3TAL's CLI isn't available. Remember, M3TAL's `up`, `down`, and `logs` commands handle *all* compose files in `/docker/`. For granular control, you'd target specific files.

*   **Start a specific stack (e.g., Traefik & Cloudflared)**:
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml up -d
    ```
    *Note: `m3tal up` does this for all files automatically, including handling overrides like `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`.*

*   **Stop a specific stack**:
    ```bash
    docker compose -f /docker/my-custom-stack-compose.yml down
    ```

*   **View logs for a specific stack (e.g., M3TAL Dashboard)**:
    ```bash
    docker compose -f /docker/m3tal-compose.yml logs -f
    ```

*   **List all running containers managed by Docker Compose**:
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack-compose.yml ps
    ```
    *Note: You would need to manually list all compose files that define running services.*

*   **Update and restart a specific container (e.g., `m3tal-dashboard`)**:
    ```bash
    docker pull ghcr.io/jakej985-rgb/m3tal-godash:debug
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml restart m3tal-dashboard
    ```
    *Note: `m3tal dash up` handles image pulling and dynamic override selection automatically.*

By understanding both the `m3tal` CLI and its underlying Docker Compose operations, you are fully equipped to manage your M3TAL ecosystem with confidence and precision. Happy self-hosting!