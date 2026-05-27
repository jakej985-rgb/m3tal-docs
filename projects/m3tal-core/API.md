```md
# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for all available `m3tal` CLI commands, their purpose, and real-world usage examples.

---

## Systemd Service Management

The M3TAL API daemon is managed by `systemd`. You can interact with it using the following commands:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **Restart the service:**
    ```bash
    systemctl restart m3tal-api
    ```

*   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Docker Compose Fallback

While the `m3tal` CLI is the primary interface, you can directly interact with Docker Compose for advanced troubleshooting or manual control. All compose files are located in `/docker/`.

*   **Start all services:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
    ```
    *(Note: The specific compose files used here reflect a common `local` mode setup. Adjust as needed for your configuration.)*

*   **Stop all services:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down
    ```
    *(Note: The specific compose files used here reflect a common `local` mode setup. Adjust as needed for your configuration.)*

*   **View logs from all services:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml logs -f
    ```
    *(Note: The specific compose files used here reflect a common `local` mode setup. Adjust as needed for your configuration.)*

---

## M3TAL CLI Commands

### Interactive TUI

*   **`sudo m3tal`**
    *   **Purpose:** Opens the interactive TUI Control Center, presenting a numbered menu for various operations.
    *   **Example:**
        ```bash
        sudo m3tal
        ```
        *(This will display a menu of options like "Init", "Config", "Dashboard", "System", etc.)*

### Initialization and Configuration

*   **`m3tal init`**
    *   **Purpose:** Generates the default `/etc/m3tal/.env` file. Use this on a fresh installation to create the initial configuration.
    *   **Example:**
        ```bash
        m3tal init
        ```

*   **`m3tal config wizard`**
    *   **Purpose:** Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

*   **`m3tal config set KEY VALUE`**
    *   **Purpose:** Sets a single environment variable in `/etc/m3tal/.env`.
    *   **Example:**
        ```bash
        m3tal config set API_TOKEN my-super-secret-api-token
        ```

*   **`m3tal config get KEY`**
    *   **Purpose:** Reads and displays the value of a single environment variable from `/etc/m3tal/.env`.
    *   **Example:**
        ```bash
        m3tal config get DASHBOARD_PORT
        ```
        *(This might output `8082`)*

*   **`m3tal config scan`**
    *   **Purpose:** Lists all environment variables currently recognized by M3TAL across all managed stacks, showing their current values.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Purpose:** Displays the entire content of the current `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config list
        ```

### Dashboard Management

*   **`m3tal dashpass [username] [password]`**
    *   **Purpose:** Updates the password for a dashboard user. If username and password are not provided, the command becomes interactive.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
        *(Prompts for username and new password)*
    *   **Example (non-interactive):**
        ```bash
        m3tal dashpass admin new_secure_password
        ```

*   **`m3tal dash up`**
    *   **Purpose:** Pulls the latest dashboard compose configuration from GitHub and starts the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash up
        ```

*   **`m3tal dash down`**
    *   **Purpose:** Stops and removes the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash down
        ```

*   **`m3tal dash restart`**
    *   **Purpose:** Restarts the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash restart
        ```

*   **`m3tal dash logs`**
    *   **Purpose:** Streams the logs from the dashboard container in real-time.
    *   **Example:**
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**
    *   **Purpose:** Shows the current status of the dashboard container (e.g., running, exited).
    *   **Example:**
        ```bash
        m3tal dash status
        ```

### Global System Management

*   **`m3tal up`**
    *   **Purpose:** Runs `docker compose up -d` across all `*-compose.yml` files found in the `/docker/` directory, starting all managed services.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Purpose:** Runs `docker compose down` for all stacks defined in the `/docker/` directory, stopping and removing associated containers, networks, and volumes.
    *   **Example:**
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**
    *   **Purpose:** Streams aggregated logs from all currently running M3TAL services.
    *   **Example:**
        ```bash
        m3tal logs
        ```

---

## APT Installation

To install or update M3TAL on your system, use the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```
```