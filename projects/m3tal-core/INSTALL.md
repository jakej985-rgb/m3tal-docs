# M3TAL Get Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before you begin, ensure you have the following installed on your system:

*   **Docker Engine:** The containerization platform.
*   **Docker Compose V2:** The orchestration tool.

Verify their installation by running the following commands in your terminal:

```bash
docker --version
docker compose version
```

If either command fails, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed via a dedicated APT repository for easy installation and updates. Execute the following commands to add the repository and install the `m3tal` package:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential environment variables. Run the following command to start the wizard:

```bash
sudo m3tal config wizard
```

You will be presented with a series of prompts. Here's what each one means:

*   **`DASHBOARD_EXPOSE_MODE`**: This determines how the M3TAL dashboard is made accessible.
    *   `local`: Exposes the dashboard directly via a port on your host machine (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup and LAN-only access.
    *   `traefik`: Exposes the dashboard through Traefik, allowing access via a domain name (e.g., `http://dash.DOMAIN`). This requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name for accessing M3TAL services if `DASHBOARD_EXPOSE_MODE` is set to `traefik`. Defaults to `localhost` if not specified or in `local` mode.
*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. Defaults to `5050`.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will listen internally. Defaults to `8082`.
*   **`HTTP_PORT`**: The port Traefik will use for its HTTP entry point. Defaults to `8080`.
*   **`TRAEFIK_WEB_PORT`**: The external port Traefik will listen on for HTTP traffic. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The external port Traefik will listen on for HTTPS traffic. Defaults to `443`.
*   **`PUID` / `PGID`**: The User ID and Group ID for running Docker containers. Usually your user's ID.
*   **`TZ`**: Your timezone (e.g., `America/New_York`, `Europe/London`).
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. You will be prompted to change the default.
*   **`API_TOKEN`**: An API token for authentication with the M3TAL API. You will be prompted to change the default.
*   **`ADMIN_PASSWORD`**: The password for the dashboard administrator. You will be prompted to change the default.

The wizard will save your selections to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

Traefik acts as the primary reverse proxy for M3TAL services. To start the routing stack, run:

```bash
m3tal up
```

This command orchestrates all Docker Compose files located in the `/docker/` directory, which includes the Traefik configuration.

## Step 5: Start the Dashboard

The M3TAL dashboard provides a web interface to manage your system. To start the dashboard container, run:

```bash
m3tal dash up
```

This command will:
1. Pull the latest `m3tal-dashboard` Docker image if it's not already present.
2. Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3. Start the dashboard container, applying the appropriate Docker Compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your configuration.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard URL:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local`: `http://YOUR_IP:8082` (Replace `YOUR_IP` with your server's IP address or `localhost` if on the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`: `http://dash.YOUR_DOMAIN` (Replace `YOUR_DOMAIN` with the domain you configured. Ensure Traefik is running via `m3tal up`.)

## Step 7: Log In

Upon accessing the dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default if you haven't changed it during the wizard).

**Important:** It is highly recommended to change the default administrator password for security reasons. You can do this via the command line:

```bash
sudo m3tal dashpass
```

This command will prompt you for a new password for the dashboard's administrator account.

## Filesystem Contract

M3TAL utilizes a specific filesystem structure for configuration and state management. Understanding these paths is crucial for advanced administration and troubleshooting.

| Path                    | Purpose                                                                      | Managed By       |
| :---------------------- | :--------------------------------------------------------------------------- | :--------------- |
| `/etc/m3tal/.env`       | Primary M3TAL environment configuration file. Populated by the wizard.       | `m3tal config`   |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL system state. Auto-created by the API daemon. | `m3tal-api`      |
| `/opt/m3tal/stack/`     | Canonical directory containing M3TAL's core Docker Compose files and Traefik config. | M3TAL Installation |
| `/docker`               | **Symlink** pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations (e.g., `m3tal up`). | M3TAL Installation |
| `/docker/users.json`    | Stores dashboard user credentials.                                           | `m3tal dashpass` |

## Port Map

The following ports are used by M3TAL components:

| Port     | Service             | Access Type                 |
| :------- | :------------------ | :-------------------------- |
| 80       | Traefik (HTTP)      | Public (when exposed)       |
| 8080     | M3TAL API Daemon    | Host-local                  |
| 8081     | Traefik Dashboard   | Host-local only             |
| 8082     | M3TAL Dashboard     | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If you are exposing Traefik to the public internet and not using a cloud-based firewall, ensure that port 80 (and 443 if HTTPS is configured) is open in your host's firewall. For example, if you are using `ufw`:

```bash
sudo ufw allow 80
# If using HTTPS:
# sudo ufw allow 443
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View service logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```