```markdown
# M3TAL CLI Command Reference

Greetings, Operators of the M3TAL Ecosystem! DocSmith here, your M3TAL Ecosystem Documentation Architect. This document serves as your definitive cheat-sheet for interacting with the M3TAL CLI (`m3tal`), the unified Go binary that orchestrates your entire home server infrastructure.

From initial setup to day-to-day operations, the `m3tal` command is your gateway to managing your services, configurations, and the underlying Docker environment. Let's delve into the commands that keep your M3TAL ecosystem humming.

---

## 1. M3TAL Installation

For new installations, please follow these steps to add the M3TAL APT repository and install the `m3tal` CLI and its core components.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## 2. Core M3TAL Commands

These commands provide fundamental interactions with the M3TAL ecosystem, from launching the UI to checking system health.

### `m3tal` (or `m3tal ui`)
Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a graphical interface is available. It interacts with the local M3TAL API daemon.

**Usage Example:**
```bash
m3tal
```

### `m3tal headless [port]`
Starts the M3TAL API daemon in headless mode. This is the recommended way to run M3TAL on servers (e.g., Raspberry Pi, NAS) where a GUI is not desired. The API daemon manages Docker, the state database (`/var/lib/m3tal/state.db`), and provides API routes for the dashboard and CLI. If no port is specified, it defaults to 8080.

**Usage Examples:**
```bash
m3tal headless                  # Starts the API daemon on port 8080
m3tal headless 8081             # Starts the API daemon on port 8081
```

### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. This provides a quick overview of your server's health and resource utilization.

**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`
Performs a pre-flight health check of your M3TAL environment. This diagnostic tool verifies essential components such as Docker connectivity, the validity of your primary configuration file (`/etc/m3tal/.env`), and the availability of critical ports. Run this if you encounter issues after setup.

**Usage Example:**
```bash
m3tal doctor
```

---

## 3. Configuration Management

The `m3tal config` sub-commands are used to manage the primary configuration file, `/etc/m3tal/.env`. This file dictates how M3TAL and its services operate.

### `m3tal config wizard`
Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the easiest way to set up M3TAL for the first time or adjust major settings.

**Usage Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`
Sets a single environment variable within `/etc/m3tal/.env`. This is useful for making quick, specific changes without going through the entire wizard. Remember to restart relevant services (like `m3tal-api` or `m3tal dash restart`) after making changes.

**Usage Example:**
```bash
m3tal config set DOMAIN "myhomeserver.net"
```

### `m3tal config get KEY`
Reads and prints the value of a specific environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config get PUID
```

### `m3tal config scan`
Lists all known environment variables across all M3TAL stacks, including their default values. This helps you understand what variables are available for configuration.

**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`
Displays the current contents of your `/etc/m3tal/.env` file.

**Usage Example:**
```bash
m3tal config list
```

---

## 4. Dashboard Management

The `m3tal dash` sub-commands are dedicated to managing the M3TAL Dashboard container, which provides a web interface for your ecosystem. The dashboard's accessibility depends on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

### `m3tal dashpass [username] [password]`
Updates the password for a dashboard user. If `username` and `password` are omitted, the command becomes interactive, prompting you for the details. This manages the `/docker/users.json` credential store.

**Usage Examples:**
```bash
m3tal dashpass admin newSecurePassword123  # Set password for 'admin' user
m3tal dashpass                            # Interactive prompt to update password
```

### `m3tal dash up`
Pulls the latest dashboard compose configuration from GitHub (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) and then starts the `m3tal-dashboard` container. This command intelligently selects the correct override file based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

**Dashboard Access Modes (Controlled by `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`):**

*   **Mode 1: `local` (Default)**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
    *   **Override File:** Uses `m3tal-compose.local.yml`
    *   **Functionality:** Adds a direct port binding, exposing the dashboard container's port 8082 to the host on `${DASHBOARD_PORT:-8082}`. Traefik is not required for access.
    *   **Access URL:** `http://HOST_IP:8082` or `http://localhost:8082`
    *   **Best For:** LAN-only setups, first-time users, local testing.
*   **Mode 2: `traefik`**
    *   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
    *   **Override File:** Uses `m3tal-compose.traefik.yml`
    *   **Functionality:** Adds Traefik labels to the dashboard container, allowing Traefik to route traffic from `dash.${DOMAIN}` to the dashboard on port 8082. `m3tal up` must be run to start Traefik.
    *   **Access URL:** `http://dash.yourdomain.com` (e.g., `http://dash.myhomeserver.net`)
    *   **Best For:** Domain-based setups, multiple services behind a reverse proxy.

**Usage Example:**
```bash
m3tal dash up
```

### `m3tal dash down`
Stops the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`
Restarts the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`
Streams the logs from the `m3tal-dashboard` container to your terminal. Useful for debugging dashboard-related issues.

**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`
Shows the current running status of the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash status
```

---

## 5. M3TAL Stack Management

These commands manage all your Docker Compose stacks defined in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).

### `m3tal up`
Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command starts all your defined services (including the Traefik gateway if `routing-compose.yml` is present). New stacks are brought online by placing their compose files in `/docker/` and then running `m3tal up`.

**Usage Example:**
```bash
m3tal up
```

### `m3tal down`
Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This stops and removes all containers, networks, and volumes defined by your M3TAL stacks.

