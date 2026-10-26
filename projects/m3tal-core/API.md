# M3TAL CLI Command Reference

This document serves as a comprehensive cheat-sheet for the M3TAL CLI, providing a quick reference for all available commands and their usage.

---

## Table of Contents

1.  [Core M3TAL Commands](#core-m3tal-commands)
2.  [Configuration Management](#configuration-management)
3.  [Dashboard Management](#dashboard-management)
4.  [Systemd Service Management](#systemd-service-management)
5.  [Direct Docker Compose Commands](#direct-docker-compose-commands)
6.  [APT Installation](#apt-installation)

---

## Core M3TAL Commands

These commands provide fundamental operations for launching, monitoring, and managing your M3TAL system.

*   **`m3tal`** or **`m3tal ui`**: Launches the M3TAL Linux Desktop Application.
    *   **Usage**:
        ```bash
        m3tal
        ```
        or
        ```bash
        m3tal ui
        ```

*   **`m3tal headless [port]`**: Starts M3TAL in headless mode, ideal for home servers. You can optionally specify a port.
    *   **Usage**:
        ```bash
        m3tal headless
        ```
        or
        ```bash
        m3tal headless 8080
        ```

*   **`m3tal stats`**: Prints instant host system, GPU, thermal, and storage metrics to the terminal.
    *   **Usage**:
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**: Performs a pre-flight health check, verifying Docker connectivity, `.env` validity, and port availability.
    *   **Usage**:
        ```bash
        m3tal doctor
        ```

---

## Configuration Management

Manage your M3TAL environment variables and configuration files.

*   **`m3tal config wizard`**: Launches an interactive wizard to configure `/etc/m3tal/.env`.
    *   **Usage**:
        ```bash
        m3tal config wizard
        ```

*   **`m3tal config set KEY VALUE`**: Sets a single environment variable in your `.env` file.
    *   **Usage**:
        ```bash
        m3tal config set DOMAIN mydomain.com
        ```

*   **`m3tal config get KEY`**: Reads and displays the value of a single environment variable from your `.env` file.
    *   **Usage**:
        ```bash
        m3tal config get DASHBOARD_PORT
        ```

*   **`m3tal config scan`**: Lists all environment variables across all managed stacks.
    *   **Usage**:
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**: Lists the current contents of your `.env` file.
    *   **Usage**:
        ```bash
        m3tal config list
        ```

---

## Dashboard Management

Control and monitor the M3TAL dashboard application.

*   **`m3tal dashpass [username] [password]`**: Updates the dashboard user password. If arguments are omitted, it will prompt interactively.
    *   **Usage**:
        ```bash
        m3tal dashpass
        ```
        or
        ```bash
        m3tal dashpass admin mynewsecretpassword
        ```

*   **`m3tal dash up`**: Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.
    *   **Usage**:
        ```bash
        m3tal dash up
        ```

*   **`m3tal dash down`**: Stops the dashboard container.
    *   **Usage**:
        ```bash
        m3tal dash down
        ```

*   **`m3tal dash restart`**: Restarts the dashboard container.
    *   **Usage**:
        ```bash
        m3tal dash restart
        ```

*   **`m3tal dash logs`**: Streams the dashboard container logs.
    *   **Usage**:
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**: Shows the current status of the dashboard container.
    *   **Usage**:
        ```bash
        m3tal dash status
        ```

---

## System Management

Commands for managing all deployed stacks and services.

*   **`m3tal up`**: Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This command starts all your deployed stacks.
    *   **Usage**:
        ```bash
        m3tal up
        ```

*   **`m3tal down`**: Runs `docker compose down` across all stacks, stopping all your deployed services.
    *   **Usage**:
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**: Streams aggregated logs from all currently running M3TAL stacks.
    *   **Usage**:
        ```bash
        m3tal logs
        ```

---

## Systemd Service Management

The M3TAL API daemon is managed by `systemd`. You can interact with it using `systemctl` and view its logs with `journalctl`.

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```

*   **Restart the service**:
    ```bash
    systemctl restart m3tal-api
    ```

*   **Stream logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Direct Docker Compose Commands (Fallback)

While the `m3tal` CLI provides a unified interface, you can always interact directly with Docker Compose for specific stack management if needed. M3TAL uses Docker Compose V2.

*   **Navigate to the stack directory**: The primary location for compose files is `/docker/`.
    ```bash
    cd /docker
    ```

*   **Start all services in the current directory**:
    ```bash
    docker compose up -d
    ```

*   **Stop all services in the current directory**:
    ```bash
    docker compose down
    ```

*   **View logs for a specific service**:
    ```bash
    docker compose logs -f <service_name>
    ```
    *   **Example**: `docker compose logs -f m3tal-dashboard`

*   **Check the status of services**:
    ```bash
    docker compose ps
    ```

*   **Pull latest images for services**:
    ```bash
    docker compose pull
    ```

---

## APT Installation

This section details how to install M3TAL via your system's package manager.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Update package lists and install M3TAL
sudo apt update && sudo apt install -y m3tal
```