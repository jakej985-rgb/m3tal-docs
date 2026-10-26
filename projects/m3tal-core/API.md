# M3TAL Ecosystem Documentation — CLI Command Reference

Welcome, Architect. This document serves as the comprehensive command-line interface (CLI) cheat-sheet for the M3TAL ecosystem, providing a complete reference for every `m3tal` command, core architectural insights, and critical operational procedures.

The M3TAL CLI, executed via the `/usr/bin/m3tal` binary, is your single entrypoint for managing your M3TAL server. It orchestrates the M3TAL API daemon, Docker containers, configuration files, and system services.

---

## 1. M3TAL CLI Commands

The `m3tal` command offers a robust set of tools for every aspect of M3TAL management.

### General Commands

**`m3tal` (or `m3tal ui`)**
Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where the full M3TAL UI is desired.
**Example:**
```bash
m3tal
```
_Alternatively, if you prefer explicit naming:_
```bash
m3tal ui
```

**`m3tal headless [port]`**
Starts M3TAL in headless mode, ideal for home servers or environments without a graphical user interface. By default, it runs on port `8080`, serving the M3TAL API.
**Example:**
_To start headless on the default port:_
```bash
m3tal headless
```
_To start headless on a custom port, e.g., 5050:_
```bash
m3tal headless 5050
```

**`m3tal stats`**
Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. Useful for quick health checks and resource monitoring.
**Example:**
```bash
m3tal stats
```

**`m3tal doctor`**
Runs a pre-flight health check on your M3TAL installation. This command verifies essential dependencies like Docker connectivity, the validity of your `/etc/m3tal/.env` configuration, and the availability of critical ports.
**Example:**
```bash
m3tal doctor
```

### Configuration Management (`m3tal config`)

The `m3tal config` subcommand manages the primary M3TAL configuration file located at `/etc/m3tal/.env`.

**`m3tal config wizard`**
Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended method for initial setup and major configuration changes.
**Example:**
```bash
m3tal config wizard
```

**`m3tal config set KEY VALUE`**
Sets a single environment variable within `/etc/m3tal/.env`. This is useful for making quick, specific changes without running the full wizard.
**Example:**
_To set your primary domain for Traefik routing:_
```bash
m3tal config set DOMAIN myhomelab.com
```
_To switch your dashboard to Traefik mode:_
```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

**`m3tal config get KEY`**
Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.
**Example:**
_To check your configured time zone:_
```bash
m3tal config get TZ
```

**`m3tal config scan`**
Lists all detected environment variables, including their default values and overrides, across all M3TAL stacks. This provides a comprehensive overview of your environment.
**Example:**
```bash
m3tal config scan
```

**`m3tal config list`**
Prints the current contents of your `/etc/m3tal/.env` file to the terminal.
**Example:**
```bash
m3tal config list
```

### Dashboard Management (`m3tal dash`)

The `m3tal dash` subcommand specifically manages the `m3tal-dashboard` container. It ensures you're running the latest dashboard version and handles user credentials.

**`m3tal dashpass [username] [password]`**
Updates the password for a dashboard user. If `username` and `password` are omitted, it will launch an interactive prompt. The credentials are stored in `/docker/users.json`.
**Example:**
_To interactively set/update the 'admin' user password:_
```bash
m3tal dashpass
```
_To directly set the 'admin' user password (replacing `newstrongpassword`):_
```bash
m3tal dashpass admin newstrongpassword
```

**`m3tal dash up`**
Pulls the latest dashboard compose configuration (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub, then starts the `m3tal-dashboard` container using the appropriate override based on `DASHBOARD_EXPOSE_MODE`.
**Example:**
```bash
m3tal dash up
```

**`m3tal dash down`**
Stops and removes the `m3tal-dashboard` container.
**Example:**
```bash
m3tal dash down
```

**`m3tal dash restart`**
Restarts the `m3tal-dashboard` container.
**Example:**
```bash
m3tal dash restart
```

**`m3tal dash logs`**
Streams real-time logs from the `m3tal-dashboard` container. Useful for debugging dashboard-related issues.
**Example:**
```bash
m3tal dash logs
```

**`m3tal dash status`**
Shows the current status of the `m3tal-dashboard` container (e.g., `running`, `exited`).
**Example:**
```bash
m3tal dash status
```

### Docker Stack Management (`m3tal up`, `m3tal down`, `m3tal logs`)

These commands manage all Docker Compose stacks defined by `*-compose.yml` files within `/docker/`.

**`m3tal up`**
Runs `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This command brings up all your configured services, including Traefik, your custom applications, and optionally the dashboard if not managed separately.
**Example:**
```bash
m3tal up
```

