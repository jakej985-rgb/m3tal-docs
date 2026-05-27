# M3TAL CLI Command Reference

Greetings, M3TAL Operative. I am DocSmith, your M3TAL Ecosystem Documentation Architect. This document serves as your complete cheat-sheet for interacting with the M3TAL system via its command-line interface. Master these commands, and you will master your infrastructure.

---

## M3TAL APT Installation

To install the M3TAL CLI and API daemon, execute the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## Filesystem Contract

The M3TAL ecosystem maintains a strict filesystem contract to ensure consistent operation and simplify management. Understanding these paths is critical:

| Path                        | Purpose                                                      |
| :-------------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`           | **Primary Configuration File.** All M3TAL environment variables are stored here. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | **SQLite State Database.** Stores internal M3TAL API daemon state. Auto-created. |
| `/opt/m3tal/stack/`         | **Canonical Stack Directory.** Contains core compose files and Traefik configuration files. |
| `/docker`                   | **Symlink to `/opt/m3tal/stack/`.** This is the **user-facing path** for all stack operations. Users should place their custom `*-compose.yml` files here. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.     |

---

## Core Commands

### `sudo m3tal`

Opens the interactive TUI (Terminal User Interface) Control Center. This provides a numbered menu for common operations and system status.

*   **Usage Example:**
    ```bash
    sudo m3tal
    ```

### `m3tal init`

Generates the initial `/etc/m3tal/.env` configuration file from defaults. This command should be run on the first install to prepare the system for configuration.

*   **Usage Example:**
    ```bash
    m3tal init
    ```

### `m3tal doctor`

Performs a pre-flight health check of the M3TAL system. It verifies Docker connectivity, validates the `/etc/m3tal/.env` file, and checks for necessary port availability (e.g., 80, 8080, 8082).

*   **Usage Example:**
    ```bash
    m3tal doctor
    ```

---

## Configuration Management

M3TAL uses `/etc/m3tal/.env` as its primary configuration file. These commands facilitate its management.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating variables in `/etc/m3tal/.env`. This is the recommended way to modify your M3TAL environment.

*   **Usage Example:**
    ```bash
    m3tal config wizard
    ```

### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`. This is useful for scripting or quick, targeted changes.

*   **Usage Example:**
    ```bash
    m3tal config set DOMAIN "my-metal-server.com"
    ```

### `m3tal config get KEY`

Reads and displays the value of a single environment variable from `/etc/m3tal/.env`.

*   **Usage Example:**
    ```bash
    m3tal config get DASHBOARD_EXPOSE_MODE
    ```

### `m3tal config scan`

Lists all known environment variables across all active Docker Compose stacks managed by M3TAL, including default values and currently set values. This provides a comprehensive overview of the system's potential and active configuration.

*   **Usage Example:**
    ```bash
    m3tal config scan
    ```

### `m3tal config list`

Displays the current contents of the `/etc/m3tal/.env` file.

*   **Usage Example:**
    ```bash
    m3tal config list
    ```

---

## Dashboard Management

The M3TAL Dashboard provides a web interface for controlling your M3TAL ecosystem. These commands specifically manage the dashboard container.

### Dashboard Access — TWO MODES (CRITICAL)

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

**Mode 1: `local` (Default)**
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** Adds a direct port binding to the dashboard container (`${DASHBOARD_PORT:-8082}:8082`). This mode does **not** require Traefik.
*   **Access Via:** `http://HOST_IP:8082` or `http://localhost:8082`
*   **Best For:** LAN-only setups, first-time users, local development, or environments without a public domain.

**Mode 2: `traefik`**
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** Adds Traefik labels to the dashboard container, allowing Traefik to route traffic from `dash.${DOMAIN}` to the dashboard on its internal port 8082. Traefik (started via `m3tal up`) *must* be running for this mode to work.
*   **Access Via:** `http://dash.DOMAIN` (e.g., `http://dash.my-metal-server.com`)
*   **Best For:** Domain-based setups, integration with other services behind a reverse proxy, or when `DOMAIN` is configured for external access.

### `m3tal dashpass [username] [password]`

Updates the password for a user on the M3TAL Dashboard. If `username` and `password` are omitted, the command will prompt for them interactively. This updates the `/docker/users.json` file.

*   **Usage Examples:**
    ```bash
    # Set password for 'admin' user non-interactively
    m3tal dashpass admin newSecureP@ssw0rd

    # Interactively set password (will prompt for username and password)
    m3tal dashpass
    ```

### `m3tal dash up`

Pulls the latest dashboard compose configuration files from GitHub, then starts the `m3tal-dashboard` container. It automatically selects the correct compose override (`local.yml` or `traefik.yml`) based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