**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`
Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of all service outputs, invaluable for system-wide debugging.

**Usage Example:**
```bash
m3tal logs
```

---

## 6. Systemd Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage this service directly using `systemctl` and view its logs with `journalctl`.

**Check Status:**
```bash
systemctl status m3tal-api
```

**Restart the API Daemon:**
```bash
sudo systemctl restart m3tal-api
```

**View API Daemon Logs (Follow Live):**
```bash
sudo journalctl -u m3tal-api -f
```

---

## 7. Advanced Docker / Compose Operations (Fallback)

M3TAL uses **Docker Engine** and **Docker Compose V2** under the hood. While `m3tal up`/`down` simplifies stack management, you can always directly interact with Docker Compose for granular control or debugging. The canonical stack directory is `/opt/m3tal/stack/`, which is symlinked to `/docker/` for user convenience.

**Start a specific stack:**
```bash
# Navigate to the stacks directory
cd /docker/

# Start only the routing stack (Traefik, Cloudflared)
docker compose -f routing-compose.yml up -d
```

**Stop a specific stack:**
```bash
# Stop only the M3TAL dashboard (if not using `m3tal dash down`)
cd /docker/
docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml down
```

**View logs for a specific container:**
```bash
docker logs -f m3tal-dashboard
```

**Inspect a container:**
```bash
docker inspect m3tal-dashboard
```

---

## 8. M3TAL System Architecture & Concepts

Understanding the underlying architecture is key to mastering M3TAL.

### Filesystem Contract

M3TAL adheres to a strict filesystem contract to ensure stability and predictability.

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | **Primary configuration file.** Managed by `m3tal config wizard` and `m3tal config set/get`. |
| `/var/lib/m3tal/state.db` | **SQLite state database.** Auto-created by the API daemon (`m3tal-api.service`). Stores internal M3TAL state. |
| `/opt/m3tal/stack/` | **Canonical stack directory.** Contains core M3TAL compose files (`m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration. |
| `/docker` | **Symlink to `/opt/m3tal/stack/`.** This is the user-facing path where you should place your custom `*-compose.yml` files for new stacks. |
| `/docker/users.json` | **Dashboard credential store.** Managed by `m3tal dashpass`. Contains hashed user credentials for the M3TAL Dashboard. |

### Docker / Compose Runtime Explained

*   **Dependencies:** M3TAL requires **Docker Engine** and **Docker Compose V2** to be installed and running. These are hard dependencies.
*   **`m3tal up` operation:** This command iterates through all files ending in `*-compose.yml` within the `/docker/` directory and executes `docker compose up -d` for each. This ensures all your desired services are brought online.
*   **`m3tal dash up` operation:** This command has a specialized workflow:
    1.  It first downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files from GitHub into `/docker/`.
    2.  It then reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Finally, it starts the `m3tal-dashboard` container using `docker compose up -d -f m3tal-compose.yml -f m3tal-compose.<mode>.yml`, applying the appropriate override file (`local` or `traefik`).
*   **User Stacks:** To add new services or applications, simply create your `my-app-compose.yml` file in `/docker/` and then run `m3tal up`. M3TAL will discover and manage it.

### Deployment Lifecycle — Day 2 Operations (Installing a New Stack)

1.  **Place your compose file:** Create or copy your Docker Compose file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
2.  **Configure Environment Variables:** Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value`.
3.  **Start all stacks:** Run `m3tal up` to bring your new stack, along with all other existing M3TAL services, online.

### Traefik Routing Architecture

Traefik acts as the reverse proxy (API Gateway) for your M3TAL ecosystem, handling incoming HTTP traffic and routing it to the correct services.

*   **Deployment:** Traefik is deployed as a Docker container via `routing-compose.yml`.
*   **Entry Points:** It binds to port 80 on the host as its primary HTTP entry point (and 443 for HTTPS if configured).
*   **Service Discovery:** Traefik automatically discovers services by reading Docker labels on running containers (e.g., `m3tal-dashboard` when `DASHBOARD_EXPOSE_MODE=traefik`).
*   **Dynamic Configuration:** Traefik loads additional routing rules from `/docker/dynamic/` using a file provider. These configurations are hot-reloaded without restarting Traefik.
    *   **Example: API Routing:** Traefik routes requests for `api.${DOMAIN}` directly to the M3TAL API daemon running on the host at `http://host.docker.internal:8080` (defined in `/docker/dynamic/api.yml`).
    *   **Example: Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.${DOMAIN}` to the `m3tal-dashboard` container via specific Docker labels embedded in `m3tal-compose.traefik.yml`.

### M3TAL Port Map

Understanding which services listen on which ports is crucial for network configuration and troubleshooting.

| Port | Service | Access |
|------|---------|--------|
| 80 | Traefik HTTP entry point | Public (if `routing-compose.yml` is running) |
| 8080 | M3TAL API daemon (Go) | Host-local (accessed by dashboard and CLI) |
| 8081 | Traefik dashboard | Host-local only (for Traefik's own management UI) |
| 8082 | M3TAL Dashboard | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

---

This completes your comprehensive command reference for the M3TAL ecosystem. Should you have further queries or require deeper dives into specific components, consult the broader M3TAL documentation.

Happy orchestrating!
— DocSmith, M3TAL Ecosystem Documentation Architect
```