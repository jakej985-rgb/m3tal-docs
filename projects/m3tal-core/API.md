```markdown
# M3TAL CLI Command Reference

This document provides a comprehensive list of all available M3TAL CLI commands, including their usage and examples.

## Core M3TAL Commands

These commands interact directly with the M3TAL application and its core functionalities.

### Launching M3TAL

*   **`m3tal`**
    *   **Description:** Launches the M3TAL Linux Desktop Application. This is the primary command for accessing the graphical user interface.
    *   **Example:**
        ```bash
        m3tal
        ```

*   **`m3tal ui`**
    *   **Description:** An alias for `m3tal`, launches the M3TAL Linux Desktop Application.
    *   **Example:**
        ```bash
        m3tal ui
        ```

### Headless Mode

*   **`m3tal headless [port]`**
    *   **Description:** Starts M3TAL in headless mode. This is ideal for running M3TAL on home servers or systems without a graphical environment. The optional `port` argument specifies the port to listen on.
    *   **Example:**
        ```bash
        m3tal headless 8080
        ```

### System Information and Diagnostics

*   **`m3tal stats`**
    *   **Description:** Prints instant host system, GPU, thermal, and storage metrics to the terminal.
    *   **Example:**
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**
    *   **Description:** Performs a pre-flight health check. It verifies Docker connectivity, the validity of the `.env` file, and checks for port availability.
    *   **Example:**
        ```bash
        m3tal doctor
        ```

## Configuration Management

These commands allow you to manage M3TAL's configuration settings.

### Configuration Wizard

*   **`m3tal config wizard`**
    *   **Description:** Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

### Environment Variable Management

*   **`m3tal config set KEY VALUE`**
    *   **Description:** Sets a single environment variable in the M3TAL configuration.
    *   **Example:**
        ```bash
        m3tal config set DOMAIN mymetaldomain.com
        ```

*   **`m3tal config get KEY`**
    *   **Description:** Reads the value of a single environment variable from the M3TAL configuration.
    *   **Example:**
        ```bash
        m3tal config get DOMAIN
        ```

*   **`m3tal config scan`**
    *   **Description:** Lists all environment variables currently defined across all managed stacks.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Description:** Lists the contents of the current `.env` file.
    *   **Example:**
        ```bash
        m3tal config list
        ```

## Dashboard Management

These commands specifically manage the M3TAL dashboard container.

### Dashboard Authentication

*   **`m3tal dashpass [username] [password]`**
    *   **Description:** Updates the dashboard user password. If `username` and `password` are omitted, the command will prompt for them interactively.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Example (with arguments):**
        ```bash
        m3tal dashpass admin new_secure_password
        ```

### Dashboard Container Operations

*   **`m3tal dash up`**
    *   **Description:** Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.
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
    *   **Description:** Streams the logs from the dashboard container in real-time.
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

## Stack Management

These commands manage the Docker Compose stacks defined in the `/docker/` directory.

### Starting and Stopping Stacks

*   **`m3tal up`**
    *   **Description:** Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all defined services and stacks.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Description:** Runs `docker compose down` across all managed stacks, stopping and removing associated containers, networks, and volumes.
    *   **Example:**
        ```bash
        m3tal down
        ```

### Log Aggregation

*   **`m3tal logs`**
    *   **Description:** Streams aggregated logs from all currently running M3TAL stacks to the terminal.
    *   **Example:**
        ```bash
        m3tal logs
        ```

---

## Systemd Service Management

M3TAL's API daemon is managed by systemd for robust background operation.

*   **`systemctl status m3tal-api`**
    *   **Description:** Checks the current status of the `m3tal-api` systemd service.
    *   **Example:**
        ```bash
        systemctl status m3tal-api
        ```

*   **`systemctl restart m3tal-api`**
    *   **Description:** Restarts the `m3tal-api` systemd service.
    *   **Example:**
        ```bash
        systemctl restart m3tal-api
        ```

*   **`journalctl -u m3tal-api -f`**
    *   **Description:** Streams the logs from the `m3tal-api` systemd service in real-time. Use `Ctrl+C` to exit the stream.
    *   **Example:**
        ```bash
        journalctl -u m3tal-api -f
        ```

---

## Direct Docker Compose Commands (Fallback)

In scenarios where direct control is needed, you can use Docker Compose commands. M3TAL relies on Docker Engine and Docker Compose V2.

*   **To start all stacks:**
    ```bash
    docker compose --profile default up -d --remove-orphans
    ```
    *Note: This command should be executed from within the `/docker/` directory or when the `docker compose` command can resolve compose files from that location.*

*   **To stop all stacks:**
    ```bash
    docker compose down -v
    ```
    *Note: This command should be executed from within the `/docker/` directory or when the `docker compose` command can resolve compose files from that location.*

*   **To manage the dashboard specifically (if not using `m3tal dash up`/`down`):**
    Navigate to the directory containing `m3tal-compose.yml`, `m3tal-compose.local.yml`, or `m3tal-compose.traefik.yml` (depending on your `DASHBOARD_EXPOSE_MODE`) and use:
    ```bash
    docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d # For local mode
    docker compose -f m3tal-compose.yml -f m3tal-compose.traefik.yml up -d # For traefik mode
    docker compose -f m3tal-compose.yml -f <override_file> down
    ```
    *Note: The exact compose files used depend on the `DASHBOARD_EXPOSE_MODE` configuration.*

---

## M3TAL APT Installation

To install M3TAL, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```
```