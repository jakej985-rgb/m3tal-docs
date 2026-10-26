# M3TAL CLI Command Reference

This document provides a comprehensive cheat sheet for all M3TAL CLI commands, designed to help you manage and interact with your M3TAL environment efficiently.

## M3TAL CLI Commands

The M3TAL CLI is the primary interface for managing your M3TAL system. It's a unified Go binary that orchestrates various services and configurations.

### Core Functionality

*   **`m3tal`** or **`m3tal ui`**
    *   **Description:** Launches the M3TAL Linux Desktop Application. This is the graphical user interface for interacting with M3TAL.
    *   **Example:**
        ```bash
        m3tal
        ```

*   **`m3tal headless [port]`**
    *   **Description:** Starts M3TAL in headless mode. This is ideal for running M3TAL on a server without a graphical interface. You can optionally specify a port to listen on.
    *   **Example:**
        ```bash
        m3tal headless 8080
        ```

*   **`m3tal stats`**
    *   **Description:** Prints instant host system, GPU, thermal, and storage metrics to the terminal. This command provides a quick overview of your system's health and performance.
    *   **Example:**
        ```bash
        m3tal stats
        ```

*   **`m3tal doctor`**
    *   **Description:** Performs a pre-flight health check. It verifies Docker connectivity, the validity of your `.env` configuration, and checks for port availability.
    *   **Example:**
        ```bash
        m3tal doctor
        ```

### Configuration Management

*   **`m3tal config wizard`**
    *   **Description:** Initiates an interactive wizard to configure the primary M3TAL configuration file located at `/etc/m3tal/.env`. This is the recommended way to set up your initial environment.
    *   **Example:**
        ```bash
        m3tal config wizard
        ```

*   **`m3tal config set KEY VALUE`**
    *   **Description:** Sets a single environment variable in the `/etc/m3tal/.env` file. Replace `KEY` with the variable name and `VALUE` with its desired setting.
    *   **Example:**
        ```bash
        m3tal config set DOMAIN mym3tal.local
        ```

*   **`m3tal config get KEY`**
    *   **Description:** Reads a single environment variable from the `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config get DOMAIN
        ```

*   **`m3tal config scan`**
    *   **Description:** Lists all environment variables across all active M3TAL stacks. This provides a comprehensive view of your M3TAL configuration.
    *   **Example:**
        ```bash
        m3tal config scan
        ```

*   **`m3tal config list`**
    *   **Description:** Displays the current contents of the `/etc/m3tal/.env` file.
    *   **Example:**
        ```bash
        m3tal config list
        ```

### Dashboard Management

The M3TAL dashboard provides a web-based interface for managing your M3TAL services.

*   **`m3tal dashpass [username] [password]`**
    *   **Description:** Updates the dashboard user password. If `username` and `password` are omitted, the command will become interactive. The credentials are stored in `/docker/users.json`.
    *   **Example (interactive):**
        ```bash
        m3tal dashpass
        ```
    *   **Example (non-interactive):**
        ```bash
        m3tal dashpass admin new_secure_password
        ```

*   **`m3tal dash up`**
    *   **Description:** Pulls the latest dashboard Docker Compose configuration from GitHub and then starts the dashboard container. This ensures you're running the most up-to-date dashboard setup.
    *   **Example:**
        ```bash
        m3tal dash up
        ```

*   **`m3tal dash down`**
    *   **Description:** Stops the dashboard Docker container.
    *   **Example:**
        ```bash
        m3tal dash down
        ```

*   **`m3tal dash restart`**
    *   **Description:** Restarts the dashboard Docker container.
    *   **Example:**
        ```bash
        m3tal dash restart
        ```

*   **`m3tal dash logs`**
    *   **Description:** Streams the logs from the dashboard Docker container. This is useful for troubleshooting.
    *   **Example:**
        ```bash
        m3tal dash logs
        ```

*   **`m3tal dash status`**
    *   **Description:** Shows the current status of the dashboard Docker container.
    *   **Example:**
        ```bash
        m3tal dash status
        ```

### Stack Management

These commands manage the Docker Compose stacks that power your M3TAL services.

*   **`m3tal up`**
    *   **Description:** Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This starts all your deployed M3TAL stacks.
    *   **Example:**
        ```bash
        m3tal up
        ```

*   **`m3tal down`**
    *   **Description:** Runs `docker compose down` across all stacks defined in the `/docker/` directory. This stops all running M3TAL services.
    *   **Example:**
        ```bash
        m3tal down
        ```

*   **`m3tal logs`**
    *   **Description:** Streams aggregated logs from all running M3TAL stacks. This provides a centralized view of your system's activity.
    *   **Example:**
        ```bash
        m3tal logs
        ```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service, allowing for robust background operation and management.

*   **`systemctl status m3tal-api`**
    *   **Description:** Checks the current status of the `m3tal-api.service`.
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
    *   **Description:** Streams the logs from the `m3tal-api.service` in real-time. Use `-f` to follow the log output. Press `Ctrl+C` to exit.
    *   **Example:**
        ```bash
        journalctl -u m3tal-api -f
        ```

---

## Direct Docker Compose Commands (Fallback)

M3TAL utilizes Docker Engine and Docker Compose V2. In cases where the `m3tal` CLI might not be sufficient or for advanced debugging, you can interact with Docker Compose directly.

*   **Starting all stacks:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml up -d
    # Add any other -f paths for your custom stacks, e.g., -f /docker/my-custom-stack.yml
    ```

*   **Stopping all stacks:**
    ```bash
    docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml down
    # Add any other -f paths for your custom stacks, e.g., -f /docker/my-custom-stack.yml
    ```

*   **Starting only the dashboard (if `DASHBOARD_EXPOSE_MODE=local`):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard
    ```

*   **Starting only the dashboard (if `DASHBOARD_EXPOSE_MODE=traefik`):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```

*   **Stopping the dashboard:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml down m3tal-dashboard
    ```

*   **Viewing dashboard logs:**
    ```bash
    docker logs -f m3tal-dashboard
    ```

---

## M3TAL Filesystem Contract

Understanding these paths is crucial for managing M3TAL configurations and data.

| Path                       | Purpose                                                                 | Managed By                                      |
| :------------------------- | :---------------------------------------------------------------------- | :---------------------------------------------- |
| `/etc/m3tal/.env`          | Primary configuration file. Stores all environment variables.           | `m3tal config wizard`, `m3tal config set`       |
| `/var/lib/m3tal/state.db`  | SQLite state database for the API daemon.                               | M3TAL API daemon                                |
| `/opt/m3tal/stack/`        | Canonical stack directory containing Compose files and Traefik config.  | M3TAL Installation & Updates                    |
| `/docker`                  | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations. | M3TAL Installation & Updates                    |
| `/docker/users.json`       | Dashboard credential store.                                             | `m3tal dashpass`                                |

---

## APT Installation

To install or update M3TAL, use the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Update package lists and install M3TAL
sudo apt update && sudo apt install -y m3tal
```