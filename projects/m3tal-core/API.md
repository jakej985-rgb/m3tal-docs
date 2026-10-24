# M3TAL CLI Command Reference

This document provides a comprehensive reference for all M3TAL command-line interface (CLI) commands, serving as a cheat sheet for managing your M3TAL ecosystem.

## Table of Contents

1.  [M3TAL Core Commands](#m3tal-core-commands)
2.  [M3TAL Configuration Management](#m3tal-configuration-management)
3.  [M3TAL Dashboard Management](#m3tal-dashboard-management)
4.  [M3TAL Stack Management](#m3tal-stack-management)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## M3TAL Core Commands

These commands are your primary interface for interacting with the M3TAL system.

*   **`m3tal`** or **`m3tal ui`**
    *   **Description:** Launches the M3TAL Linux Desktop Application. This is the graphical user interface for managing your M3TAL instances.
    *   **Example:**
        ```bash
        m3tal ui
        ```

*   **`m3tal headless [port]`**
    *   **Description:** Starts M3TAL in headless mode. This is ideal for running M3TAL on home servers or systems without a graphical interface. The optional `[port]` argument specifies the port to bind the API to.
    *   **Example:**
        ```bash
        m3tal headless 8080
        ```

*   **`m3tal stats`**
    *   **Description:** Prints instant host system, GPU, thermal, and storage metrics to the terminal. This provides a quick overview of your system's health and performance.
    *   **Example:**
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**
    *   **Description:** Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, checks the validity of your `.env` file, and ensures necessary ports are available.
    *   **Example:**
        ```bash
        m3tal doctor
        ```

---

## M3TAL Configuration Management

Manage your M3TAL environment variables and configurations using these commands.

*   **`m3tal config wizard`**
    *   **Description:** Launches an interactive wizard to configure the primary M3TAL configuration file located at `/etc/m3tal/.env`.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

*   **`m3tal config set KEY VALUE`**
    *   **Description:** Sets a single environment variable in your M3TAL configuration. Replace `KEY` with the environment variable name and `VALUE` with its desired setting.
    *   **Example:**
        ```bash
        m3tal config set DOMAIN mym3tal.example.com
        ```

*   **`m3tal config get KEY`**
    *   **Description:** Reads and prints the value of a single environment variable from your M3TAL configuration.
    *   **Example:**
        ```bash
        m3tal config get LOG_LEVEL
        ```

*   **`m3tal config scan`**
    *   **Description:** Lists all environment variables across all configured M3TAL stacks, providing a comprehensive view of your settings.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Description:** Lists the current contents of your `.env` file, displaying all configured environment variables and their values.
    *   **Example:**
        ```bash
        m3tal config list
        ```

---

## M3TAL Dashboard Management

Control the M3TAL dashboard container and its credentials.

*   **`m3tal dashpass [username] [password]`**
    *   **Description:** Updates the dashboard user password. If `[username]` and `[password]` are omitted, the command will prompt you interactively for these details. The credentials are stored in `/docker/users.json`.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Example (non-interactive):**
        ```bash
        m3tal dashpass m3taluser new_secure_password
        ```

*   **`m3tal dash up`**
    *   **Description:** Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you are running the most up-to-date dashboard setup.
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
    *   **Description:** Streams the logs from the dashboard container in real-time. This is invaluable for debugging.
    *   **Example:**
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**
    *   **Description:** Shows the current status of the dashboard container.
    *   **Example:**
        ```bash
        m3tal dash status
        ```

---

## M3TAL Stack Management

Manage all your deployed M3TAL stacks using these commands.

*   **`m3tal up`**
    *   **Description:** Runs `docker compose up` across all `*-compose.yml` files located in the `/docker/` directory. This starts all your defined M3TAL services.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Description:** Runs `docker compose down` across all stacks, stopping and removing all containers defined by the compose files in `/docker/`.
    *   **Example:**
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**
    *   **Description:** Streams aggregated logs from all running M3TAL stacks to your terminal, providing a consolidated view of your system's activity.
    *   **Example:**
        ```bash
        m3tal logs
        ```

---

## Systemd Service Management

The M3TAL API daemon is managed by systemd. You can interact with it using `systemctl` and `journalctl`.

*   **`systemctl status m3tal-api`**
    *   **Description:** Displays the current status of the `m3tal-api.service` systemd unit.
    *   **Example:**
        ```bash
        systemctl status m3tal-api
        ```

*   **`systemctl restart m3tal-api`**
    *   **Description:** Restarts the `m3tal-api.service` systemd unit.
    *   **Example:**
        ```bash
        systemctl restart m3tal-api
        ```

*   **`journalctl -u m3tal-api -f`**
    *   **Description:** Streams the logs for the `m3tal-api.service` unit in real-time. This is equivalent to `m3tal logs` specifically for the API daemon.
    *   **Example:**
        ```bash
        journalctl -u m3tal-api -f
        ```

---

## Direct Docker Compose Fallback

In scenarios where direct control over Docker Compose is needed, you can execute the commands manually. M3TAL leverages Docker Compose V2.

*   **Starting all stacks:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.$(grep -oP '(?<=DASHBOARD_EXPOSE_MODE=)\w+' /etc/m3tal/.env).yml up -d
    ```
    *(Note: This example assumes a basic setup. You may need to adjust the `-f` flags based on your specific compose files in `/docker/` and the value of `DASHBOARD_EXPOSE_MODE`)*

*   **Stopping all stacks:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.$(grep -oP '(?<=DASHBOARD_EXPOSE_MODE=)\w+' /etc/m3tal/.env).yml down
    ```
    *(Note: Similar to `up`, adjust flags as needed)*

*   **Starting only the dashboard (requires specifying the correct override):**
    ```bash
    # For local mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard

    # For traefik mode
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```

*   **Stopping only the dashboard:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.$(grep -oP '(?<=DASHBOARD_EXPOSE_MODE=)\w+' /etc/m3tal/.env).yml down m3tal-dashboard
    ```

---

## APT Installation

To install or update M3TAL on your system, use the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install or update M3TAL
sudo apt update && sudo apt install -y m3tal
```