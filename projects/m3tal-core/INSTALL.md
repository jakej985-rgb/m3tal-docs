# M3TAL - Get Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following installed:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installation by running:

```bash
docker --version && docker compose version
```

## Step 2: Install M3TAL via APT

Install M3TAL using the provided APT repository. Execute the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential parameters. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted for the following:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   `local`: Exposes the dashboard directly via a port. Recommended for initial setup or LAN-only access.
    *   `traefik`: Exposes the dashboard through the Traefik reverse proxy. Requires Traefik to be running.
*   **`DOMAIN`**: The domain name you wish to use for accessing services. Defaults to `localhost` if not set.
*   **`API_PORT`**: The port for the M3TAL API daemon. Defaults to `5050`.
*   **`DASHBOARD_PORT`**: The port for the M3TAL dashboard. Defaults to `8082`.
*   **`TRAEFIK_WEB_PORT`**: The public-facing HTTP port for Traefik. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The public-facing HTTPS port for Traefik. Defaults to `443`.
*   **`STATE_DIR`**: The directory where M3TAL stores its state. Defaults to `./state`.
*   **`BASE_STORAGE_PATH`**: The base directory for storing data, media, config, and downloads. Defaults to `./data`.
*   **`PUID`**: The User ID to run Docker containers as. Defaults to `1000`.
*   **`PGID`**: The Group ID to run Docker containers as. Defaults to `1000`.
*   **`TZ`**: Your timezone. Defaults to `America/Denver`.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. It's recommended to change the default value.
*   **`API_TOKEN`**: A token for API authentication. It's recommended to change the default value.
*   **`ADMIN_PASSWORD`**: The password for the dashboard administrator. It's recommended to change the default value.

## Step 4: Start the Routing Stack (Traefik)

This command starts the core routing stack, which includes Traefik, by orchestrating all `*-compose.yml` files found in the `/docker/` directory.

```bash
m3tal up
```

This command will:
*   Download and configure Traefik.
*   Start Traefik as a Docker container.
*   Ensure Traefik is listening on the configured ports (defaulting to 80 and 443).

## Step 5: Start the Dashboard

This command specifically pulls the necessary Docker image and starts the M3TAL dashboard container.

```bash
m3tal dash up
```

This command will:
*   Check the `DASHBOARD_EXPOSE_MODE` setting from your `/etc/m3tal/.env` file.
*   If `local`, it will start the dashboard with a direct port binding.
*   If `traefik`, it will start the dashboard with Traefik labels, allowing it to be routed by Traefik.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address.

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), access the dashboard at:
    `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or `http://localhost:8082` if accessing from the server itself).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, and you have a domain configured, access it at:
    `http://dash.DOMAIN` (replace `DOMAIN` with your configured domain, e.g., `http://dash.localhost`).

## Step 7: Log In

Upon accessing the dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default value. **It is strongly recommended to change this immediately.**)

To change the dashboard administrator password, use the following command:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new password.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure to manage its configuration and state.

| Path                      | Purpose                                                              |
| :------------------------ | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | Primary M3TAL configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database used by the API daemon. Auto-created.        |
| `/docker`                 | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`      | Stores dashboard credential information. Managed by `m3tal dashpass`. |

## Port Map

| Port | Service             | Access                                              |
| :--- | :------------------ | :-------------------------------------------------- |
| 80   | Traefik HTTP        | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)       |
| 8080 | M3TAL API Daemon    | Host-local only                                     |
| 8081 | Traefik Dashboard   | Host-local only (access via `http://127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard     | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If you have a firewall enabled and want Traefik to be accessible from outside your local network, you must allow traffic on port 80:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage it and view its logs as follows:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```