**`m3tal down`**
Runs `docker compose down` across all `*-compose.yml` files in `/docker/`. This command gracefully stops and removes all containers, networks, and volumes defined by your M3TAL stacks.
**Example:**
```bash
m3tal down
```

**`m3tal logs`**
Streams aggregated logs from all currently running Docker containers managed by M3TAL. This provides a consolidated view of all service output.
**Example:**
```bash
m3tal logs
```

---

## 2. M3TAL Filesystem Contract

Understanding the M3TAL filesystem layout is crucial for advanced configuration and troubleshooting.

| Path                         | Purpose                                                              |
|------------------------------|----------------------------------------------------------------------|
| `/etc/m3tal/.env`            | **Primary configuration file.** Managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db`    | **SQLite state database.** Auto-created by the M3TAL API daemon.     |
| `/opt/m3tal/stack/`          | **Canonical stack directory.** Contains core compose files and Traefik config. |
| `/docker`                    | **User-facing stack directory.** This is a symlink to `/opt/m3tal/stack/`. All user-defined `*-compose.yml` files should be placed here. |
| `/docker/users.json`         | **Dashboard credential store.** Managed by `m3tal dashpass`.         |

---

## 3. M3TAL Core Architecture

M3TAL is built on a robust, modular architecture designed for stability and extensibility.

### 3.1. CLI Binary (`/usr/bin/m3tal`)
The M3TAL CLI is a unified Go binary installed via APT, serving as the central command dispatcher for all operations.

### 3.2. API Daemon (`m3tal-api.service`)
This is a Go binary running as a systemd service, exposed on host port `8080`. It acts as the backend for the M3TAL UI and interacts with Docker Engine, manages the `/var/lib/m3tal/state.db` SQLite database, and serves API routes.
*   **Access:** Primarily accessed by other M3TAL components (e.g., `m3tal-dashboard`) at `http://host.docker.internal:8080` or `http://localhost:8080`. Traefik can also route to it via `api.DOMAIN`.

### 3.3. Dashboard Container (`m3tal-dashboard`)
A Python/Flask container that provides a web-based interface for M3TAL. It runs internally on port `8082` and communicates with the M3TAL API daemon at `http://host.docker.internal:8080`. Its exposure to the network is controlled by `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.

### 3.4. Traefik Gateway & Cloudflared
*   **Traefik (`routing-compose.yml`):** A Docker container that acts as a reverse proxy, typically binding host port `80`. It automatically discovers services via Docker labels and loads dynamic routing configurations from `/docker/dynamic/`. It's responsible for routing `api.DOMAIN` to the API daemon and `dash.DOMAIN` to the dashboard (when `DASHBOARD_EXPOSE_MODE=traefik`).
*   **Cloudflared (`routing-compose.yml`):** An optional Cloudflare tunnel container, also deployed via `routing-compose.yml`, which provides zero-configuration internet access for your M3TAL services without opening ports on your router.

### 3.5. Docker / Compose Runtime Explained
M3TAL leverages **Docker Engine** and **Docker Compose V2** as hard dependencies.
*   The `m3tal up` command executes `docker compose up -d` for all `*-compose.yml` files found in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`).
*   The `m3tal dash up` command is specialized:
    1.  It downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` files from GitHub.
    2.  It then starts the `m3tal-dashboard` container, dynamically applying the correct compose override file based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.
*   Users can extend M3TAL functionality by placing their own `*-compose.yml` files directly into `/docker/`.

---

## 4. M3TAL Dashboard Access Modes

The M3TAL dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### 4.1. Mode 1: `local` (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port (`8082`) to a host port (default `8082`). No Traefik required.
*   **Access URL:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Best for:** LAN-only setups, first-time users, local testing, and environments where a reverse proxy isn't desired or available. This mode works out-of-the-box on a home server.

**Example Access (assuming default port and local IP):**
```
http://192.168.1.100:8082
```

### 4.2. Mode 2: `traefik`
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard container. Traefik (which must be running via `m3tal up`) then automatically routes traffic from a specific domain/subdomain (`dash.${DOMAIN}`) to the dashboard's internal port `8082`.
*   **Access URL:** `http://dash.YOUR_DOMAIN` (e.g., `http://dash.myhomelab.com`).
*   **Best for:** Domain-based setups, environments with multiple services behind a reverse proxy, and integration with dynamic DNS or Cloudflare tunnels.

