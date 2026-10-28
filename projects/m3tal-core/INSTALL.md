This document provides a complete guide for installing and configuring M3TAL on a Linux system.

---

# M3TAL Setup Guide

This guide details the steps required to install M3TAL, configure its core components, and access its dashboard.

## Step 1: Prerequisites

M3TAL requires a functional Docker environment. Ensure Docker Engine and Docker Compose V2 are installed on your system.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Expected output will show versions for Docker Engine and Docker Compose V2.

## Step 2: Install M3TAL via APT

Install the M3TAL CLI and API daemon using the provided APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

This installs the `m3tal` CLI binary (`/usr/bin/m3tal`) and sets up the `m3tal-api.service` systemd daemon.

## Step 3: Run the M3TAL Configuration Wizard

Initialize M3TAL's primary configuration file (`/etc/m3tal/.env`) using the interactive wizard.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several configuration parameters:

*   **`DASHBOARD_EXPOSE_MODE` (local/traefik)**:
    *   `local` (default): The M3TAL Dashboard will be directly accessible via `http://YOUR_IP:8082`. This mode does not require Traefik.
    *   `traefik`: The M3TAL Dashboard will be accessible via `http://dash.YOUR_DOMAIN` through the Traefik reverse proxy. This mode requires Traefik to be running.
*   **`DOMAIN`**: The base domain for services exposed via Traefik (e.g., `example.com`). If using `DASHBOARD_EXPOSE_MODE=traefik`, the dashboard will be at `dash.DOMAIN`.
*   **`PUID` / `PGID`**: The User ID and Group ID that M3TAL containers will run as, typically `1000` for the default user.
*   **`TZ`**: Your system's timezone (e.g., `America/Denver`).
*   **`BASE_STORAGE_PATH`**: The base directory for M3TAL's data (e.g., `/mnt/m3tal_data`).
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. A default will be generated if left empty.
*   **`ADMIN_PASSWORD`**: The default password for the dashboard `admin` user. This can be changed later.
*   Other prompts will configure paths and internal network settings. Defaults are generally suitable for initial setup.

## Step 4: Start the Routing Stack

M3TAL uses Docker Compose to manage its services. The `m3tal up` command starts all `*-compose.yml` files located in `/docker/`. This includes the core routing stack (Traefik gateway and optional Cloudflared tunnel).

```bash
m3tal up
```

This command initiates the `routing-compose.yml` stack, deploying Traefik as a reverse proxy container. Traefik listens on port 80 and manages routing for other services.

## Step 5: Start the M3TAL Dashboard

Deploy the M3TAL Dashboard container. The `m3tal dash up` command handles pulling the dashboard image and starting the container with the appropriate configuration.

```bash
m3tal dash up
```

This command will:
1.  Download the latest dashboard compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
2.  Read `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
3.  Start the `m3tal-dashboard` container, applying either `m3tal-compose.local.yml` (for direct port binding) or `m3tal-compose.traefik.yml` (for Traefik integration).

## Step 6: Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard URL. The URL depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    Access via `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your M3TAL server).
    *Example: `http://192.168.1.100:8082`*

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    Access via `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in Step 3).
    *Example: `http://dash.example.com`*

## Step 7: Log In to the Dashboard

The default username for the M3TAL Dashboard is `admin`. The password is the `ADMIN_PASSWORD` you set during the configuration wizard (Step 3). If you left it empty, it defaults to `admin_pass`.

To change the dashboard password for the `admin` user:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new password. This updates the `/docker/users.json` file.

---

## Filesystem Contract

M3TAL adheres to a defined filesystem structure for its configuration and data:

| Path                        | Purpose                                                      |
| :-------------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the API daemon.       |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.     |

---

## Core M3TAL Components

M3TAL consists of several integrated components:

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port 8080. It manages Docker, the state database (`/var/lib/m3tal/state.db`), and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port 8082. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services by domain name on port 80. It uses a file provider for dynamic routing and automatically discovers services via Docker labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for secure, zero-config internet access to your services.

---

## Dashboard Access Modes

The M3TAL Dashboard supports two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

1.  **Local Mode (`DASHBOARD_EXPOSE_MODE=local`)**
    *   Uses the `m3tal-compose.local.yml` override.
    *   Adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
    *   **Access via**: `http://HOST_IP:8082` or `http://localhost:8082`.
    *   No Traefik configuration is required for dashboard access.
    *   **Best for**: LAN-only setups, first-time users, or local testing.

2.  **Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)**
    *   Uses the `m3tal-compose.traefik.yml` override.
    *   Adds Traefik labels to the dashboard container, allowing Traefik to route `dash.${DOMAIN}` to the dashboard on port 8082.
    *   **Access via**: `http://dash.DOMAIN` (Traefik must be running via `m3tal up`).
    *   **Best for**: Domain-based setups, where multiple services are exposed behind a single reverse proxy.

---

## M3TAL Runtime and Docker

M3TAL heavily leverages Docker Engine and Docker Compose V2.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files found within the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`). This allows M3TAL to manage entire Docker Compose stacks.
*   The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It downloads the necessary `m3tal-compose.yml` and override files from GitHub and starts the dashboard using the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting.
*   User-defined Docker Compose stacks for additional services should be placed as `*-compose.yml` files directly in `/docker/`.

---

## Port Overview

The following ports are used by M3TAL and its components:

| Port | Service               | Access Level                                   |
| :--- | :-------------------- | :--------------------------------------------- |
| 80   | Traefik HTTP entry    | Public (if Traefik is exposed)                 |
| 8080 | M3TAL API daemon (Go) | Host-local only                                |
| 8081 | Traefik Dashboard     | Host-local only (`127.0.0.1:8081`)             |
| 8082 | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) |

---

## Firewall Configuration

If you expose Traefik on port 80 to the internet or your local network, ensure your firewall allows incoming connections on that port.

For `ufw` (Uncomplicated Firewall):

```bash
sudo ufw allow 80
```

---

## M3TAL API Service Management

The M3TAL API daemon runs as a systemd service. Use `systemctl` for service management and `journalctl` for logging.

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View API service logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```

*   **Restart the API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

---

## Day 2 Operations: Adding New Stacks

To deploy additional Docker Compose stacks with M3TAL:

1.  **Place your compose file:** Create your Docker Compose file (e.g., `my-app-compose.yml`) in the `/docker/` directory.
    *Example: `sudo nano /docker/my-app-compose.yml`*
2.  **Configure environment variables:** Ensure any necessary environment variables for your new stack are set in `/etc/m3tal/.env`. You can use the wizard or set them directly:
    ```bash
    sudo m3tal config wizard
    # OR
    sudo m3tal config set MY_APP_VAR "some_value"
    ```
3.  **Start all stacks:** Run `m3tal up` to deploy your new stack alongside existing ones.
    ```bash
    m3tal up
    ```
    This command reads all `*-compose.yml` files in `/docker/` and applies the Docker Compose configuration.