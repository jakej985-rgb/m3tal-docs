# M3TAL CLI Command Reference

Greetings, Operative. DocSmith here, your M3TAL Ecosystem Documentation Architect. This document serves as your essential guide to the `m3tal` Command Line Interface (CLI), your primary interface for interacting with and managing the entire M3TAL ecosystem. From system health checks to orchestrating your containerized applications, `m3tal` is your unified entry point.

## 1. M3TAL Ecosystem Overview

Before diving into commands, let's establish the fundamental architecture of M3TAL. Understanding these core components will empower you to leverage the CLI effectively.

### Components

*   **CLI Binary (`/usr/bin/m3tal`)**: A single Go binary, installed via APT, serving as the central control point for all M3TAL operations.
*   **API Daemon (`m3tal-api.service`)**: A Go binary running as a systemd service, listening on port 8080. It's the brains of the operation, managing Docker interactions, persistent state in a SQLite database, and exposing API routes.
*   **Dashboard Container (`m3tal-dashboard`)**: A Python/Flask application running inside Docker on internal port 8082. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`)**: A Docker container acting as a reverse proxy, exposing M3TAL services (and your custom stacks) by domain name on host port 80. It uses a file provider for dynamic routing based on Docker labels and configuration files.
*   **Cloudflared (`routing-compose.yml`)**: An *optional* Cloudflare tunnel container, enabling zero-configuration, secure internet access to your M3TAL instance without port forwarding.

### Filesystem Contract

M3TAL adheres to a strict filesystem layout to ensure predictable operation and easy management.

| Path                     | Purpose                                                                        |
| :----------------------- | :----------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard` and `m3tal config`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the API daemon.             |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal stack files and Traefik configuration.|
| `/docker`                | **Symlink** to `/opt/m3tal/stack/`. This is the **user-facing path** for all Docker Compose stack operations. Place your custom `*-compose.yml` files here. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.                       |

### Docker / Compose Runtime Explained

M3TAL is built upon **Docker Engine** and **Docker Compose V2**. These are hard dependencies for the system's core functionality.

*   The `m3tal up` command orchestrates all `*-compose.yml` files found within the `/docker/` directory, treating them as a single, cohesive Docker Compose project.
*   The `m3tal dash up` command performs a specialized operation for the M3TAL Dashboard:
    1.  It pulls the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` definitions directly from GitHub.
    2.  It then consults the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env` to apply the appropriate Compose override file (`local` or `traefik`) before starting the dashboard container.

### Dashboard Access — Two Modes

Accessing the M3TAL Dashboard is critical, and the system supports two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

#### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode utilizes the `m3tal-compose.local.yml` override, which directly binds the dashboard container's internal port 8082 to the host system's `DASHBOARD_PORT` (defaulting to 8082).
*   **Access**: Navigate to `http://HOST_IP:8082` or `http://localhost:8082` in your web browser.
*   **Requirements**: No Traefik configuration is needed. This mode works immediately upon `m3tal dash up`.
*   **Best for**: Local Area Network (LAN) only setups, initial deployments, first-time users, and local testing.

#### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode utilizes the `m3tal-compose.traefik.yml` override, adding specific Traefik labels to the dashboard container. These labels instruct Traefik to route incoming HTTP requests for `dash.${DOMAIN}` to the dashboard container on its internal port 8082.
*   **Access**: Navigate to `http://dash.YOUR_DOMAIN` (e.g., `http://dash.m3tal.local`). For this to work, Traefik **must be running** (via `m3tal up`).
*   **Requirements**: Requires Traefik to be deployed and configured (which `m3tal up` handles with `routing-compose.yml`). You'll need `DOMAIN` correctly set in `/etc/m3tal/.env`.
*   **Best for**: Domain-based access, systems with multiple services behind a reverse proxy, and advanced deployments.

### Traefik Routing Architecture

Traefik is a pivotal component, deployed as a container via `routing-compose.yml`.

