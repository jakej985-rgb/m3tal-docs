```markdown
# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the comprehensive command-line interface (CLI) cheat-sheet for the M3TAL system. This document serves as your essential guide for managing, monitoring, and configuring your M3TAL instance, detailing every command and providing practical usage examples.

## M3TAL System Architecture Overview

M3TAL is a robust, modular platform designed for seamless home server management and beyond. It comprises several interconnected components:

*   **CLI binary (`/usr/bin/m3tal`)**: Your primary interface, a unified Go binary installed via APT, serving as the single entrypoint for all M3TAL operations.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a `systemd` service on host port `8080`. It orchestrates Docker operations, maintains the system's state database, and exposes API routes for internal and external communication.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask web application running internally on port `8082` within Docker. It communicates with the API daemon at `http://host.docker.internal:8080` to provide a user-friendly management interface.
*   **Traefik Gateway (`routing-compose.yml`)**: A reverse proxy container designed to expose your services, including the M3TAL Dashboard and API, via domain names on host port `80`. It dynamically discovers services using Docker labels and loads additional configuration via a file provider.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, enabling secure, zero-configuration internet access to your M3TAL services without direct port forwarding.

## Filesystem Contract

M3TAL establishes a clear filesystem contract for its operational files and configurations:

| Path                        | Purpose                                                                                             |
| :-------------------------- | :-------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file, containing environment variables crucial for M3TAL's operation. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | The SQLite state database, automatically created and managed by the API daemon.                     |
| `/opt/m3tal/stack/`         | The canonical directory for all Docker Compose files and Traefik dynamic configurations.            |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | The credential store for the M3TAL Dashboard, managed by `m3tal dashpass`.                        |

## M3TAL CLI Command Reference

This section details all available `m3tal` commands, their functionality, and concrete usage examples.

### Core M3TAL Commands

#### `m3tal` or `m3tal ui`

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where the GUI is desired.

```bash
m3tal
# or
m3tal ui
```

#### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for server environments without a graphical interface. The API daemon will listen on the specified port, or `8080` by default.

```bash
# Start headless mode on the default port (8080)
m3tal headless

# Start headless mode on a custom port (e.g., 9000)
m3tal headless 9000
```

#### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to the terminal, providing a quick overview of your system's health.

```bash
m3tal stats
```

#### `m3tal doctor`

Performs a pre-flight health check to diagnose common issues. It verifies Docker connectivity, validates the `/etc/m3tal/.env` file, and checks for port availability, helping you troubleshoot your setup.

```bash
m3tal doctor
```

### Configuration Management (`m3tal config`)

M3TAL uses environment variables in `/etc/m3tal/.env` for configuration. The `m3tal config` sub-commands provide a streamlined way to manage these settings.

#### `m3tal config wizard`

Launches an interactive wizard that guides you through configuring essential environment variables in `/etc/m3tal/.env`. This is the recommended way for initial setup or major configuration changes.

```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env` to a specified value.

```bash
# Set the primary domain for Traefik
m3tal config set DOMAIN "myhomeserver.net"

# Set the dashboard's expose mode to local
m3tal config set DASHBOARD_EXPOSE_MODE "local"
```

#### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

```bash
# Get the currently configured domain
m3tal config get DOMAIN
```

#### `m3tal config scan`

Lists all environment variables that M3TAL recognizes, across all its internal stacks and configurations. This helps in understanding available configuration options.

```bash
m3tal config scan
```

#### `m3tal config list`

Prints the current contents of the `/etc/m3tal/.env` file to the terminal, showing all configured environment variables and their values.

```bash
m3tal config list
```

### Dashboard Management (`m3tal dash`)

These commands are dedicated to managing the M3TAL Dashboard container and its associated settings.

#### `m3tal dashpass [username] [password]`

Updates the password for a specified dashboard user. If username and password are omitted, an interactive prompt will guide you through the process. The credentials are stored in `/docker/users.json`.

```bash
# Interactively set a new password for an existing user or create a new one
m3tal dashpass

# Set the password for user 'admin' non-interactively
m3tal dashpass admin "MyStrongSecurePassword123!"
```

#### `m3tal dash up`

Pulls the latest dashboard compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the M3TAL Dashboard container using the appropriate override file based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

```bash
m3tal dash up
```

#### `m3tal dash down`

Stops and removes the M3TAL Dashboard container and its associated network.

```bash
m3tal dash down
```

#### `m3tal dash restart`

Restarts the M3TAL Dashboard container, applying any new configurations or updates.

```bash
m3tal dash restart
```

#### `m3tal dash logs`

Streams the real-time logs from the M3TAL Dashboard container to your terminal, useful for debugging.

```bash
m3tal dash logs
```

#### `m3tal dash status`

Shows the current operational status of the M3TAL Dashboard container (e.g., running, stopped, unhealthy).

```bash
m3tal dash status
```

### Stack Management (`m3tal up`/`down`/`logs`)

These commands manage all Docker Compose stacks defined in the `/docker/` directory.

#### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command starts all your defined services (including Traefik and any user-added stacks) in detached mode.

```bash
m3tal up
```

#### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files found in `/docker/`. This command stops and removes all services managed by M3TAL.

```bash
m3tal down
```

#### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL, providing a consolidated view for monitoring and debugging.

```bash
m3tal logs
```

