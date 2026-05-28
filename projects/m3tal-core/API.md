# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, I present the definitive guide to managing your M3TAL environment. This cheat-sheet covers every essential command, offering practical examples and diving deep into the architecture that powers your M3TAL instance.

## Introduction to M3TAL

M3TAL is a unified ecosystem for deploying and managing services on your server. It leverages Docker Engine and Docker Compose V2, providing a Go-based CLI (`/usr/bin/m3tal`) and an API daemon (`m3tal-api.service`) to orchestrate your applications. The system prioritizes simplicity, security, and maintainability, ensuring your services are always running smoothly.

## APT Installation

To install the M3TAL CLI and API daemon on Debian-based systems, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract

Understanding the M3TAL filesystem is crucial for configuration and troubleshooting.

| Path                     | Purpose                                                      |
|--------------------------|--------------------------------------------------------------|
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the API daemon.       |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                | **Symlink** → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. Place your `*-compose.yml` files here. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.     |

---

## M3TAL CLI Commands

The `m3tal` command is your primary interface for interacting with the M3TAL ecosystem. All commands communicate with the `m3tal-api.service` daemon, which in turn manages your Docker containers.

### Core M3TAL Commands

#### `sudo m3tal`

Opens the interactive TUI (Terminal User Interface) Control Center. This provides a user-friendly, numbered menu for common operations and system status.
This command requires `sudo` privileges to interact directly with the M3TAL API daemon.

**Usage Example:**
```bash
sudo m3tal
```

#### `m3tal init`

Generates the primary configuration file, `/etc/m3tal/.env`, from system defaults. This command should be run on first installation or if the `.env` file is missing. It's a non-destructive operation for existing `.env` files, typically prompting before overwriting.

**Usage Example:**
```bash
m3tal init
```

#### `m3tal doctor`

Performs a pre-flight health check of your M3TAL system. This includes verifying Docker connectivity, validating the `/etc/m3tal/.env` file, and checking for port availability to prevent conflicts. Essential for diagnosing setup issues.

**Usage Example:**
```bash
m3tal doctor
```

### Configuration Management

M3TAL's configuration is driven by `/etc/m3tal/.env`. These commands allow you to manage environment variables used by the M3TAL API and your Docker stacks.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended method for initial setup and extensive configuration changes.

**Usage Example:**
```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env` to the specified value. This is useful for quick, targeted changes.

**Usage Example:**
```bash
m3tal config set DOMAIN "my.example.com"
```
*(This will set `DOMAIN=my.example.com` in your .env file)*

#### `m3tal config get KEY`

Retrieves and displays the current value of a specific environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config get PUID
```
*(This might output `1000`, for instance)*

#### `m3tal config scan`

Lists all environment variables known to the M3TAL ecosystem, including those defined in `/etc/m3tal/.env` and any defaults relevant to deployed stacks. This provides a comprehensive overview of your system's configuration context.

**Usage Example:**
```bash
m3tal config scan
```

#### `m3tal config list`

Displays the entire contents of the current primary configuration file, `/etc/m3tal/.env`. This is useful for reviewing all active settings at once.

**Usage Example:**
```bash
m3tal config list
```

### Dashboard Management

The M3TAL Dashboard provides a web-based interface for monitoring and managing your services. These commands specifically control the `m3tal-dashboard` container.

#### `m3tal dashpass [username] [password]`

Updates the password for a dashboard user. If `username` and `password` are omitted, the command becomes interactive, prompting you for the details. By default, the dashboard uses `admin` as the initial username. This command manages the `/docker/users.json` file.

**Usage Examples:**

*   **Interactive (recommended):**
    ```bash
    m3tal dashpass
    ```
*   **Direct (non-interactive):**
    ```bash
    m3tal dashpass admin new_secure_password123
    ```

#### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts or updates the `m3tal-dashboard` container based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal dash up
```

#### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container and its associated resources.

**Usage Example:**
```bash
m3tal dash down
```

#### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. This is useful after making configuration changes that affect the dashboard.

**Usage Example:**
```bash
m3tal dash restart
```

#### `m3tal dash logs`

Streams the logs from the `m3tal-dashboard` container in real-time. Useful for debugging dashboard-related issues.

**Usage Example:**
```bash
m3tal dash logs
```

#### `m3tal dash status`

Displays the current running status of the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash status
```

### Stack Management

These commands manage all Docker Compose stacks defined by `*-compose.yml` files located in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`).

#### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command starts or updates all your configured services in detached mode. This includes core M3TAL stacks (like Traefik if configured) and any custom stacks you've added.

**Usage Example:**
```bash
m3tal up
```

#### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command stops and removes all containers, networks, and volumes defined by your Docker Compose files.

**Usage Example:**
```bash
m3tal down
```