*   **Entry Point**: Binds host port 80 (HTTP) to serve as the primary entry point for all web traffic.
*   **Service Discovery**: Automatically discovers services by inspecting Docker container labels, making integration seamless.
*   **Dynamic Configuration**: Loads additional routing rules from `/etc/traefik/dynamic` (symlinked from `/docker/dynamic`), supporting hot-reloading for instant updates.
*   **API Routing**: Routes `api.${DOMAIN}` to the M3TAL API daemon (running on the host at `http://host.docker.internal:8080`) using a file provider definition (`/docker/dynamic/api.yml`).
*   **Dashboard Routing**: Routes `dash.${DOMAIN}` to the M3TAL Dashboard container using Traefik labels (only when `DASHBOARD_EXPOSE_MODE=traefik`).

### Port Map

| Port | Service                    | Access                                        |
| :--- | :------------------------- | :-------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public (when Traefik mode is active)          |
| 8080 | M3TAL API daemon (Go)      | Host-local only                               |
| 8081 | Traefik dashboard          | Host-local only (e.g., `http://localhost:8081`) |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (Traefik mode) |

## 2. APT Installation

To get M3TAL up and running on your Debian-based system, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. CLI Command Reference

Here's your comprehensive guide to the `m3tal` CLI commands.

---

### `m3tal` (or `m3tal ui`)

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment to bring up the graphical interface.

**Usage Example:**

```bash
m3tal
# Or explicitly:
m3tal ui
```

---

### `m3tal headless [port]`

Starts the M3TAL API daemon in headless mode, typically on port 8080. This is the ideal command for home servers or environments where a graphical interface isn't needed. If a port is not specified, it defaults to 8080.

**Usage Examples:**

```bash
# Start the API daemon on its default port 8080
sudo m3tal headless

# Start the API daemon on a custom port, e.g., 9000
sudo m3tal headless 9000
```

---

### `m3tal stats`

Provides an instant snapshot of your host system's vitals, including CPU, RAM, GPU, thermal sensors, and storage metrics, directly to your terminal.

**Usage Example:**

```bash
m3tal stats
```

---

### `m3tal doctor`

Performs a crucial pre-flight health check of your M3TAL environment. This includes verifying Docker connectivity, validating the contents of `/etc/m3tal/.env`, and checking for port availability. Essential for troubleshooting.

**Usage Example:**

```bash
m3tal doctor
```

---

### `m3tal config wizard`

Launches an interactive, step-by-step wizard to guide you through configuring or updating the `/etc/m3tal/.env` file. This is the recommended method for initial setup or major configuration changes.

**Usage Example:**

```bash
sudo m3tal config wizard
```

---

### `m3tal config set KEY VALUE`

Sets a single environment variable within `/etc/m3tal/.env` to the specified value. Useful for quick, targeted adjustments.

**Usage Example:**

```bash
# Change the dashboard exposure mode to Traefik
sudo m3tal config set DASHBOARD_EXPOSE_MODE traefik

# Set the primary domain for Traefik routing
sudo m3tal config set DOMAIN myhomelab.local
```

---

### `m3tal config get KEY`

Retrieves and prints the value of a specific environment variable from `/etc/m3tal/.env`.

**Usage Example:**

```bash
# Check the currently configured domain
m3tal config get DOMAIN

# See the dashboard's exposure mode
m3tal config get DASHBOARD_EXPOSE_MODE
```

---

### `m3tal config scan`

Scans all active Docker Compose files in `/docker/` and lists all environment variables referenced across those stacks, including their default values if applicable. Helps identify all potential configuration points.

**Usage Example:**

```bash
m3tal config scan
```

---

### `m3tal config list`

Displays the current contents of your primary configuration file, `/etc/m3tal/.env`, directly in the terminal.

**Usage Example:**

```bash
m3tal config list
```

---

### `m3tal dashpass [username] [password]`

Manages user credentials for the M3TAL Dashboard, stored in `/docker/users.json`. If `username` and `password` are omitted, the command will launch an interactive prompt.

**Usage Examples:**

```bash
# Interactively set or update a user's password
m3tal dashpass

# Set the password for the 'admin' user directly
m3tal dashpass admin MySuperSecretPassword123!
```

---

### `m3tal dash up`

Pulls the latest M3TAL Dashboard Docker Compose configuration from GitHub and then starts the dashboard container, applying the correct override (`local` or `traefik`) based on `DASHBOARD_EXPOSE_MODE`.