*   **Usage Example:**
    ```bash
    m3tal dash up
    ```

### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

*   **Usage Example:**
    ```bash
    m3tal dash down
    ```

### `m3tal dash restart`

Restarts the `m3tal-dashboard` container.

*   **Usage Example:**
    ```bash
    m3tal dash restart
    ```

### `m3tal dash logs`

Streams logs from the `m3tal-dashboard` container, useful for debugging.

*   **Usage Example:**
    ```bash
    m3tal dash logs
    ```

### `m3tal dash status`

Shows the current operational status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

*   **Usage Example:**
    ```bash
    m3tal dash status
    ```

---

## Stack Management

M3TAL uses Docker Compose V2 to manage its core services (like Traefik, Cloudflared) and any user-defined applications. All compose files reside in the user-facing `/docker/` directory.

### `m3tal up`

Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all services defined in your M3TAL ecosystem, including Traefik and any custom stacks you've added.

*   **Usage Example:**
    ```bash
    m3tal up
    ```
*   **Note:** To deploy a new stack, simply place its `my-stack-compose.yml` file in `/docker/`, ensure required `.env` variables are set, then run `m3tal up`.

### `m3tal down`

Runs `docker compose down` across all `*-compose.yml` files in the `/docker/` directory. This command gracefully stops and removes all containers, networks, and volumes defined by your M3TAL stacks.

*   **Usage Example:**
    ```bash
    m3tal down
    ```

### `m3tal logs`

Streams aggregated logs from all currently running Docker Compose stacks managed by M3TAL. This provides a unified view of all service outputs.

*   **Usage Example:**
    ```bash
    m3tal logs
    ```

---

## Systemd Service Management

The M3TAL API daemon (`m3tal-api`) runs as a systemd service. These commands allow you to interact with it directly.

### `systemctl status m3tal-api`

Checks the current status of the M3TAL API daemon systemd service, including whether it's active, its PID, and recent log entries.

*   **Usage Example:**
    ```bash
    systemctl status m3tal-api
    ```

### `journalctl -u m3tal-api -f`

Streams real-time logs from the M3TAL API daemon. This is invaluable for debugging issues with the core M3TAL API.

*   **Usage Example:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Docker Fallback Commands (Advanced)

M3TAL leverages Docker Engine and Docker Compose V2 extensively. While the `m3tal` CLI provides a streamlined interface, you can always interact directly with Docker Compose for more granular control or troubleshooting.

**Important:** M3TAL's core compose files are located in `/opt/m3tal/stack/`, which is symlinked to `/docker/`. When running `docker compose` commands directly, you'll typically execute them from the `/docker/` directory or specify the compose files directly.

### General Stack Management

To manage all stacks defined in `/docker/` (including M3TAL's core routing stack and any custom stacks):

*   **Start all services:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack-compose.yml up -d
    # Alternatively, if you are in the /docker/ directory:
    # cd /docker/
    # sudo docker compose up -d
    ```
    *   **Note:** The `m3tal-compose.yml` file will automatically include the correct dashboard override (`local.yml` or `traefik.yml`) based on your `.env` configuration, if present. For manual `docker compose` calls specifically for the dashboard, see below.

*   **Stop all services:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack-compose.yml down
    # Alternatively, if you are in the /docker/ directory:
    # cd /docker/
    # sudo docker compose down
    ```

*   **View logs from all services:**
    ```bash
    sudo docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack-compose.yml logs -f
    # Alternatively, if you are in the /docker/ directory:
    # cd /docker/
    # sudo docker compose logs -f
    ```

### Dashboard Specific Management

To manually manage the `m3tal-dashboard` container using `docker compose`, you need to explicitly include the correct override file based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

*   **Start the dashboard in `local` mode (`DASHBOARD_EXPOSE_MODE=local`):**
    ```bash
    # Ensure you are in the /docker/ directory or specify the paths correctly
    cd /docker/
    sudo docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d m3tal-dashboard
    ```

*   **Start the dashboard in `traefik` mode (`DASHBOARD_EXPOSE_MODE=traefik`):**
    ```bash
    # Ensure you are in the /docker/ directory or specify the paths correctly
    cd /docker/
    sudo docker compose -f m3tal-compose.yml -f m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```

*   **Stop the dashboard:**
    ```bash
    cd /docker/
    sudo docker compose -f m3tal-compose.yml down m3tal-dashboard
    ```

*   **View dashboard logs:**
    ```bash
    cd /docker/
    sudo docker compose -f m3tal-compose.yml logs -f m3tal-dashboard
    ```

---

This command reference provides the essential tools to operate and manage your M3TAL ecosystem effectively. Keep it close, and may your M3TAL run smoothly.