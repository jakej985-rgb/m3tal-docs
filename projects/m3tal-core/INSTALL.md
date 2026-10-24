# M3TAL Setup Guide for New Users

This guide will walk you through the initial setup of the M3TAL system.

## Step 1: Prerequisites

Before you begin, ensure that you have the following installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation with the following commands:

```bash
docker --version
docker compose version
```

If these commands do not return version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed via an APT repository. Use the following commands to add the repository and install the `m3tal` package:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The `m3tal config wizard` command will guide you through setting up essential configuration values for M3TAL.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's what each prompt means:

*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. The default is `5050`.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. The default is `8082`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via `DASHBOARD_PORT` (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    *   `traefik`: Exposes the dashboard through the Traefik reverse proxy, typically on a subdomain (e.g., `http://dash.YOUR_DOMAIN`). This requires Traefik to be running.
*   **`HTTP_PORT`**: The port for the M3TAL API daemon.
*   **`STATE_DIR`**: The directory where M3TAL will store its state data. The default is `./state` relative to the current directory or a default location managed by the system.
*   **`LOG_LEVEL`**: The verbosity of logging. Options typically include `debug`, `info`, `warn`, `error`.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard. **Change this from the default `change_me_immediately`**.
*   **`API_TOKEN`**: A token for API authentication. **Change this from the default `change_me_api_token`**.
*   **`ADMIN_PASSWORD`**: The password for the dashboard's administrative user. **Change this from the default `admin_pass`**.
*   **`NETWORK_NAME`**: The name of the Docker network used by M3TAL. The default is `m3tal`.
*   **`LOCAL_IP`**: The local IP address of your machine. The default is `127.0.0.1`.
*   **`DOMAIN`**: Your primary domain name. Used for Traefik routing if `DASHBOARD_EXPOSE_MODE` is set to `traefik`. The default is `localhost`.
*   **`VPN_USER`**: Username for VPN configuration (if applicable).
*   **`VPN_PASSWORD`**: Password for VPN configuration (if applicable).
*   **`BASE_STORAGE_PATH`**: The base directory for M3TAL storage.
*   **`MEDIA_PATH`**: Directory for media files.
*   **`CONFIG_PATH`**: Directory for configuration files.
*   **`DOWNLOADS_PATH`**: Directory for downloads.
*   **`PUID`**: The user ID to run containers as.
*   **`PGID`**: The group ID to run containers as.
*   **`TZ`**: Your timezone (e.g., `America/Denver`).
*   **`TRAEFIK_WEB_PORT`**: The host port Traefik listens on for HTTP. Default `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The host port Traefik listens on for HTTPS. Default `443`.
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik's own dashboard is accessible on. Default `8080`.
*   **`DEBUG_MODE`**: Enable or disable debug mode.
*   **`METRICS_ENABLED`**: Enable or disable metrics collection.

After completing the wizard, the configuration will be saved to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

This command starts the core routing infrastructure, primarily Traefik, which acts as a reverse proxy for your M3TAL services.

```bash
m3tal up
```

This command will execute `docker compose` commands across all `*-compose.yml` files found in the `/docker/` directory. This typically includes the `routing-compose.yml` file which sets up Traefik.

## Step 5: Start the Dashboard

This command specifically pulls the latest M3TAL dashboard image and starts the dashboard container.

```bash
m3tal dash up
```

This command will:
1. Download the appropriate `m3tal-compose.yml` and its associated override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.
2. Start the dashboard container using Docker Compose.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard URL.

*   If you used `DASHBOARD_EXPOSE_MODE=local` (the default), access the dashboard at:
    `http://YOUR_IP:8082`
    Replace `YOUR_IP` with the IP address of the machine where M3TAL is installed. You can also use `http://localhost:8082` if accessing from the same machine.

*   If you configured `DASHBOARD_EXPOSE_MODE=traefik` and have set a `DOMAIN` in your `.env` file, access the dashboard at:
    `http://dash.YOUR_DOMAIN`
    (Ensure Traefik is running and your DNS is configured correctly for this to work).

## Step 7: Log In

When you access the dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (or whatever you set during the wizard).

**It is highly recommended to change the default administrative password immediately after your first login.**

You can change the dashboard password using the following command:

```bash
sudo m3tal dashpass
```

This command will prompt you for a new password and update the `users.json` file.

---

## Filesystem Contract

M3TAL relies on a specific directory structure and configuration files:

| Path                    | Purpose                                                                   | Managed By          |
| :---------------------- | :------------------------------------------------------------------------ | :------------------ |
| `/etc/m3tal/.env`       | Primary configuration file. Stores all environment variables.             | `m3tal config wizard` |
| `/var/lib/m3tal/state.db` | SQLite state database for M3TAL. Stores operational data.               | M3TAL API daemon    |
| `/docker`               | Symlink → `/opt/m3tal/stack/`. This is the primary directory for compose files and related configurations. | System              |
| `/docker/users.json`    | Stores dashboard user credentials.                                        | `m3tal dashpass`    |

---

## Port Map

The following ports are used by M3TAL services:

| Port   | Service             | Access Method                                   |
| :----- | :------------------ | :---------------------------------------------- |
| `80`   | Traefik (HTTP)      | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)     |
| `8080` | M3TAL API daemon    | Host-local                                      |
| `8081` | Traefik Dashboard   | Host-local only (`http://localhost:8081`)       |
| `8082` | M3TAL Dashboard     | Direct port (`http://YOUR_IP:8082`) or via Traefik |

---

## Firewall Note

If Traefik is exposed to the public internet (e.g., running on a server), ensure that port `80` (for HTTP) is allowed through your firewall.

Example using `ufw`:

```bash
sudo ufw allow 80
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View the logs of the M3TAL API service in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```

You can also use `systemctl restart m3tal-api` to restart the service if needed.