# M3TAL - Getting Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running:

```bash
docker --version
docker compose version
```

## Step 2: Install M3TAL via APT

Install M3TAL using the provided APT repository. Execute the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up your environment:

```bash
sudo m3tal config wizard
```

This command will guide you through several prompts. Here's what each prompt signifies:

*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. The default is `8082`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via its port (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup on a local network.
    *   `traefik`: Exposes the dashboard through the Traefik reverse proxy. This requires Traefik to be running and a domain configured.
*   **`HTTP_PORT`**: The port for the M3TAL API daemon. The default is `8080`.
*   **`STATE_DIR`**: The directory where M3TAL stores its state information. The default is `./state` within the M3TAL configuration directory.
*   **`LOG_LEVEL`**: Sets the verbosity of M3TAL logs. Options include `debug`, `info`, `warn`, `error`.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. **Change this from the default `change_me_immediately`**.
*   **`API_TOKEN`**: An API token for authentication. **Change this from the default `change_me_api_token`**.
*   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard. **Change this from the default `admin_pass`**.
*   **`NETWORK_NAME`**: The name of the Docker network M3TAL services will use. The default is `m3tal`.
*   **`LOCAL_IP`**: The IP address of your host machine. Defaults to `127.0.0.1`.
*   **`DOMAIN`**: The domain name you intend to use for M3TAL services when `DASHBOARD_EXPOSE_MODE=traefik`. Defaults to `localhost`.
*   **`VPN_USER`**: Username for VPN connection (if configured).
*   **`VPN_PASSWORD`**: Password for VPN connection (if configured).
*   **`BASE_STORAGE_PATH`**: The base directory for M3TAL data storage. Defaults to `./data`.
*   **`MEDIA_PATH`**: The directory for media files.
*   **`CONFIG_PATH`**: The directory for M3TAL configuration files.
*   **`DOWNLOADS_PATH`**: The directory for downloads.
*   **`PUID`**: The user ID for running Docker containers. Defaults to `1000`.
*   **`PGID`**: The group ID for running Docker containers. Defaults to `1000`.
*   **`TZ`**: Your timezone. Defaults to `America/Denver`.
*   **`TRAEFIK_WEB_PORT`**: The port Traefik listens on for HTTP traffic. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The port Traefik listens on for HTTPS traffic. Defaults to `443`.
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik's own dashboard is accessible on. Defaults to `8080`.
*   **`DEBUG_MODE`**: Enable or disable debug mode for M3TAL.
*   **`METRICS_ENABLED`**: Enable or disable metrics collection.

## Step 4: Start the Routing Stack (Traefik)

This command starts the core M3TAL routing stack, primarily Traefik, which acts as a reverse proxy for your services.

```bash
m3tal up
```

This command orchestrates all Docker Compose files found in the `/docker/` directory.

## Step 5: Start the Dashboard

Next, start the M3TAL dashboard container.

```bash
m3tal dash up
```

This command pulls the latest M3TAL dashboard image from the container registry and starts the dashboard container based on your `DASHBOARD_EXPOSE_MODE` configuration.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address.

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), access it at:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your machine's IP address or use `http://localhost:8082` if accessing from the same machine.)

*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, and Traefik is configured with a domain, access it at:
    `http://dash.YOUR_DOMAIN`
    (Replace `YOUR_DOMAIN` with the domain you configured.)

## Step 7: Log In

You will be presented with a login screen.

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard.

To change the dashboard password after initial setup, use the following command:

```bash
sudo m3tal dashpass
```

## Filesystem Contract

M3TAL relies on a specific directory structure and file locations:

| Path                     | Purpose                                                              |
| :----------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary M3TAL configuration file. Managed by `m3tal config wizard`.  |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL stack files, including compose files and Traefik configuration. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for interacting with Docker Compose stacks. |
| `/docker/users.json`     | Stores dashboard credentials. Managed by `m3tal dashpass`.           |

## Port Map

The following ports are used by M3TAL and its components:

| Port | Service/Component       | Access                                        |
| :--- | :---------------------- | :-------------------------------------------- |
| 80   | Traefik (HTTP Entrypoint)| Public (when `DASHBOARD_EXPOSE_MODE=traefik`) |
| 8080 | M3TAL API Daemon (Go)   | Host-local                                    |
| 8081 | Traefik Dashboard       | Host-local only                               |
| 8082 | M3TAL Dashboard         | Direct port (when `local`) or via Traefik (when `traefik`) |

## Firewall Note

If you are exposing Traefik to the public internet (e.g., `DASHBOARD_EXPOSE_MODE=traefik` and you want your services accessible from outside your local network), ensure that port `80` (HTTP) is allowed through your firewall. If you are using `ufw`, run:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```