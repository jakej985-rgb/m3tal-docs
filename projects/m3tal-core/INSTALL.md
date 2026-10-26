```markdown
# GET_STARTED.md - M3TAL Ecosystem Setup Guide

This document provides a step-by-step guide for first-time users to set up and run the M3TAL ecosystem.

---

## 1. Prerequisites

Before installing M3TAL, ensure that Docker Engine and Docker Compose V2 are installed on your system.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Expected output will show Docker Engine and Docker Compose V2 versions.

## 2. Install M3TAL via APT

M3TAL is distributed as a single Go binary (`m3tal`) via an APT repository. Execute the following commands to install it:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the M3TAL configuration wizard. This interactive tool will guide you through setting essential environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

During the wizard, you will be prompted for various settings. Here are some key prompts and their explanations:

*   **`DOMAIN`**: The primary domain name for your M3TAL services (e.g., `example.com`). If you plan to expose services via Traefik (Mode 2), this is crucial. For local-only setups, `localhost` is acceptable.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard will be accessible.
    *   `local`: Exposes the dashboard directly on a host port (`8082` by default). Recommended for initial setup and LAN-only access.
    *   `traefik`: Exposes the dashboard via the Traefik reverse proxy (e.g., `dash.YOUR_DOMAIN`). Requires Traefik to be running.
*   **`PUID` / `PGID`**: The User ID (UID) and Group ID (GID) that Docker containers will use to access host filesystem resources. Defaults to your current user's UID/GID for optimal permissions.
*   **`TZ`**: Your local timezone (e.g., `America/Denver`). This ensures logs and container timestamps are accurate.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for session management. It's automatically generated if not provided, but you can set a custom one.
*   **`API_TOKEN`**: An authentication token for the M3TAL API. Automatically generated.
*   **`ADMIN_PASSWORD`**: The default password for the M3TAL Dashboard's `admin` user. **Change this immediately after setup.**

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. This includes the `routing-compose.yml` file, which deploys the Traefik reverse proxy and optional Cloudflared tunnel.

```bash
m3tal up
```

This command uses Docker Compose V2 to bring up all services defined in your stack files.

## 5. Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It performs the following actions:

1.  Downloads the latest dashboard Docker Compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from your `/etc/m3tal/.env` file.
3.  Starts the `m3tal-dashboard` container using the appropriate compose override file based on the `DASHBOARD_EXPOSE_MODE`. This ensures the dashboard is configured correctly for your chosen access method.

```bash
m3tal dash up
```

## 6. Access the Dashboard

How you access the M3TAL Dashboard depends on the `DASHBOARD_EXPOSE_MODE` configured in Step 3:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    Open your web browser and navigate to `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your M3TAL host) or `http://localhost:8082` if accessing from the host itself.

*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    Ensure Traefik is running (`m3tal up` was executed in Step 4). Open your web browser and navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in Step 3).

## 7. Log In to the Dashboard

The default login credentials for the dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the `m3tal config wizard` in Step 3. If you skipped this, it defaults to `admin_pass`.

**It is highly recommended to change the default password immediately.** You can do this using the `m3tal dashpass` command:

```bash
sudo m3tal dashpass
```
This command will prompt you to set a new password for the `admin` user.

---

## Filesystem Contract

M3TAL adheres to a defined filesystem structure for its configuration, state, and Docker Compose files.

| Path                     | Purpose                                                                                                                                                                                                      |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for M3TAL. Contains environment variables used by the API and Docker Compose stacks. Managed by `m3tal config wizard` and `m3tal config set`.                                       |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the M3TAL API daemon. Stores internal state, service information, and other operational data.                                                               |
| `/opt/m3tal/stack/`      | The canonical directory containing M3TAL's core Docker Compose files (e.g., `routing-compose.yml`) and Traefik dynamic configuration.                                                                           |
| `/docker`                | **Symlink to `/opt/m3tal/stack/`**. This is the user-facing path where all Docker Compose files (`*-compose.yml`) for M3TAL's core components and any user-added stacks should reside.                         |
| `/docker/users.json`     | Credential store for the M3TAL Dashboard. Contains hashed user passwords. This file is managed by the `m3tal dashpass` command.                                                                               |

