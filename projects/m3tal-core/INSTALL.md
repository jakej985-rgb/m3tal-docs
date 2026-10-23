`docs/GET_STARTED.md`

# M3TAL Ecosystem: Getting Started Guide

This document provides a step-by-step guide for setting up M3TAL on your system.

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation by running:

```bash
docker --version && docker compose version
```

## 2. Install M3TAL via APT

M3TAL is distributed as a single Go binary via an APT repository. Execute the following commands to install it:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, configure M3TAL using the interactive wizard. This sets up essential environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several configuration options:

*   **`DOMAIN`**: The base domain for your M3TAL services (e.g., `example.com`). If you don't have a domain or are running M3TAL locally, use `localhost`. This is primarily used for Traefik routing.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard will be directly accessible via `http://YOUR_IP:8082`. No Traefik configuration is required for dashboard access. This is recommended for first-time users or local network setups.
    *   `traefik`: The dashboard will be accessible via `http://dash.YOUR_DOMAIN`, routed through Traefik. This requires Traefik to be running.
*   **`PUID` / `PGID`**: The User ID (PUID) and Group ID (PGID) that Docker containers will use. It is recommended to use the PUID and PGID of your current user to ensure proper file permissions. You can find these with `id -u` and `id -g`.
*   **`TZ`**: Your local timezone (e.g., `America/Denver`).
*   **`DASHBOARD_SECRET`**: A secret key used for encrypting session data in the dashboard. Generate a strong, random string.
*   **`API_TOKEN`**: A token used for API authentication. Generate a strong, random string.
*   **`ADMIN_PASSWORD`**: The initial default password for the M3TAL Dashboard's `admin` user. **Change this immediately after login.**
*   **`BASE_STORAGE_PATH`**: The base directory on your host where application data volumes will be stored (e.g., `/mnt/data`).
*   **`CONFIG_PATH`**, **`MEDIA_PATH`**, **`DOWNLOADS_PATH`**: Subdirectories within `BASE_STORAGE_PATH` for specific data types.
*   **`TRAEFIK_WEB_PORT`** / **`TRAEFIK_WEBHTTPS_PORT`**: If you need to change the default Traefik ports (80 for HTTP, 443 for HTTPS).
*   Other prompts (e.g., `LOG_LEVEL`, `DEBUG_MODE`) can usually be left at their defaults for a first-time setup.

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. This includes the core routing stack (Traefik).

```bash
m3tal up
```

This command will:
*   Pull the `traefik:latest` image if not present.
*   Start the Traefik container, exposing port 80 (and 443 if configured) for HTTP/HTTPS routing.
*   Start the optional `cloudflared` container if enabled in your `routing-compose.yml`.

## 5. Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container.

```bash
m3tal dash up
```

This command will:
*   Download the latest dashboard compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
*   Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
*   Start the `m3tal-dashboard` container with the appropriate configuration (either directly exposing port 8082 or configuring Traefik labels).

## 6. Access the Dashboard

The method to access the dashboard depends on the `DASHBOARD_EXPOSE_MODE` you configured in Step 3.

*   **If `DASHBOARD_EXPOSE_MODE` is `local`:**
    Open your web browser and navigate to `http://YOUR_HOST_IP:8082` (replace `YOUR_HOST_IP` with the IP address of your server, or `localhost` if accessing from the server itself).
    *Example: `http://192.168.1.100:8082`*

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Open your web browser and navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in the wizard). Traefik must be running for this to work.
    *Example: `http://dash.example.com`*

## 7. Log In

Use the default credentials to log in:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the configuration wizard (Step 3).

**Immediately change the `admin` user's password** after your first login for security purposes. You can do this within the dashboard interface or via the CLI:

```bash
sudo m3tal dashpass admin
```

This command will prompt you to set a new password for the `admin` user.

---

## Filesystem Contract

M3TAL establishes a clear filesystem contract to organize its components and user data.