## Dashboard Access Modes (Critical Information)

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Mode 1: `local` (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the `m3tal-compose.local.yml` override, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access via**: `http://HOST_IP:8082` or `http://localhost:8082` (replace `HOST_IP` with your server's actual IP address).
*   **Benefits**: No Traefik required, works out of the box, ideal for LAN-only setups, first-time users, or local testing.

### Mode 2: `traefik`

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard container. Traefik then routes `dash.${DOMAIN}` to the dashboard's internal port `8082`.
*   **Access via**: `http://dash.DOMAIN` (e.g., `http://dash.myhomeserver.net`). Traefik must be running via `m3tal up` for this to work.
*   **Benefits**: Ideal for domain-based setups, integrating multiple services behind a single reverse proxy, and using features like HTTPS via Traefik.

## Docker / Compose Runtime Explained

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its underlying containerization and orchestration platform. These are hard dependencies for M3TAL's operation.

*   The `m3tal up` command abstracts `docker compose` operations, intelligently orchestrating all `*-compose.yml` files found within the `/docker/` directory.
*   The `m3tal dash up` command performs a specialized routine:
    1.  It first downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files from GitHub, ensuring your dashboard configuration is up-to-date.
    2.  It then reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Finally, it starts the M3TAL Dashboard container using the appropriate base compose file (`m3tal-compose.yml`) combined with the relevant override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).
*   **User Stacks**: You can integrate your own Docker Compose applications by simply placing their `*-compose.yml` files into the `/docker/` directory. M3TAL's `up` and `down` commands will then manage them alongside its core services.

## Deployment Lifecycle — Day 2 Operations

Integrating a new Docker Compose stack into your M3TAL ecosystem is straightforward:

1.  **Place your Compose File**: Copy your Docker Compose file (e.g., `my-cool-app-compose.yml`) into the `/docker/` directory.
    ```bash
    sudo cp my-cool-app-compose.yml /docker/
    ```
2.  **Configure Environment Variables**: If your new stack requires specific environment variables, set them in `/etc/m3tal/.env`. You can use `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific variables.
    ```bash
    m3tal config set COOL_APP_PORT 8000
    m3tal config set COOL_APP_DOMAIN "app.myhomeserver.net"
    ```
3.  **Start All Stacks**: Run `m3tal up` to start your new stack along with all existing M3TAL services.
    ```bash
    m3tal up
    ```

## Traefik Routing Architecture

Traefik acts as the intelligent reverse proxy within the M3TAL ecosystem, providing a unified entry point for your services:

*   **Deployment**: Traefik is deployed as a Docker container via `routing-compose.yml`.
*   **Entry Point**: It binds to host port `80`, serving as the primary HTTP entry point for all incoming web traffic.
*   **Service Discovery**: Traefik automatically discovers services by reading Docker labels applied to containers.
*   **Dynamic Configuration**: It loads additional dynamic configuration from YAML files within the `/docker/dynamic/` directory, leveraging Traefik's file provider for hot-reloading configurations without restarting Traefik itself.
*   **API Routing**: Traefik routes requests for `api.DOMAIN` (e.g., `api.myhomeserver.net`) directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080`. This is configured via a file like `/docker/dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.DOMAIN` (e.g., `dash.myhomeserver.net`) to the M3TAL Dashboard container, utilizing the Traefik labels added by `m3tal-compose.traefik.yml`.

## Systemd Service Management

The M3TAL API daemon (`m3tal-api`) runs as a `systemd` service, ensuring its persistent operation and proper lifecycle management.

*   **Check Status**: To view the current status of the API daemon:
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart Service**: To restart the API daemon (e.g., after modifying `/etc/m3tal/.env`):
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream Logs**: To view real-time logs from the API daemon for debugging:
    ```bash
    journalctl -u m3tal-api -f
    ```

## Direct Docker Compose Fallback Commands

While `m3tal up` and `m3tal down` are the recommended ways to manage your stacks, you can always use direct `docker compose` commands as a fallback or for more granular control.

Remember that M3TAL uses **Docker Engine + Docker Compose V2**.

1.  **Navigate to the Stacks Directory**: Always perform these commands from the `/docker/` directory, which symlinks to `/opt/m3tal/stack/`.
    ```bash
    cd /docker
    ```

2.  **Start Specific Stacks**: To bring up specific services or compose files:
    ```bash
    # Start only Traefik and Cloudflared (from routing-compose.yml)
    docker compose -f routing-compose.yml up -d

    # Start the M3TAL Dashboard in local mode (assuming DASHBOARD_EXPOSE_MODE=local is set in .env)
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d

    # Start a custom user stack
    docker compose -f my-app-compose.yml up -d
    ```

3.  **Stop Specific Stacks**: To bring down specific services or compose files:
    ```bash
    # Stop only Traefik and Cloudflared
    docker compose -f routing-compose.yml down

    # Stop the M3TAL Dashboard
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml down

    # Stop a custom user stack
    docker compose -f my-app-compose.yml down
    ```

4.  **View Logs for Specific Services**:
    ```bash
    # Stream logs for the Traefik container
    docker compose -f routing-compose.yml logs -f traefik

    # Stream logs for the M3TAL Dashboard container
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml logs -f m3tal-dashboard
    ```

## Port Map

Here's a quick reference for the standard ports used by M3TAL services:

| Port | Service                    | Access                                |
| :--- | :------------------------- | :------------------------------------ |
| 80   | Traefik HTTP entry point   | Public (when `DASHBOARD_EXPOSE_MODE=traefik`) |
| 8080 | M3TAL API daemon (Go)      | Host-local only                       |
| 8081 | Traefik dashboard          | Host-local only                       |
| 8082 | M3TAL Dashboard container  | Direct port (when `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (`DASHBOARD_EXPOSE_MODE=traefik`) |

## APT Installation

To install M3TAL on Debian/Ubuntu-based systems, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

This concludes the M3TAL CLI Command Reference. Should you require further assistance or encounter specific challenges, consult the M3TAL community or official documentation channels.