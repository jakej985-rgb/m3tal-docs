# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for all available M3TAL CLI commands.

## System Management

### Launching M3TAL
*   **`m3tal`**
    Launches the M3TAL Linux Desktop Application.
    ```bash
    m3tal
    ```
*   **`m3tal ui`**
    Launches the M3TAL Linux Desktop Application. This is an alias for `m3tal`.
    ```bash
    m3tal ui
    ```

### Headless Mode
*   **`m3tal headless [port]`**
    Starts M3TAL in headless mode, ideal for home servers. You can optionally specify a port.
    ```bash
    m3tal headless 8080
    ```

### System Information & Diagnostics
*   **`m3tal stats`**
    Prints instant host system, GPU, thermal, and storage metrics to the terminal.
    ```bash
    m3tal stats
    ```
*   **`m3tal doctor`**
    Performs a pre-flight health check, verifying Docker connectivity, `.env` validity, and port availability.
    ```bash
    m3tal doctor
    ```

## Configuration Management

### `.env` File Management
*   **`m3tal config wizard`**
    Launches an interactive wizard to configure the `/etc/m3tal/.env` file.
    ```bash
    m3tal config wizard
    ```
*   **`m3tal config set KEY VALUE`**
    Sets a single environment variable in the `.env` file.
    ```bash
    m3tal config set API_PORT 5050
    ```
*   **`m3tal config get KEY`**
    Reads a single environment variable from the `.env` file.
    ```bash
    m3tal config get DOMAIN
    ```
*   **`m3tal config scan`**
    Lists all environment variables across all active stacks, including their current values and defaults.
    ```bash
    m3tal config scan
    ```
*   **`m3tal config list`**
    Lists the current contents of the `.env` file.
    ```bash
    m3tal config list
    ```

## Dashboard Management

### Dashboard Authentication
*   **`m3tal dashpass [username] [password]`**
    Updates the dashboard user password. If username and password are not provided, it will prompt interactively.
    ```bash
    m3tal dashpass admin new_secure_password
    ```

### Dashboard Container Control
*   **`m3tal dash up`**
    Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.
    ```bash
    m3tal dash up
    ```
*   **`m3tal dash down`**
    Stops the dashboard container.
    ```bash
    m3tal dash down
    ```
*   **`m3tal dash restart`**
    Restarts the dashboard container.
    ```bash
    m3tal dash restart
    ```
*   **`m3tal dash logs`**
    Streams the dashboard container logs.
    ```bash
    m3tal dash logs
    ```
*   **`m3tal dash status`**
    Shows the current status of the dashboard container.
    ```bash
    m3tal dash status
    ```

## Stack Management

### General Stack Control
*   **`m3tal up`**
    Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all configured stacks.
    ```bash
    m3tal up
    ```
*   **`m3tal down`**
    Runs `docker compose down` across all configured stacks.
    ```bash
    m3tal down
    ```
*   **`m3tal logs`**
    Streams aggregated logs from all running M3TAL stacks.
    ```bash
    m3tal logs
    ```

## Systemd Service Management

The M3TAL API daemon is managed by `systemd`.

*   **`systemctl status m3tal-api`**
    Checks the status of the M3TAL API service.
    ```bash
    systemctl status m3tal-api
    ```
*   **`journalctl -u m3tal-api -f`**
    Streams the logs for the `m3tal-api.service` in real-time.
    ```bash
    journalctl -u m3tal-api -f
    ```

## Docker Compose Fallback

In situations where the `m3tal` CLI commands are insufficient or for direct control, you can use Docker Compose V2 directly. M3TAL orchestrates Docker Compose V2 for all stack operations.

*   **Running all stacks:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/*.yml up -d
    ```
*   **Stopping all stacks:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/*.yml down
    ```
*   **Managing the dashboard container specifically (e.g., if `m3tal dash up` failed):**
    ```bash
    # Assuming you want local expose mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard
    ```
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down m3tal-dashboard
    ```

---

This document is automatically generated and covers the standard command-line interface for M3TAL. Always refer to the official documentation for the most up-to-date information.