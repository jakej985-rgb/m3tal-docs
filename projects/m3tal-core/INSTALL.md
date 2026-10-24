```markdown
# GET_STARTED.md - M3TAL Setup Guide

This document provides a step-by-step guide for first-time users to set up the M3TAL Ecosystem.

---

## 1. Prerequisites

Before installing M3TAL, ensure your system meets the following requirements:

*   **Docker Engine** and **Docker Compose V2** must be installed.

Verify your Docker installation by running:

```bash
docker --version && docker compose version
```

Example output:

```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.0-desktop.1
```

## 2. Install M3TAL via APT

Install the M3TAL CLI binary and API daemon using the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the configuration wizard to set up your M3TAL environment variables. This creates the primary configuration file at `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several details:

*   **`DOMAIN`**: The base domain name for your M3TAL services (e.g., `example.com`). This is used for Traefik routing. Defaults to `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly accessible on a port (e.g., `http://YOUR_IP:8082`). No Traefik configuration is required for the dashboard itself. Best for local testing or LAN-only access.
    *   `traefik`: The dashboard is exposed via Traefik (e.g., `http://dash.DOMAIN`). This requires Traefik to be running. Best for domain-based setups.
*   **`PUID` (User ID) and `PGID` (Group ID)**: The user and group IDs that Docker containers will run as. Typically, these correspond to your primary user (`id -u` and `id -g`).
*   **`TZ`**: Your system's timezone (e.g., `America/Denver`). This ensures correct time synchronization within containers.
*   Other prompts may appear for specific environment variables. Provide values as appropriate for your setup.

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`). This includes the Traefik reverse proxy, which provides HTTP routing for your services.

```bash
m3tal up
```

This command downloads necessary Docker images (such as Traefik) and starts the associated containers. Traefik will begin listening on port 80 (HTTP) by default.

## 5. Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It pulls the dashboard Docker image and starts the container according to your `DASHBOARD_EXPOSE_MODE` setting.

```bash
m3tal dash up
```

This command will download the `m3tal-dashboard` image (e.g., `ghcr.io/jakej985-rgb/m3tal-godash:debug`) and start its container.

## 6. Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` setting:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local`:
    `http://YOUR_SERVER_IP:8082` (or `http://localhost:8082` if accessing from the server itself)
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`:
    `http://dash.YOUR_DOMAIN` (e.g., `http://dash.example.com`). This requires Traefik to be running and correctly configured via `m3tal up`.

## 7. Log In to the Dashboard

The default login credentials for the dashboard are:

*   **Username**: `admin`
*   **Password**: `admin_pass`

**Immediately change the default password.** You can do this using the CLI:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new password for the `admin` user.

---

## M3TAL Filesystem Contract

M3TAL establishes a specific filesystem structure for its operation and configuration:

| Path                        | Purpose                                                                                                 |
| :-------------------------- | :------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`           | Primary configuration file. Contains environment variables for M3TAL components. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Stores M3TAL's internal state. Auto-created by the M3TAL API daemon.             |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files (`*-compose.yml`) and Traefik configuration.   |
| `/docker`                   | **User-facing symlink** to `/opt/m3tal/stack/`. This is where users interact with stack operations.      |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                                                |

## M3TAL Port Map

M3TAL utilizes specific ports for its components and services:

| Port | Service               | Access                                                 |
| :--- | :-------------------- | :----------------------------------------------------- |
| 80   | Traefik HTTP entry point | Public (if Traefik is exposed, `TRAEFIK_WEB_PORT`)     |
| 8080 | M3TAL API daemon (Go) | Host-local only (internal communication)               |
| 8081 | Traefik dashboard     | Host-local only (`127.0.0.1:8081:8080` in Traefik config) |
| 8082 | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) |

### Firewall Note

If you expose Traefik on port 80 to the internet or your local network, you must allow incoming connections through your firewall. For `ufw`, use:

```bash
sudo ufw allow 80
```

## M3TAL Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage its state and view logs using `systemctl` and `journalctl`:

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## M3TAL Architecture Overview

M3TAL orchestrates Docker-based services. Key components include:

*   **CLI Binary (`/usr/bin/m3tal`)**: The unified command-line tool for all M3TAL operations.
*   **API Daemon (`m3tal-api.service`)**: A Go binary running as a systemd service on port 8080. It manages Docker, the internal state database (`/var/lib/m3tal/state.db`), and provides API routes for the dashboard and other tools.
*   **Dashboard Container (`m3tal-dashboard`)**: A Python/Flask container that provides a web-based UI. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`)**: A reverse proxy container that exposes services by domain name on port 80. It automatically discovers services via Docker labels and loads dynamic configurations.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container for secure, zero-config internet access to your services.

## Dashboard Access Modes

The M3TAL Dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Mode 1: `local` (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: M3TAL uses an override file (`m3tal-compose.local.yml`) that adds a direct port binding to the dashboard container (e.g., `8082:8082`).
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requirements**: No Traefik is needed for dashboard access in this mode.
*   **Use Case**: Ideal for LAN-only setups, first-time users, or local testing where direct IP/port access is sufficient.

### Mode 2: `traefik`

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: M3TAL uses an override file (`m3tal-compose.traefik.yml`) that applies Traefik labels to the dashboard container. Traefik then routes requests for `dash.${DOMAIN}` to the dashboard container on its internal port 8082.
*   **Access**: `http://dash.YOUR_DOMAIN` (e.g., `http://dash.example.com`)
*   **Requirements**: Traefik must be running via `m3tal up`. DNS records must point `dash.YOUR_DOMAIN` to your server's IP.
*   **Use Case**: Best for domain-based setups where multiple services are exposed through Traefik.

## Docker / Compose Runtime Explained

M3TAL acts as a management layer over **Docker Engine** and **Docker Compose V2**. These are hard dependencies.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files located within the `/docker/` directory. This is how entire stacks (like the routing stack, or any user-defined applications) are brought online.
*   The `m3tal dash up` command is a specialized operation for the M3TAL Dashboard. It performs the following:
    1.  Downloads the latest dashboard-related compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the appropriate compose override file based on the configured exposure mode.

User-defined Docker Compose stacks should be placed in the `/docker/` directory.

## Day 2 Operations: Installing New Stacks

To add a new application stack to your M3TAL ecosystem:

1.  Create your Docker Compose file (e.g., `my-app-compose.yml`) and place it in the `/docker/` directory.
2.  Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`. You can set them manually or using `sudo m3tal config set KEY value`.
3.  Run `m3tal up` to deploy your new stack along with all other existing stacks. Traefik will automatically pick up and route to services with appropriate Docker labels.
```