**Usage Example:**

```bash
m3tal dash up
```

---

### `m3tal dash down`

Stops and removes the M3TAL Dashboard container.

**Usage Example:**

```bash
m3tal dash down
```

---

### `m3tal dash restart`

Restarts the M3TAL Dashboard container. Useful after configuration changes or for troubleshooting.

**Usage Example:**

```bash
m3tal dash restart
```

---

### `m3tal dash logs`

Streams the real-time logs from the M3TAL Dashboard container to your terminal. Press `Ctrl+C` to exit the stream.

**Usage Example:**

```bash
m3tal dash logs
```

---

### `m3tal dash status`

Displays the current operational status of the M3TAL Dashboard container (e.g., running, stopped, unhealthy).

**Usage Example:**

```bash
m3tal dash status
```

---

### `m3tal up`

Initiates the deployment and startup of *all* Docker Compose services defined by `*-compose.yml` files found within the `/docker/` directory. This includes Traefik, Cloudflared, and any custom stacks you've added.

**Usage Example:**

```bash
m3tal up
```

---

### `m3tal down`

Stops and removes all Docker Compose services defined by `*-compose.yml` files in `/docker/`. Use with caution as this will stop all your M3TAL-managed applications.

**Usage Example:**

```bash
m3tal down
```

---

### `m3tal logs`

Streams aggregated, real-time logs from *all* currently running Docker containers managed by M3TAL. This provides a holistic view of your ecosystem's activity. Press `Ctrl+C` to exit the stream.

**Usage Example:**

```bash
m3tal logs
```

## 4. Systemd Service Management

The M3TAL API daemon is managed as a systemd service, `m3tal-api.service`. You can interact with it directly using standard `systemctl` and `journalctl` commands.

*   **Check API Status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart API Daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stream API Logs (real-time):**
    ```bash
    journalctl -u m3tal-api -f
    ```

## 5. Direct Docker / Compose Fallback

The `m3tal` CLI provides a convenient abstraction layer over Docker Engine and Docker Compose V2. In scenarios where you need to perform more granular Docker operations or troubleshoot directly, you can bypass the `m3tal` CLI and use `docker` and `docker compose` commands directly.

Remember that `m3tal up` primarily executes `docker compose` operations within the `/docker/` directory.

**Common Fallback Operations:**

1.  **Navigate to the Stacks Directory:**
    ```bash
    cd /docker/
    ```

2.  **Start all M3TAL services in detached mode (equivalent to `m3tal up`):**
    ```bash
    docker compose -f m3tal-compose.yml -f routing-compose.yml -f your-custom-stack-compose.yml up -d
    # Include all relevant *-compose.yml files as needed.
    ```

3.  **Stop all M3TAL services (equivalent to `m3tal down`):**
    ```bash
    docker compose -f m3tal-compose.yml -f routing-compose.yml -f your-custom-stack-compose.yml down
    ```

4.  **View logs for a specific service (e.g., the dashboard):**
    ```bash
    docker compose logs -f m3tal-dashboard
    ```

5.  **Restart a specific service (e.g., Traefik):**
    ```bash
    docker compose restart traefik
    ```

6.  **Pull latest images for all services:**
    ```bash
    docker compose pull
    ```

## 6. Deployment Lifecycle: Day 2 Operations

Integrating new applications and services into your M3TAL ecosystem is straightforward.

**Installing a New Stack:**

1.  **Place Your Compose File:** Create or copy your Docker Compose file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
    ```bash
    sudo cp ~/my-app-compose.yml /docker/
    ```
2.  **Configure Environment Variables:** Ensure all necessary environment variables for your new stack are defined in `/etc/m3tal/.env`. You can use the interactive wizard or set them individually:
    ```bash
    sudo m3tal config wizard
    # OR
    sudo m3tal config set MY_APP_PORT 9090
    ```
3.  **Start All Stacks:** Run `m3tal up` to deploy your new service alongside the existing M3TAL components.
    ```bash
    m3tal up
    ```

This comprehensive reference should provide you with all the tools necessary to manage your M3TAL ecosystem efficiently. Should you require further assistance, consult the M3TAL knowledge base or engage with the community. DocSmith out.