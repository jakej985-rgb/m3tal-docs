`docs/COMMAND_REFERENCE.md`

# M3TAL CLI Command Reference

Greetings, fellow sysadmin and architect of your digital domain! I am DocSmith, the M3TAL Ecosystem Documentation Architect, and you hold in your hands the definitive guide to harnessing the power of the M3TAL CLI. This document is your cheat-sheet, your quick-reference, your bedrock of operational knowledge for managing your M3TAL server.

M3TAL is designed to simplify the deployment and management of self-hosted services using Docker and Docker Compose. It provides a unified command-line interface (`m3tal`) to control an underlying API daemon, manage configurations, and orchestrate your entire stack.

---

## M3TAL Installation

The `m3tal` CLI binary is distributed via an APT repository for Debian/Ubuntu-based systems. Follow these steps to install it:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## M3TAL System Architecture Overview

Understanding the fundamental components and their interactions is crucial for effective M3TAL management.

### Components

*   **CLI binary** (`/usr/bin/m3tal`): Your primary interface. A unified Go binary installed via APT, acting as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): The heart of M3TAL. This Go binary runs as a systemd service, typically listening on port `8080`. It's responsible for managing Docker containers, maintaining the state database, and exposing the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running inside Docker, usually on internal port `8082`. It communicates with the `m3tal-api` daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): Your intelligent reverse proxy. This Docker container exposes services by domain name on port `80` (and `443` for HTTPS). It uses a file provider for dynamic routing and automatically discovers Docker services via labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, offering zero-configuration internet access for your services without opening ports on your router.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract to ensure consistency and predictability.

| Path                         | Purpose                                                                          |
| :--------------------------- | :------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | The primary configuration file, containing all environment variables for M3TAL and your stacks. Managed primarily by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`    | The SQLite state database where M3TAL stores its internal state, container information, and more. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`          | The canonical directory for all Docker Compose files and Traefik dynamic configurations. |
| `/docker`                    | A user-friendly symlink pointing to `/opt/m3tal/stack/`. This is the recommended path for users to place their `*-compose.yml` files. |
| `/docker/users.json`         | The credential store for the M3TAL Dashboard, managed by `m3tal dashpass`. |

### Dashboard Access — Two Modes

The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which adds a direct host port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: You access the dashboard directly via your host's IP address or `localhost` on the specified port.
    *   **Example**: `http://YOUR_HOST_IP:8082` or `http://localhost:8082`
*   **Best for**: LAN-only setups, first-time users, environments without a reverse proxy, and local testing. Traefik is *not* required for this mode.

#### Mode 2: `traefik`

*   **Configuration**: Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override. This adds Traefik-specific labels to the dashboard container, allowing Traefik to automatically route requests (e.g., `dash.${DOMAIN}`) to the dashboard's internal port `8082`.
*   **Access**: You access the dashboard via a domain name configured through Traefik.
    *   **Example**: `http://dash.yourdomain.com` (assuming `DOMAIN=yourdomain.com` in `.env` and Traefik is running via `m3tal up`).
*   **Best for**: Domain-based access, environments using Traefik as a central reverse proxy for multiple services, and production deployments.

### Docker / Compose Runtime

M3TAL relies heavily on **Docker Engine** and **Docker Compose V2** as its foundational technologies. These are hard dependencies for `m3tal` to function.

*   The `m3tal up` command orchestrates all Docker Compose files by running `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory.
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container. It first ensures the latest compose configurations (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) are pulled from GitHub, then starts the dashboard using the appropriate override file based on your `DASHBOARD_EXPOSE_MODE` setting.
*   You can extend your M3TAL ecosystem by placing any standard Docker Compose file (named `*-compose.yml`) directly into the `/docker/` directory.

### Deployment Lifecycle — Day 2 Operations

Installing and managing new services with M3TAL is straightforward:

1.  **Place your compose file**: Copy your `docker-compose.yml` file to `/docker/my-new-service-compose.yml`.
2.  **Configure variables**: Ensure all necessary environment variables for your new service are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` for an interactive setup or `m3tal config set KEY value` for specific variables.
3.  **Start your stacks**: Run `m3tal up` to bring up all services, including your newly added stack.

### Traefik Routing Architecture

Traefik acts as the central entry point for external access to your services.

*   **Deployment**: Traefik is deployed as a Docker container via `routing-compose.yml`.
*   **Entry Points**: It binds to port `80` (and typically `443` for HTTPS) on the host, serving as the HTTP entry point (`web`).
*   **Service Discovery**: Traefik automatically discovers and configures routes for services based on Docker labels.
*   **Dynamic Configuration**: It loads additional dynamic configurations from `/docker/dynamic/` (which is symlinked to `/opt/m3tal/stack/dynamic/`) using a file provider. Changes to these files are hot-reloaded.
*   **API Routing**: Traefik routes requests for `api.${DOMAIN}` to the `m3tal-api` daemon running on the host at `http://host.docker.internal:8080` via `dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` to the `m3tal-dashboard` container using labels defined in `m3tal-compose.traefik.yml`.

### Port Map

| Port | Service                                | Access                 |
| :--- | :------------------------------------- | :--------------------- |
| 80   | Traefik HTTP entry point               | Public (Traefik mode)  |
| 443  | Traefik HTTPS entry point              | Public (Traefik mode)  |
| 8080 | M3TAL API daemon (Go)                  | Host-local             |
| 8081 | Traefik dashboard (internal)           | Host-local only (127.0.0.1:8081) |
| 8082 | M3TAL Dashboard (internal container) | Direct port (local mode) or via Traefik (traefik mode) |

