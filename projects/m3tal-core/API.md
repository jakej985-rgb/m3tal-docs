```markdown
# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for all M3TAL command-line interface (CLI) commands.

## Getting Started

To install M3TAL, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core M3TAL Commands

These commands are the primary interface for interacting with the M3TAL ecosystem.

### Launching M3TAL

*   **`m3tal`** or **`m3tal ui`**
    *   **Description:** Launches the M3TAL Linux Desktop Application. This provides a graphical user interface for managing your M3TAL services.
    *   **Example:**
        ```bash
        m3tal ui
        ```

*   **`m3tal headless [port]`**
    *   **Description:** Starts M3TAL in headless mode, ideal for home servers or environments without a graphical interface. You can optionally specify a port for communication.
    *   **Example (default port):**
        ```bash
        m3tal headless
        ```
    *   **Example (custom port):**
        ```bash
        m3tal headless 8080
        ```

### System Information & Health Checks

*   **`m3tal stats`**
    *   **Description:** Prints instant host system, GPU, thermal, and storage metrics to the terminal. Useful for quick health monitoring.
    *   **Example:**
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**
    *   **Description:** Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, `.env` file validity, and port availability.
    *   **Example:**
        ```bash
        m3tal doctor
        ```

## Configuration Management

Manage M3TAL's configuration settings via these commands. Configuration is primarily stored in `/etc/m3tal/.env`.

### Interactive Configuration

*   **`m3tal config wizard`**
    *   **Description:** An interactive wizard to guide you through configuring your `/etc/m3tal/.env` file. Recommended for initial setup.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

### Direct Configuration Access

*   **`m3tal config set KEY VALUE`**
    *   **Description:** Sets a single environment variable in the `.env` file.
    *   **Example:**
        ```bash
        m3tal config set LOG_LEVEL debug
        ```

*   **`m3tal config get KEY`**
    *   **Description:** Reads and prints the value of a single environment variable from the `.env` file.
    *   **Example:**
        ```bash
        m3tal config get DOMAIN
        ```

*   **`m3tal config scan`**
    *   **Description:** Lists all environment variables across all managed stacks, including their current values and defaults if applicable.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Description:** Lists the current contents of the `.env` file.
    *   **Example:**
        ```bash
        m3tal config list
        ```

## Dashboard Management

Commands specifically for managing the M3TAL dashboard interface.

*   **`m3tal dashpass [username] [password]`**
    *   **Description:** Updates the dashboard user password. If username and password are not provided, the command will prompt you interactively. The credentials are stored in `/docker/users.json`.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Example (with credentials):**
        ```bash
        m3tal dashpass m3taluser newsecurepassword
        ```

*   **`m3tal dash up`**
    *   **Description:** Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you have the most up-to-date dashboard setup.
    *   **Example:**
        ```bash
        m3tal dash up
        ```

*   **`m3tal dash down`**
    *   **Description:** Stops and removes the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash down
        ```

*   **`m3tal dash restart`**
    *   **Description:** Restarts the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash restart
        ```

*   **`m3tal dash logs`**
    *   **Description:** Streams the logs from the dashboard container in real-time. Useful for debugging.
    *   **Example:**
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**
    *   **Description:** Shows the current status of the dashboard container (e.g., running, exited).
    *   **Example:**
        ```bash
        m3tal dash status
        ```

## Stack Management

These commands manage your M3TAL application stacks, which are defined by Docker Compose files in `/docker/`.

*   **`m3tal up`**
    *   **Description:** Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This starts all your defined M3TAL services and any custom stacks you've added.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Description:** Runs `docker compose down` across all managed stacks, stopping and removing the containers defined in your compose files.
    *   **Example:**
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**
    *   **Description:** Streams aggregated logs from all currently running M3TAL stacks. This provides a unified view of your service logs.
    *   **Example:**
        ```bash
        m3tal logs
        ```

## Systemd Service Management

M3TAL's API daemon runs as a systemd service for robust background operation.

*   **`systemctl status m3tal-api`**
    *   **Description:** Checks the status of the `m3tal-api.service`.
    *   **Example:**
        ```bash
        systemctl status m3tal-api
        ```

*   **`systemctl restart m3tal-api`**
    *   **Description:** Restarts the `m3tal-api.service`.
    *   **Example:**
        ```bash
        systemctl restart m3tal-api
        ```

*   **`journalctl -u m3tal-api -f`**
    *   **Description:** Streams the logs for the `m3tal-api.service` in real-time, similar to `m3tal dash logs` but specifically for the API daemon.
    *   **Example:**
        ```bash
        journalctl -u m3tal-api -f
        ```

## Direct Docker Compose Commands (Fallback)

While the `m3tal` CLI abstracts these, you can always interact with Docker Compose directly if needed. M3TAL utilizes Docker Engine and Docker Compose V2.

*   **Starting all services:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml -f /docker/m3tal-compose.traefik.yml -f /docker/routing-compose.yml up -d
    ```
    *(Note: The specific files used may vary based on your `DASHBOARD_EXPOSE_MODE` and installed stacks. `m3tal up` handles this automatically.)*

*   **Stopping all services:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml -f /docker/m3tal-compose.traefik.yml -f /docker/routing-compose.yml down
    ```
    *(Note: `m3tal down` handles this automatically.)*

*   **Starting the dashboard specifically:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
    ```
    or
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d
    ```
    *(Note: `m3tal dash up` handles this logic based on `DASHBOARD_EXPOSE_MODE`.)*

*   **Stopping the dashboard specifically:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down
    ```
    or
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml down
    ```
    *(Note: `m3tal dash down` handles this.)*

*   **Viewing dashboard logs:**
    ```bash
    docker logs -f m3tal-dashboard
    ```
    *(Note: `m3tal dash logs` provides this functionality.)*

## Filesystem Contract

Understanding the M3TAL filesystem layout is crucial for advanced usage and troubleshooting.

| Path                        | Purpose                                               |
| :-------------------------- | :---------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                   | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`. |

## Dashboard Access Modes

The dashboard has two access modes, controlled by `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`:

### Mode 1: `local` (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses override: `m3tal-compose.local.yml`
*   Adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`
*   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`
*   No Traefik required for direct dashboard access. Works out of the box on a home server.
*   **Best for:** LAN-only setups, first-time users, local testing.

### Mode 2: `traefik`

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses override: `m3tal-compose.traefik.yml`
*   Adds Traefik labels so Traefik routes `dash.${DOMAIN}` → dashboard on port 8082.
*   **Access via:** `http://dash.DOMAIN` (Traefik must be running via `m3tal up`)
*   **Best for:** Domain-based setups, multiple services behind a reverse proxy.

## Port Map

| Port | Service                 | Access                                      |
| :--- | :---------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point| Public (traefik mode)                       |
| 8080 | M3TAL API daemon (Go)   | Host-local                                  |
| 8081 | Traefik dashboard       | Host-local only                             |
| 8082 | M3TAL Dashboard         | Direct port (local mode) or via Traefik (traefik mode) |
```