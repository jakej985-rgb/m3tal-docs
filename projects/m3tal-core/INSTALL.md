```markdown
# M3TAL Ecosystem: Getting Started Guide

This document provides a step-by-step guide for first-time M3TAL users to set up and configure the ecosystem.

## Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.

Verify their installation and version:

```bash
docker --version && docker compose version
```

## Step 2: Install M3TAL via APT

Install the M3TAL CLI binary and API daemon using the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

Initialize the M3TAL configuration and set essential variables. This creates the `/etc/m3tal/.env` file.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values:

*   **`DOMAIN`**: The base domain name for your services (e.g., `example.com`). If you don't have a domain or are running locally, `localhost` is the default. This is used by Traefik for routing.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard will be directly accessible via `http://YOUR_IP:8082`. This is recommended for local or LAN-only setups and first-time users.
    *   `traefik`: The dashboard will be accessible via Traefik at `http://dash.YOUR_DOMAIN`. This requires Traefik to be running and a configured `DOMAIN`.
*   **`PUID` (User ID) / `PGID` (Group ID)**: The User and Group ID that containers will run as. Use `id -u YOUR_USERNAME` and `id -g YOUR_USERNAME` to find your user's IDs. Typically `1000` for the first non-root user.
*   **`TZ` (Timezone)**: Your server's timezone (e.g., `America/Denver`).
*   **`API_TOKEN`**: A secret token used to secure the M3TAL API. A strong, random string is recommended.
*   **`DASHBOARD_SECRET`**: A secret key used by the M3TAL Dashboard for session management. A strong, random string is recommended.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user on the M3TAL Dashboard. Change this immediately after setup.

## Step 4: Start the Routing Stack

Start the core Docker Compose services, including the Traefik reverse proxy. This command executes `docker compose up` against all `*-compose.yml` files found within the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).

```bash
m3tal up
```

This will start the Traefik gateway (from `routing-compose.yml`), which listens on port 80 (HTTP) for incoming requests and routes them to your Docker services based on their configurations.

## Step 5: Start the M3TAL Dashboard

Start the M3TAL Dashboard container. This command fetches the latest dashboard compose files from GitHub, reads your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard container with the appropriate configuration.

```bash
m3tal dash up
```

This will pull the `ghcr.io/jakej985-rgb/m3tal-godash:debug` image and start the `m3tal-dashboard` container.

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard using the address determined by your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    Access the dashboard via `http://YOUR_SERVER_IP:8082` or `http://localhost:8082`.
*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    Access the dashboard via `http://dash.YOUR_DOMAIN` (e.g., `http://dash.example.com`). This requires Traefik to be running (as started in Step 4) and proper DNS configuration for `dash.YOUR_DOMAIN` pointing to your server's IP.

## Step 7: Log In to the Dashboard

Use the following credentials to log in:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the `m3tal config wizard` in Step 3.

**Important**: Change the `ADMIN_PASSWORD` immediately after your first login for security purposes. You can do this using the CLI:

```bash
sudo m3tal dashpass
```

This command will prompt you to set a new password for the `admin` user.

---

## Filesystem Contract

M3TAL manages critical files and directories according to the following contract:

| Path                        | Purpose                                                                                |
| :-------------------------- | :------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard` and `m3tal config set`.   |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the M3TAL API daemon.               |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains base compose files and Traefik dynamic configuration. |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. Place your Docker Compose files here. |
| `/docker/users.json`        | M3TAL Dashboard credential store. Managed by `m3tal dashpass`.                         |

## Port Table

The following ports are used by M3TAL components:

| Port | Service                               | Access                                      |
| :--- | :------------------------------------ | :------------------------------------------ |
| 80   | Traefik HTTP entry point              | Public (if Traefik is exposed)              |
| 8080 | M3TAL API daemon (Go)                 | Host-local only                             |
| 8081 | Traefik Dashboard (admin interface)   | Host-local only                             |
| 8082 | M3TAL Dashboard (Python/Flask)        | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If you are exposing Traefik (e.g., using `DASHBOARD_EXPOSE_MODE=traefik` or planning to expose other services via port 80), you may need to open port 80 on your system's firewall:

```bash
sudo ufw allow 80/tcp
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using `systemctl`:

*   **Check API service status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View API service logs in real-time**:
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Day 2 Operations: Deploying a New Stack

To add and manage your own Docker Compose applications with M3TAL:

1.  **Place your compose file**: Create or place your Docker Compose file (e.g., `my-app-compose.yml`) directly into the `/docker/` directory.
2.  **Configure environment variables (if needed)**: If your new stack requires specific environment variables, set them in `/etc/m3tal/.env` using `m3tal config wizard` or `m3tal config set KEY value`.
3.  **Start all stacks**: Run `m3tal up` to detect and start your new stack along with all other configured services. This command ensures all compose files in `/docker/` are brought up.
```