| Path                     | Purpose                                                                                                                                                                                                                                                                                                                                                                      |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | **Primary configuration file.** Contains all environment variables for M3TAL operations. This file is managed by the `m3tal config wizard` and `m3tal config set` commands.                                                                                                                                                                                           |
| `/var/lib/m3tal/state.db` | **SQLite state database.** This database is automatically created and managed by the `m3tal-api.service` daemon. It stores internal M3TAL state, such as registered Docker services and their configurations.                                                                                                                                                           |
| `/opt/m3tal/stack/`      | **Canonical stack directory.** This directory holds all core Docker Compose files (`routing-compose.yml`, `m3tal-compose.yml`, etc.) and Traefik dynamic configuration files. While this is the actual location, users typically interact with the symlink `/docker/`.                                                                                              |
| `/docker`                | **Symlink to `/opt/m3tal/stack/`.** This is the **user-facing path** for all Docker Compose stack operations. When you place a new `*-compose.yml` file here, `m3tal up` will include it.                                                                                                                                                                           |
| `/docker/users.json`     | **Dashboard credential store.** This file stores the hashed credentials for M3TAL Dashboard users. It is managed by the `m3tal dashpass` command. **Do not modify this file directly.**                                                                                                                                                                                |

## M3TAL Dashboard Access Modes

The M3TAL Dashboard's exposure mechanism is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`. This choice affects how Docker Compose orchestrates the dashboard container.

### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: M3TAL uses the `m3tal-compose.local.yml` override file, which adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container.
*   **Access Method**: `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requirements**: No Traefik is required for dashboard access in this mode.
*   **Use Case**: Ideal for initial setup, local testing, or scenarios where the dashboard is only accessed within a local network (LAN-only setups).

### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: M3TAL uses the `m3tal-compose.traefik.yml` override file. This file adds Traefik labels to the dashboard container, instructing Traefik to route traffic from `dash.${DOMAIN}` to the dashboard's internal port (8082).
*   **Access Method**: `http://dash.DOMAIN` (e.g., `http://dash.example.com`)
*   **Requirements**: Traefik must be running via `m3tal up` to route requests to the dashboard. You must have also configured a valid `DOMAIN` in `/etc/m3tal/.env`.
*   **Use Case**: Suited for domain-based setups where you want all services, including the dashboard, to be accessible through a single reverse proxy (Traefik).

## Understanding M3TAL's Docker Runtime

M3TAL relies on **Docker Engine and Docker Compose V2** as its underlying container orchestration tools.

*   The `m3tal up` command acts as a wrapper for `docker compose`. It iterates through all `*-compose.yml` files present in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`) and starts the services defined within them. This means any user-defined `*-compose.yml` file placed in `/docker/` will be brought up by `m3tal up`.
*   The `m3tal dash up` command is a specialized operation for the M3TAL Dashboard. It performs the following actions:
    1.  Downloads the latest base and override compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from the M3TAL GitHub repository.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the `m3tal-dashboard` container using the appropriate compose override file (`m3tal-compose.local.yml` for `local` mode or `m3tal-compose.traefik.yml` for `traefik` mode), along with the base `m3tal-compose.yml`.

## Port Map

The following ports are used by M3TAL components:

| Port | Service                     | Access                                                                  |
| :--- | :-------------------------- | :---------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public (if `TRAEFIK_WEB_PORT` is 80 and Traefik is exposed)             |
| 8080 | M3TAL API daemon (Go binary) | Host-local only, used by dashboard and Traefik (via `host.docker.internal`) |
| 8081 | Traefik Dashboard           | Host-local only (`127.0.0.1:8081`)                                      |
| 8082 | M3TAL Dashboard             | Direct port binding (local mode) or via Traefik (traefik mode)          |

## Firewall Note

If you are running a firewall (e.g., `ufw`) and have configured Traefik to be publicly accessible (e.g., on port 80), you may need to open the required ports:

```bash
sudo ufw allow 80/tcp
sudo ufw enable
```

Adjust the port if you've configured Traefik to listen on a different `TRAEFIK_WEB_PORT`.

## Service Management (M3TAL API Daemon)

The M3TAL API daemon (`m3tal-api`) runs as a systemd service. You can manage its state using standard `systemctl` commands:

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **Stop the service**:
    ```bash
    sudo systemctl stop m3tal-api
    ```
*   **Start the service**:
    ```bash
    sudo systemctl start m3tal-api
    ```

---

## Day 2 Operations: Installing New Stacks

To deploy additional Docker Compose services (user stacks) with M3TAL:

1.  **Create your Compose File**: Place your `docker-compose.yml` (or any `*-compose.yml` file) into the `/docker/` directory. For example, `/docker/my-stack-compose.yml`.
2.  **Configure Environment Variables**: Ensure all required environment variables for your new stack are set in `/etc/m3tal/.env`. You can use `m3tal config wizard` or `m3tal config set KEY value` to manage these variables.
3.  **Start All Stacks**: Run `m3tal up` to start all services, including your newly added stack. M3TAL will automatically detect and bring up all valid compose files in `/docker/`.