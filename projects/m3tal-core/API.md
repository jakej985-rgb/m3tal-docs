```markdown
# M3TAL CLI Command Reference

This document provides a comprehensive reference for all M3TAL command-line interface (CLI) commands.

## Table of Contents

1.  [M3TAL Core Commands](#m3tal-core-commands)
2.  [M3TAL Configuration](#m3tal-configuration)
3.  [M3TAL Dashboard Management](#m3tal-dashboard-management)
4.  [M3TAL Stack Management](#m3tal-stack-management)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

## M3TAL Core Commands

These commands provide general system information and health checks.

*   **`m3tal` or `m3tal ui`**: Launches the M3TAL Linux Desktop Application.
    *   **Usage:**
        ```bash
        m3tal ui
        ```
*   **`m3tal headless [port]`**: Starts M3TAL in headless mode, ideal for home servers.
    *   **Usage:**
        ```bash
        m3tal headless 8080
        ```
*   **`m3tal stats`**: Prints instant host system, GPU, thermal, and storage metrics to the terminal.
    *   **Usage:**
        ```bash
        m3tal stats
        ```
*   **`m3tal doctor`**: Performs a pre-flight health check, verifying Docker connectivity, `.env` validity, and port availability.
    *   **Usage:**
        ```bash
        m3tal doctor
        ```

## M3TAL Configuration

Manage M3TAL's environment variables and configuration file.

*   **`m3tal config wizard`**: Starts an interactive wizard to configure `/etc/m3tal/.env`.
    *   **Usage:**
        ```bash
        m3tal config wizard
        ```
*   **`m3tal config set KEY VALUE`**: Sets a single environment variable in the `.env` file.
    *   **Usage:**
        ```bash
        m3tal config set DASHBOARD_PORT 8082
        ```
*   **`m3tal config get KEY`**: Reads and prints the value of a single environment variable from the `.env` file.
    *   **Usage:**
        ```bash
        m3tal config get DOMAIN
        ```
*   **`m3tal config scan`**: Lists all environment variables across all stacks defined in your configuration.
    *   **Usage:**
        ```bash
        m3tal config scan
        ```
*   **`m3tal config list`**: Lists the current contents of the `.env` file.
    *   **Usage:**
        ```bash
        m3tal config list
        ```

## M3TAL Dashboard Management

Commands specifically for managing the M3TAL dashboard container.

*   **`m3tal dashpass [username] [password]`**: Updates the dashboard user password. If username and password are omitted, the command becomes interactive.
    *   **Usage (Interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Usage (Non-interactive):**
        ```bash
        m3tal dashpass admin new_secure_password
        ```
*   **`m3tal dash up`**: Pulls the latest dashboard compose configuration from GitHub and starts the dashboard container.
    *   **Usage:**
        ```bash
        m3tal dash up
        ```
*   **`m3tal dash down`**: Stops the dashboard container.
    *   **Usage:**
        ```bash
        m3tal dash down
        ```
*   **`m3tal dash restart`**: Restarts the dashboard container.
    *   **Usage:**
        ```bash
        m3tal dash restart
        ```
*   **`m3tal dash logs`**: Streams the logs from the dashboard container.
    *   **Usage:**
        ```bash
        m3tal dash logs
        ```
*   **`m3tal dash status`**: Shows the current status of the dashboard container.
    *   **Usage:**
        ```bash
        m3tal dash status
        ```

## M3TAL Stack Management

Commands to manage all running M3TAL stacks.

*   **`m3tal up`**: Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all your defined services.
    *   **Usage:**
        ```bash
        m3tal up
        ```
*   **`m3tal down`**: Runs `docker compose down` across all stacks. This stops and removes all containers defined in your compose files.
    *   **Usage:**
        ```bash
        m3tal down
        ```
*   **`m3tal logs`**: Streams aggregated logs from all running M3TAL stacks.
    *   **Usage:**
        ```bash
        m3tal logs
        ```

## Systemd Service Management

M3TAL's core API daemon is managed by systemd.

*   **`systemctl status m3tal-api`**: Check the status of the M3TAL API service.
    *   **Usage:**
        ```bash
        systemctl status m3tal-api
        ```
*   **`systemctl restart m3tal-api`**: Restart the M3TAL API service.
    *   **Usage:**
        ```bash
        systemctl restart m3tal-api
        ```
*   **`journalctl -u m3tal-api -f`**: View and follow the logs for the M3TAL API service in real-time.
    *   **Usage:**
        ```bash
        journalctl -u m3tal-api -f
        ```

## Direct Docker Compose Fallback

In scenarios where M3TAL CLI commands are unavailable or for advanced debugging, you can interact with Docker Compose directly.

*   **Running all stacks:**
    *   **To start:**
        ```bash
        docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-custom-stack.yml up -d
        ```
        *(Note: Adjust file paths as needed for your specific setup)*
    *   **To stop:**
        ```bash
        docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/my-custom-stack.yml down
        ```
*   **Managing the dashboard specifically:**
    *   **To start (example for local mode):**
        ```bash
        docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard
        ```
    *   **To stop:**
        ```bash
        docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down m3tal-dashboard
        ```
*   **Viewing logs for a specific service:**
    *   **Example for the dashboard:**
        ```bash
        docker compose logs -f m3tal-dashboard
        ```

---

### APT Installation

To install M3TAL on your system, use the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```
```