**Example Access (assuming DOMAIN=myhomelab.com and Traefik is running):**
```
http://dash.myhomelab.com
```

---

## 5. Day 2 Operations: Installing New Stacks

Adding a new application or service to your M3TAL ecosystem is straightforward:

1.  **Place your compose file:** Create or place your Docker Compose file (e.g., `my-app-compose.yml`) directly into the `/docker/` directory.
    ```bash
    sudo nano /docker/my-app-compose.yml
    ```
2.  **Configure Environment Variables:** Ensure all necessary environment variables for your new stack are defined in `/etc/m3tal/.env`. You can use the interactive wizard or set them individually.
    ```bash
    m3tal config wizard
    # OR
    m3tal config set MY_APP_VOLUME_PATH /mnt/data/myapp
    ```
3.  **Start All Stacks:** Run `m3tal up` to deploy your new stack along with all other existing M3TAL services.
    ```bash
    m3tal up
    ```

---

## 6. Advanced Topics

### 6.1. Systemd Service Management

The M3TAL API daemon (`m3tal-api`) runs as a systemd service. You can manage it using standard `systemctl` commands.

*   **Check Status:** See if the API is running and view recent logs.
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart Service:** Apply configuration changes or resolve issues by restarting the API daemon.
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stop Service:** Halt the API daemon.
    ```bash
    sudo systemctl stop m3tal-api
    ```
*   **Start Service:** Begin running the API daemon.
    ```bash
    sudo systemctl start m3tal-api
    ```
*   **Stream Logs:** View real-time logs from the API daemon.
    ```bash
    journalctl -u m3tal-api -f
    ```

### 6.2. Direct Docker Compose Fallback

While `m3tal up` and `m3tal down` are the recommended ways to manage your Docker stacks, you can always use `docker compose` directly as a fallback or for more granular control.

**Prerequisites:**
*   Navigate to the `/docker/` directory.
*   Specify the compose files you want to manage using `-f`.

**Examples:**

*   **Start all services:**
    ```bash
    cd /docker/
    sudo docker compose -f m3tal-compose.yml -f routing-compose.yml -f my-stack-compose.yml up -d
    ```
*   **Stop only the M3TAL dashboard:**
    ```bash
    cd /docker/
    sudo docker compose -f m3tal-compose.yml down m3tal-dashboard
    ```
*   **View logs for Traefik:**
    ```bash
    cd /docker/
    sudo docker compose -f routing-compose.yml logs -f traefik
    ```
*   **Pull latest images for all services in a specific stack:**
    ```bash
    cd /docker/
    sudo docker compose -f routing-compose.yml pull
    ```

### 6.3. Port Map

| Port | Service                               | Access                                    |
|------|---------------------------------------|-------------------------------------------|
| 80   | Traefik HTTP entry point              | Public (when `DASHBOARD_EXPOSE_MODE=traefik` or other services use Traefik) |
| 8080 | M3TAL API daemon (Go)                 | Host-local (internal, or via Traefik `api.DOMAIN`) |
| 8081 | Traefik dashboard                     | Host-local only (e.g., `http://127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard                       | Direct port (`local` mode) or via Traefik (`traefik` mode) |

---

## 7. M3TAL Installation

To install the M3TAL CLI and core components, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```