#### `m3tal logs`

Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of all service activity.

**Usage Example:**
```bash
m3tal logs
```

---

## Dashboard Access Modes (Critical)

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. `m3tal dash up` automatically selects the correct Docker Compose override file based on this setting.

### Mode 1: `local` (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the Docker Compose override file: `m3tal-compose.local.yml`.
*   Adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **No Traefik required.** Works out of the box for quick setup.
*   **Best for:** LAN-only setups, first-time users, local testing, or when Traefik is not desired for the dashboard.

**Configuration Example (`/etc/m3tal/.env`):**
```dotenv
DASHBOARD_EXPOSE_MODE=local
DASHBOARD_PORT=8082
```

### Mode 2: `traefik`

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the Docker Compose override file: `m3tal-compose.traefik.yml`.
*   Adds Traefik labels to the dashboard container, allowing Traefik to route `dash.${DOMAIN}` to the dashboard on port `8082`.
*   **Access via:** `http://dash.YOUR_DOMAIN`.
*   **Requires Traefik to be running** (via `m3tal up` for the `routing-compose.yml` stack).
*   **Best for:** Domain-based setups, integrating the dashboard behind your central reverse proxy, and using custom domains.

**Configuration Example (`/etc/m3tal/.env`):**
```dotenv
DASHBOARD_EXPOSE_MODE=traefik
DOMAIN=my.example.com
```
*(Access would then be `http://dash.my.example.com`)*

---

## M3TAL API Daemon (systemd Service Management)

The core M3TAL API daemon runs as a `systemd` service, `m3tal-api.service`. This daemon manages all Docker interactions and provides the backend for the CLI.

#### `systemctl status m3tal-api`

Checks the current status of the M3TAL API daemon. This shows if it's active, running, and its most recent logs.

**Usage Example:**
```bash
systemctl status m3tal-api
```

#### `systemctl restart m3tal-api`

Restarts the M3TAL API daemon. This is often necessary after manual changes to `/etc/m3tal/.env` that might affect the API's behavior (though most `m3tal config` commands attempt to signal the API without a full restart).

**Usage Example:**
```bash
sudo systemctl restart m3tal-api
```

#### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. This is invaluable for debugging issues with the M3TAL API itself.

**Usage Example:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Commands (Fallback)

While the `m3tal` CLI is the preferred way to manage your stacks, understanding the underlying Docker Compose commands can be useful for advanced debugging or when working outside the M3TAL ecosystem. All `*-compose.yml` files are located in `/docker/`.

#### Start all M3TAL-managed stacks:

```bash
docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-stack-compose.yml up -d
```
*(Adjust `-f` flags for all your active compose files)*

#### Stop all M3TAL-managed stacks:

```bash
docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-stack-compose.yml down
```
*(Adjust `-f` flags for all your active compose files)*

#### View logs for a specific service (e.g., `m3tal-dashboard`):

```bash
docker compose -f /docker/m3tal-compose.yml logs -f m3tal-dashboard
```

#### Inspect status of a specific service:

```bash
docker compose -f /docker/m3tal-compose.yml ps m3tal-dashboard
```

#### Pull latest images for all services in a stack:

```bash
docker compose -f /docker/routing-compose.yml pull
```

---

## Traefik Routing Architecture

M3TAL utilizes Traefik as its reverse proxy gateway, deployed via `routing-compose.yml`.

*   **Entry Points:** Traefik binds port 80 on the host as its primary HTTP entry point (and typically 443 for HTTPS if configured).
*   **Service Discovery:** It automatically discovers services by reading Docker labels on containers connected to the `proxy` network.
*   **Dynamic Configuration:** It loads additional routing rules from `/docker/dynamic/` (using a file provider), allowing for hot-reloads of configuration changes.
*   **API Routing:** An example of dynamic configuration is routing `api.DOMAIN` to the M3TAL API daemon (`http://host.docker.internal:8080`) via a file like `/docker/dynamic/api.yml`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels in `m3tal-compose.traefik.yml` route `dash.DOMAIN` to the `m3tal-dashboard` container.

---

## Port Map

| Port | Service                               | Access                                    |
|------|---------------------------------------|-------------------------------------------|
| 80   | Traefik HTTP entry point              | Public (when Traefik is active)           |
| 443  | Traefik HTTPS entry point             | Public (when Traefik & HTTPS configured)  |
| 8080 | M3TAL API daemon (Go)                 | Host-local only                           |
| 8081 | Traefik dashboard (admin interface)   | Host-local only (typically `127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard (Python/Flask)        | Direct port (`local` mode) or via Traefik (`traefik` mode) |

---

This documentation should provide a solid foundation for managing your M3TAL ecosystem. For further details on specific configurations or troubleshooting, consult the M3TAL project's official documentation and community resources.