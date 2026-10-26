# M3TAL Command Reference

This document provides a comprehensive cheat-sheet for all available M3TAL CLI commands, their usage, and examples.

---

## Core M3TAL Commands

These commands are the primary interface for interacting with the M3TAL ecosystem.

*   **`m3tal`** or **`m3tal ui`**
    *   **Description:** Launches the M3TAL Linux Desktop Application. This is the graphical user interface for managing your M3TAL services.
    *   **Example:**
        ```bash
        m3tal ui
        ```

*   **`m3tal headless [port]`**
    *   **Description:** Starts M3TAL in headless mode. This is ideal for home servers or environments where a graphical desktop is not available. The optional `port` argument specifies the port the API daemon will listen on (default is 8080).
    *   **Example:**
        ```bash
        m3tal headless 8080
        ```

*   **`m3tal stats`**
    *   **Description:** Prints instant host system, GPU, thermal, and storage metrics to the terminal. Provides a quick overview of your system's health and resource utilization.
    *   **Example:**
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**
    *   **Description:** Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, the validity of your `.env` configuration, and checks for port availability.
    *   **Example:**
        ```bash
        m3tal doctor
        ```

*   **`m3tal config wizard`**
    *   **Description:** Initiates an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file. This is the recommended way to set up your environment variables.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

*   **`m3tal config set KEY VALUE`**
    *   **Description:** Sets a single environment variable within your `/etc/m3tal/.env` file. `KEY` is the name of the environment variable, and `VALUE` is its desired setting.
    *   **Example:**
        ```bash
        m3tal config set DOMAIN mydomain.local
        ```

*   **`m3tal config get KEY`**
    *   **Description:** Reads and displays the value of a single environment variable from your `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config get DOMAIN
        ```

*   **`m3tal config scan`**
    *   **Description:** Lists all environment variables found across all stacks managed by M3TAL, including their current values.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Description:** Displays the current contents of your `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config list
        ```

---

## Dashboard Management Commands

These commands control the M3TAL dashboard container.

*   **`m3tal dashpass [username] [password]`**
    *   **Description:** Updates the dashboard user's password. If `username` and `password` are omitted, the command will prompt you interactively.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Example (with arguments):**
        ```bash
        m3tal dashpass m3taluser new_secure_password
        ```

*   **`m3tal dash up`**
    *   **Description:** Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you have the most up-to-date dashboard setup.
    *   **Example:**
        ```bash
        m3tal dash up
        ```

*   **`m3tal dash down`**
    *   **Description:** Stops the dashboard container.
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
    *   **Description:** Streams the logs from the dashboard container to your terminal in real-time.
    *   **Example:**
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**
    *   **Description:** Shows the current status of the dashboard container (e.g., running, stopped).
    *   **Example:**
        ```bash
        m3tal dash status
        ```

---

## Stack Management Commands

These commands manage the Docker Compose stacks defined in your `/docker/` directory.

*   **`m3tal up`**
    *   **Description:** Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all your deployed M3TAL services.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Description:** Runs `docker compose down` across all managed stacks, stopping all running services.
    *   **Example:**
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**
    *   **Description:** Streams aggregated logs from all currently running M3TAL stacks to your terminal.
    *   **Example:**
        ```bash
        m3tal logs
        ```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service.

*   **`systemctl status m3tal-api`**
    *   **Description:** Checks the current status of the `m3tal-api.service` (e.g., active, inactive, failed).
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
    *   **Description:** Streams the logs from the `m3tal-api.service` to your terminal in real-time. Use `-f` to follow the logs.
    *   **Example:**
        ```bash
        journalctl -u m3tal-api -f
        ```

---

## Direct Docker Compose Commands (Fallback)

While `m3tal` aims to abstract Docker Compose, you can always use `docker compose` directly if needed. M3TAL primarily uses Docker Compose V2.

*   **Running all stacks:**
    *   **Description:** Executes `docker compose up` for all compose files in the `/docker` directory.
    *   **Example:**
        ```bash
        docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack.yml up -d
        ```
        *(Note: The exact files will depend on your installation and custom stacks. `m3tal up` automates this.)*

*   **Stopping all stacks:**
    *   **Description:** Executes `docker compose down` for all compose files in the `/docker` directory.
    *   **Example:**
        ```bash
        docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml -f /docker/my-custom-stack.yml down
        ```
        *(Note: `m3tal down` automates this.)*

*   **Managing the dashboard specifically:**
    *   **Description:** Starts the dashboard container using its specific compose file, applying the correct override based on `DASHBOARD_EXPOSE_MODE`.
    *   **Example:**
        ```bash
        # If DASHBOARD_EXPOSE_MODE=local
        docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard

        # If DASHBOARD_EXPOSE_MODE=traefik
        docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d m3tal-dashboard
        ```
        *(Note: `m3tal dash up` automates these complex commands.)*

---

## Installation

This is the standard procedure for installing the M3TAL CLI.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```