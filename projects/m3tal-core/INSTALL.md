# M3TAL: Getting Started Guide

This guide provides step-by-step instructions for setting up M3TAL for the first time.

## 1. Prerequisites

Before you begin, ensure that the following software is installed on your system:

*   **Docker Engine**: The core containerization platform.
*   **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

You can verify your installation by running the following command:

```bash
docker --version && docker compose version
```

If these commands do not output version information, please refer to the official Docker documentation for installation instructions.

## 2. Install M3TAL

M3TAL is installed via APT. Run the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the M3TAL configuration wizard to set up essential parameters.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of each:

*   **`Enter your desired timezone (e.g., America/Denver):`**: Specify your local timezone. This is important for accurate logging and scheduling.
*   **`Enter the domain you wish to use (e.g., example.com or localhost):`**: This domain will be used for accessing services exposed via Traefik. For local setups, `localhost` is common.
*   **`Enter the mode for exposing the dashboard (local/traefik):`**:
    *   **`local`**: Exposes the dashboard directly via a port binding (default `8082`). This is recommended for initial setup and LAN-only access.
    *   **`traefik`**: Exposes the dashboard through the Traefik reverse proxy using the configured domain (e.g., `dash.yourdomain.com`). Requires Traefik to be running.
*   **`Enter the desired dashboard port (default 8082):`**: The port on which the M3TAL dashboard will be accessible (only relevant in `local` mode).
*   **`Enter the desired API port (default 8080):`**: The port for the M3TAL API daemon.
*   **`Enter a secret for the dashboard (e.g., generate a strong password):`**: This secret is used for securing the dashboard's internal communication. **Change this from the default!**
*   **`Enter an API token (e.g., generate a strong token):`**: This token is used for authenticating with the M3TAL API. **Change this from the default!**
*   **`Enter the admin password for the dashboard (e.g., generate a strong password):`**: The password for logging into the M3TAL dashboard. **Change this from the default!**
*   **`Enter the PUID for user mapping (default 1000):`**: The user ID for running Docker containers. Usually `1000` for the first non-root user.
*   **`Enter the PGID for group mapping (default 1000):`**: The group ID for running Docker containers. Usually `1000` for the first non-root user's group.
*   **`Enter the base path for storage (default ./data):`**: The root directory where M3TAL will store its data, configurations, and media.
*   **`Enter the path for media storage (default ./data/media):`**: A sub-directory for media files.
*   **`Enter the path for configuration storage (default ./data/config):`**: A sub-directory for configuration files.
*   **`Enter the path for downloads storage (default ./data/downloads):`**: A sub-directory for download files.

## 4. Start the Routing Stack (Traefik)

This step starts the Traefik reverse proxy, which is responsible for routing traffic to your services.

```bash
m3tal up
```

This command orchestrates the startup of all Docker Compose files located in the `/docker/` directory. This includes Traefik and any other essential M3TAL components.

## 5. Start the Dashboard

Next, start the M3TAL dashboard container.

```bash
m3tal dash up
```

This command will:
1.  Pull the latest `m3tal-dashboard` Docker image if it's not already present.
2.  Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Start the dashboard container, applying the appropriate configuration override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your chosen mode.

## 6. Access the Dashboard

Open your web browser and navigate to the dashboard's address:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address or `localhost` if you are accessing it from the same machine).

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    `http://dash.YOUR_DOMAIN`
    (Replace `YOUR_DOMAIN` with the domain you configured during the wizard, e.g., `http://dash.localhost`). Ensure Traefik is running and your domain is correctly configured for DNS resolution if accessing externally.

## 7. Log In

You will be presented with the M3TAL dashboard login screen.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: The password you set during the configuration wizard (e.g., `admin_pass` if you didn't change it).

**Important:** It is crucial to change the default admin password immediately after your first login.

### Changing the Dashboard Password

To change the dashboard password programmatically, use the following command:

```bash
sudo m3tal dashpass
```

This command will prompt you for a new password and update the dashboard's credential store.

## Filesystem Contract

M3TAL utilizes a specific filesystem layout to manage its configuration and state. Understanding these paths is key to managing your M3TAL instance:

| Path                        | Purpose                                                                                                                            |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file. This file stores all environment variables and is managed by the `m3tal config wizard` and `m3tal config set` commands. |
| `/var/lib/m3tal/state.db`   | The SQLite state database used by the M3TAL API daemon to store application state, service configurations, and other critical data. |
| `/opt/m3tal/stack/`         | The canonical directory containing M3TAL's core Docker Compose files, Traefik configurations, and system-wide stack definitions.    |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose operations managed by the `m3tal` CLI. Placing your custom Compose files here allows `m3tal up` to manage them. |
| `/docker/users.json`        | Stores the dashboard's user credentials. This file is managed by the `m3tal dashpass` command.                                   |

## Port Map

The following ports are used by M3TAL and its associated services:

| Port   | Service             | Access Method                                     |
| :----- | :------------------ | :------------------------------------------------ |
| `80`   | Traefik (HTTP)      | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)     |
| `8080` | M3TAL API daemon    | Host-local                                        |
| `8082` | M3TAL Dashboard     | Direct port (when `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (when `DASHBOARD_EXPOSE_MODE=traefik`) |

*Note: Traefik's administrative dashboard is accessible on port `8081` on the host, but only for local access.*

## Firewall Note

If you intend for Traefik to be accessible from outside your local network, you will need to open port `80` (for HTTP) in your firewall. If you are using `ufw`, the command is:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```