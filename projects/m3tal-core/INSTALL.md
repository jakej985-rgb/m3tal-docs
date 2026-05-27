# M3TAL Getting Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before you begin, ensure you have the following software installed on your system.

*   **Docker Engine**: The containerization platform.
*   **Docker Compose V2**: The orchestration tool for Docker.

To verify your installation, run the following commands:

```bash
docker --version
docker compose version
```

If either of these commands fails or shows an outdated version, please refer to the official Docker documentation for installation or upgrade instructions.

## Step 2: Install M3TAL via APT

M3TAL is installed using your system's Advanced Packaging Tool (APT). Execute the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

After installation, you will run the M3TAL configuration wizard to set up essential parameters.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for the following information:

*   **`DASHBOARD_PORT`**: The local port on which the M3TAL dashboard will be accessible. The default is `8082`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is made accessible.
    *   `local` (default): Exposes the dashboard directly via its port (`8082`). Ideal for LAN access and initial setup.
    *   `traefik`: Exposes the dashboard through the Traefik reverse proxy, typically via a domain name (e.g., `dash.yourdomain.com`). Requires Traefik to be running.
*   **`HTTP_PORT`**: The port for the M3TAL API daemon. The default is `8080`.
*   **`STATE_DIR`**: The directory where M3TAL stores its state. The default is `./state`.
*   **`LOG_LEVEL`**: The verbosity of M3TAL logs. Options typically include `debug`, `info`, `warn`, `error`.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for security. It is recommended to change this from the default.
*   **`API_TOKEN`**: An API token for authentication with the M3TAL API. It is recommended to change this from the default.
*   **`ADMIN_PASSWORD`**: The password for the M3TAL dashboard administrator.
*   **`NETWORK_NAME`**: The Docker network name M3TAL will use. The default is `m3tal`.
*   **`LOCAL_IP`**: Your server's local IP address. Used for internal service communication. Defaults to `127.0.0.1`.
*   **`DOMAIN`**: The primary domain name for your M3TAL setup. Defaults to `localhost`.
*   **`VPN_USER`**: Username for VPN credentials (if applicable).
*   **`VPN_PASSWORD`**: Password for VPN credentials (if applicable).
*   **`BASE_STORAGE_PATH`**: The root directory for M3TAL's persistent data. Defaults to `./data`.
*   **`MEDIA_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for media files. Defaults to `./data/media`.
*   **`CONFIG_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for configuration files. Defaults to `./data/config`.
*   **`DOWNLOADS_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for downloads. Defaults to `./data/downloads`.
*   **`PUID`**: The User ID to run containers under. Defaults to `1000`.
*   **`PGID`**: The Group ID to run containers under. Defaults to `1000`.
*   **`TZ`**: Your server's timezone. Defaults to `America/Denver`.
*   **`TRAEFIK_WEB_PORT`**: The port Traefik listens on for HTTP traffic. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The port Traefik listens on for HTTPS traffic. Defaults to `443`.
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik's internal dashboard listens on. Defaults to `8080`.
*   **`DEBUG_MODE`**: Enables or disables debug logging for M3TAL. Defaults to `false`.
*   **`METRICS_ENABLED`**: Enables or disables metrics collection. Defaults to `true`.

These settings are saved to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

This command initiates the core M3TAL services, including the Traefik reverse proxy. It orchestrates all Docker Compose files located in the `/docker/` directory.

```bash
m3tal up
```

This command ensures that Traefik is running and configured to route traffic to other M3TAL components and any other services you deploy within the M3TAL ecosystem.

## Step 5: Start the Dashboard

This command specifically manages the M3TAL dashboard container. It will pull the latest dashboard image and start the container based on your `DASHBOARD_EXPOSE_MODE` setting from the configuration wizard.

```bash
m3tal dash up
```

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard's address.

*   If `DASHBOARD_EXPOSE_MODE` is set to `local`, access the dashboard at:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address, or use `http://localhost:8082` if accessing from the same machine).

*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, and Traefik is correctly configured with a domain, access the dashboard at:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured, e.g., `http://dash.my M3TAL.com`).

## Step 7: Log In

Upon accessing the dashboard, you will be presented with a login screen.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default value for `ADMIN_PASSWORD` if not changed during the wizard. It is highly recommended to change this immediately).

After logging in, it is strongly recommended to change the default administrator password. You can do this via the command line:

```bash
sudo m3tal dashpass <new_password>
```
Replace `<new_password>` with your desired secure password.

## Filesystem Contract

M3TAL relies on a specific filesystem structure for its configuration and state. Understanding these paths is crucial for troubleshooting and manual management.

| Path                     | Purpose                                                                                 |
| :----------------------- | :-------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | The primary configuration file. Stores all environment variables set during the wizard. |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's operational state. Automatically created by the API.  |
| `/docker`                | This directory is a symbolic link pointing to `/opt/m3tal/stack/`.                     |
| `/opt/m3tal/stack/`      | The canonical location for M3TAL's Docker Compose files and Traefik configuration.    |
| `/docker/users.json`     | Stores dashboard user credentials. Managed by `m3tal dashpass`.                         |

## Port Map

The following ports are used by M3TAL services:

| Port   | Service             | Access                                             |
| :----- | :------------------ | :------------------------------------------------- |
| `80`   | Traefik (HTTP)      | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)        |
| `443`  | Traefik (HTTPS)     | Public (if configured)                             |
| `8080` | M3TAL API Daemon    | Host-local (accessible by other containers/host) |
| `8081` | Traefik Dashboard   | Host-local only                                    |
| `8082` | M3TAL Dashboard     | Direct port (if `local` mode) or via Traefik       |

## Firewall Note

If your M3TAL server is exposed to the internet and you are using Traefik (e.g., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services), ensure that port `80` (for HTTP) and `443` (for HTTPS, if configured) are open in your firewall.

For example, if you are using `ufw`:

```bash
sudo ufw allow 80
sudo ufw allow 443 # If HTTPS is configured
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```