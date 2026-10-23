# M3TAL: Getting Started

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installation by running the following commands:

```bash
docker --version
docker compose version
```

If these commands do not output version numbers, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

Install M3TAL using the provided APT repository with the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential environment variables. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of each prompt:

*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. Defaults to `5050`.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will run. Defaults to `8082`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is accessed.
    *   `local`: Exposes the dashboard directly via its port (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    *   `traefik`: Exposes the dashboard via the Traefik reverse proxy, typically using a subdomain (e.g., `http://dash.YOUR_DOMAIN`). Requires Traefik to be running.
*   **`HTTP_PORT`**: This environment variable is not directly used by the M3TAL CLI but might be referenced by other services.
*   **`STATE_DIR`**: The directory where M3TAL stores its state. Defaults to `./state` within the base storage path.
*   **`LOG_LEVEL`**: Sets the verbosity of M3TAL logs. Options include `debug`, `info`, `warn`, `error`. Defaults to `info`.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard. **Change this from the default `change_me_immediately` to a strong, unique value.**
*   **`API_TOKEN`**: A token for API authentication. **Change this from the default `change_me_api_token` to a strong, unique value.**
*   **`ADMIN_PASSWORD`**: The default password for the dashboard administrator. **Change this from `admin_pass` to a strong, unique password.**
*   **`NETWORK_NAME`**: The name of the Docker network M3TAL will use. Defaults to `m3tal`.
*   **`LOCAL_IP`**: The local IP address of your host machine. Often `127.0.0.1` or your LAN IP.
*   **`DOMAIN`**: The primary domain you intend to use for M3TAL services (e.g., `mydomain.com`). Defaults to `localhost`.
*   **`VPN_USER`**, **`VPN_PASSWORD`**: Credentials for VPN access if your setup requires it.
*   **`BASE_STORAGE_PATH`**: The root directory for M3TAL data and configuration. Defaults to `./data`.
*   **`MEDIA_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for media files.
*   **`CONFIG_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for configuration files.
*   **`DOWNLOADS_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for downloads.
*   **`PUID`**, **`PGID`**: User and group IDs for running Docker containers. Defaults to `1000`.
*   **`TZ`**: Your timezone (e.g., `America/Denver`).
*   **`TRAEFIK_WEB_PORT`**: The host port for Traefik's HTTP entrypoint. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The host port for Traefik's HTTPS entrypoint. Defaults to `443`.
*   **`TRAEFIK_DASHBOARD_PORT`**: The host port for Traefik's own dashboard. Defaults to `8080`.
*   **`DEBUG_MODE`**: Enables or disables debug logging. Defaults to `false`.
*   **`METRICS_ENABLED`**: Enables or disables metrics collection. Defaults to `true`.

## Step 4: Start the Routing Stack (Traefik)

This command uses Docker Compose to start the core routing infrastructure, primarily Traefik. It reads all `*-compose.yml` files located in the `/docker/` directory.

```bash
m3tal up
```

This action brings up services like Traefik, which acts as a reverse proxy for your M3TAL services.

## Step 5: Start the Dashboard

This command specifically pulls the M3TAL dashboard image if it's not already present and starts the dashboard container. It uses the `DASHBOARD_EXPOSE_MODE` setting from your configuration to determine how to expose the dashboard.

```bash
m3tal dash up
```

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to one of the following addresses, depending on your `DASHBOARD_EXPOSE_MODE` configuration:

*   If `DASHBOARD_EXPOSE_MODE` is `local`:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address, or use `http://localhost:8082` if running locally).

*   If `DASHBOARD_EXPOSE_MODE` is `traefik`:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured. Traefik must be running and configured correctly for this to work).

## Step 7: Log In

You will be presented with the M3TAL dashboard login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default password. You should have changed it during the wizard.)

**Important:** For security reasons, it is highly recommended to change the default administrator password immediately after your first login. You can do this via the command line:

```bash
sudo m3tal dashpass <new_password>
```

Replace `<new_password>` with your desired strong password.

## Filesystem Contract

M3TAL relies on a specific directory structure and configuration files:

| Path                      | Purpose                                                                    |
| :------------------------ | :------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | The primary environment configuration file. Edited by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's operational state. Auto-created by the API. |
| `/docker`                 | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`      | Stores dashboard user credentials. Managed by `m3tal dashpass`.             |

## Port Map

The following ports are used by M3TAL and its components:

| Port | Service                  | Access Method        |
| :--- | :----------------------- | :------------------- |
| 80   | Traefik (HTTP)           | Public (Traefik mode) |
| 8080 | M3TAL API daemon (Go)    | Host-local only      |
| 8081 | Traefik Dashboard        | Host-local only      |
| 8082 | M3TAL Dashboard          | Direct port or via Traefik |

## Firewall Note

If Traefik is exposed to the public internet on port 80, ensure your firewall allows incoming traffic on this port. For example, if using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```