## Port Table

The following table outlines the default network ports used by M3TAL components:

| Port | Service             | Access Description                                                                                             |
| :--- | :------------------ | :------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP        | Publicly exposed HTTP entry point for domain-based routing (e.g., `dash.YOUR_DOMAIN`).                         |
| 443  | Traefik HTTPS       | Publicly exposed HTTPS entry point for domain-based routing.                                                   |
| 8080 | M3TAL API Daemon    | Host-local API endpoint for the M3TAL Go daemon. Typically accessed by the dashboard or Traefik internally.    |
| 8081 | Traefik Dashboard   | Host-local access to the Traefik management dashboard (e.g., `http://127.0.0.1:8081`).                          |
| 8082 | M3TAL Dashboard     | Direct port access to the M3TAL Dashboard when `DASHBOARD_EXPOSE_MODE=local`. Otherwise routed via Traefik. |

## Firewall Note

If your M3TAL host has a firewall (e.g., UFW) enabled, you may need to open ports to allow external access.

*   If you are using **Traefik (Mode 2)** and want to expose services to the internet, you must allow HTTP (and potentially HTTPS):
    ```bash
    sudo ufw allow 80/tcp
    # sudo ufw allow 443/tcp # If using HTTPS with Traefik
    ```
*   If you are using **local dashboard access (Mode 1)** and want to access it from another device on your local network:
    ```bash
    sudo ufw allow 8082/tcp
    ```
Remember to always review your firewall rules to align with your security requirements.

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using standard `systemctl` and `journalctl` commands:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the M3TAL API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

---

## M3TAL Docker / Compose Runtime Explained

M3TAL leverages **Docker Engine** and **Docker Compose V2** as hard dependencies. Understanding how `m3tal` interacts with Docker is crucial for effective management:

*   **`m3tal up`**: This command acts as a wrapper around `docker compose up -d` for all Docker Compose files (e.g., `*-compose.yml`) found within the `/docker/` directory. By default, this directory (which is a symlink to `/opt/m3tal/stack/`) contains `routing-compose.yml` for Traefik and `m3tal-compose.yml` for the API. When you add your own `my-stack-compose.yml` to `/docker/`, `m3tal up` will include it.
*   **`m3tal dash up`**: This command is specialized for the M3TAL Dashboard. It first ensures that the latest `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) are present. It then reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env` and dynamically selects the correct override file to start the dashboard container with either direct port exposure (`local` mode) or Traefik labels (`traefik` mode).

## Dashboard Access Modes (Detailed)

The `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env` determines how the dashboard is made available:

### Mode 1: `local` (Default)
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file, which adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the `m3tal-dashboard` container.
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik required. The port is directly exposed on the host.
*   **Best for**: Initial setup, local area network (LAN) access, or when you don't require domain-based routing.

### Mode 2: `traefik`
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the `m3tal-dashboard` container. These labels instruct Traefik to route requests for `dash.${DOMAIN}` to the dashboard container's internal port `8082`.
*   **Access**: `http://dash.YOUR_DOMAIN`.
*   **Requirements**: Traefik must be running (via `m3tal up`). Your `DOMAIN` variable must be correctly configured.
*   **Best for**: Domain-based access, integrating the dashboard into a larger reverse proxy setup, and utilizing Traefik's features like TLS termination (not covered in this basic guide).

## Adding User Stacks (Day 2 Operations)

M3TAL is designed to easily integrate your own Docker Compose stacks:

1.  **Place your compose file**: Create a `*.yml` file (e.g., `my-app-compose.yml`) containing your Docker Compose service definitions and place it in the `/docker/` directory.
2.  **Configure environment variables**: If your stack requires specific environment variables, add them to `/etc/m3tal/.env` using `sudo m3tal config set KEY value` or by running `sudo m3tal config wizard` again.
3.  **Start all stacks**: Run `m3tal up` to deploy your new stack alongside M3TAL's core components. M3TAL automatically discovers and manages all `*-compose.yml` files in `/docker/`.
```