---

## CLI Command Reference

This section details every `m3tal` command, complete with a practical usage example.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application. This is your graphical interface for managing and monitoring your M3TAL ecosystem.

```bash
# Launch the M3TAL desktop application
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for home servers or environments without a graphical interface. The API daemon will listen on the specified port. If no port is provided, it defaults to `8080`.

```bash
# Start M3TAL API in headless mode, listening on port 8080
sudo m3tal headless

# Start M3TAL API in headless mode, listening on a custom port 8090
sudo m3tal headless 8090
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. A quick snapshot of your server's health.

```bash
# Get an immediate readout of system statistics
m3tal stats
```

### `m3tal doctor`

Performs a crucial pre-flight health check. It verifies Docker connectivity, validates the `/etc/m3tal/.env` configuration, and checks for port availability, helping diagnose common setup issues.

```bash
# Run a diagnostic check on the M3TAL environment
m3tal doctor
```

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL environment variables.

```bash
# Start the interactive configuration wizard
sudo m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env`. This is useful for making quick, specific changes without the wizard.

```bash
# Set the domain for Traefik routing
sudo m3tal config set DOMAIN myhomelab.local

# Change the dashboard expose mode to Traefik
sudo m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

```bash
# Retrieve the currently configured dashboard port
m3tal config get DASHBOARD_PORT
```

### `m3tal config scan`

Lists all environment variables that M3TAL recognizes, including their current values and default values across all configured stacks. Provides a comprehensive overview of your active configuration.

```bash
# Display all environment variables used by M3TAL and its stacks
m3tal config scan
```

### `m3tal config list`

Displays the current contents of the `/etc/m3tal/.env` file.

```bash
# View the raw contents of the primary configuration file
m3tal config list
```

### `m3tal dashpass [username] [password]`

Updates the password for a user on the M3TAL Dashboard. If no username and password are provided, it will launch an interactive prompt. This manages `/docker/users.json`.

```bash
# Interactively set or update dashboard user passwords
m3tal dashpass

# Set the password for the 'admin' user directly
m3tal dashpass admin MySuperSecurePassword123!
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub, then starts the `m3tal-dashboard` container based on your `DASHBOARD_EXPOSE_MODE`.

```bash
# Ensure the dashboard is running with the latest configuration
m3tal dash up
```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

```bash
# Stop the M3TAL dashboard service
m3tal dash down
```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.

```bash
# Restart the dashboard to apply new settings or clear issues
m3tal dash restart
```

### `m3tal dash logs`

Streams the aggregated logs from the `m3tal-dashboard` container directly to your terminal. Essential for debugging dashboard-related issues.

```bash
# View real-time logs for the M3TAL dashboard
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container.

```bash
# Check if the M3TAL dashboard is running
m3tal dash status
```

### `m3tal up`

Executes `docker compose up -d` across all `*-compose.yml` files located in the `/docker/` directory. This command starts or recreates all services defined in your M3TAL ecosystem.

```bash
# Bring up all Docker Compose stacks in the /docker/ directory
m3tal up
```

### `m3tal down`

Executes `docker compose down` across all `*-compose.yml` files in the `/docker/` directory, stopping and removing all associated containers, networks, and volumes (if not managed externally).

```bash
# Stop and remove all running Docker Compose stacks
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of all service activity.

```bash
# Stream combined logs from all active M3TAL-managed containers
m3tal logs
```

---

## Service Management (systemd)

The `m3tal-api` daemon runs as a systemd service. You can manage it using standard `systemctl` commands.

*   **Check status**: View the current state of the M3TAL API service.
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart service**: Restart the M3TAL API daemon (useful after `m3tal config set` changes).
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stop service**: Stop the M3TAL API daemon.
    ```bash
    sudo systemctl stop m3tal-api
    ```
*   **Start service**: Start the M3TAL API daemon.
    ```bash
    sudo systemctl start m3tal-api
    ```
*   **View live logs**: Stream real-time logs from the M3TAL API daemon.
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Direct Docker / Compose Fallback

M3TAL leverages Docker Engine and Docker Compose V2. While the `m3tal` CLI abstracts many commands, you can always fall back to direct `docker` and `docker compose` commands for advanced debugging or specific operations.

The core Docker Compose files are located in `/docker/` (symlinked from `/opt/m3tal/stack/`).

*   **List all active compose projects and their statuses**:
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack-compose.yml ps
    ```
    *(Note: You'll need to explicitly list all relevant compose files for `docker compose` when not in their directory.)*

*   **Bring up a specific stack (e.g., just routing)**:
    ```bash
    sudo docker compose -f /docker/routing-compose.yml up -d
    ```

*   **Stop a specific service (e.g., Traefik)**:
    ```bash
    sudo docker stop traefik
    ```

*   **View logs for a specific container (e.g., M3TAL dashboard)**:
    ```bash
    docker logs -f m3tal-dashboard
    ```

*   **Inspect a container's configuration**:
    ```bash
    docker inspect m3tal-dashboard
    ```

*   **Access a shell inside a container**:
    ```bash
    docker exec -it m3tal-dashboard /bin/bash
    ```

Remember that when using `docker compose` directly, you often need to be in the directory containing the `docker-compose.yml` files (e.g., `/docker/`) or specify them with `-f`. The `m3tal` CLI handles this context automatically for you.