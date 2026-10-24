# M3TAL - Getting Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running these commands:

```bash
docker --version && docker compose version
```

If either of these commands fails, please refer to the official Docker documentation for installation instructions.

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

The M3TAL configuration wizard will guide you through setting up essential parameters. Run the following command:

```bash
sudo m3tal config wizard
```

You will be presented with a series of prompts:

*   **`DASHBOARD_EXPOSE_MODE`**: This setting determines how the M3TAL dashboard is accessed.
    *   `local`: Exposes the dashboard directly via a port on your host machine (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    *   `traefik`: Integrates the dashboard with Traefik, allowing access via a domain name (e.g., `http://dash.DOMAIN`). This requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name for accessing your M3TAL services. Defaults to `localhost` if not using Traefik with a custom domain.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. Defaults to `8082`.
*   **`PUID`** and **`PGID`**: The User ID and Group ID for the containers. Typically, these should match your host user's IDs for proper file permissions. You can find your user's UID by running `id -u` and GID by running `id -g`.
*   **`TZ`**: Your server's timezone, used for consistent logging and scheduling.
*   **`BASE_STORAGE_PATH`**: The base directory for M3TAL to store its data, configurations, and media.
*   **`MEDIA_PATH`**: A sub-directory within `BASE_STORAGE_PATH` for media files.
*   **`CONFIG_PATH`**: A sub-directory within `BASE_STORAGE_PATH` for configuration files.
*   **`DOWNLOADS_PATH`**: A sub-directory within `BASE_STORAGE_PATH` for downloads.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. **Change this from the default value.**
*   **`API_TOKEN`**: A token for API authentication. **Change this from the default value.**
*   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard. **Change this from the default value.**

The wizard will create or update the configuration file located at `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

The routing stack, primarily Traefik, acts as a reverse proxy and manages network traffic for your services. To start it, run:

```bash
m3tal up
```

This command uses Docker Compose to launch all services defined in the `*.yml` files found in the `/docker/` directory. This includes Traefik and any other core M3TAL services.

## Step 5: Start the Dashboard

Next, start the M3TAL dashboard container:

```bash
m3tal dash up
```

This command will:
1. Pull the latest `m3tal-dashboard` Docker image.
2. Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3. Start the dashboard container, applying the appropriate configuration override (either `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard URL:

*   If `DASHBOARD_EXPOSE_MODE` is `local`: `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's actual IP address, or `localhost` if accessing from the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is `traefik`: `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured in Step 3).

## Step 7: Log In

When prompted for credentials:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard (or the default `admin_pass` if you haven't changed it yet).

**It is strongly recommended to change the default password immediately after logging in.** You can do this using the following command:

```bash
sudo m3tal dashpass -p NEW_PASSWORD
```

Replace `NEW_PASSWORD` with your desired strong password.

## Filesystem Contract

The M3TAL ecosystem relies on a specific filesystem structure for its operation and configuration:

| Path                         | Purpose                                                                |
| :--------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | Primary M3TAL configuration file. Managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db`    | SQLite database storing M3TAL's operational state. Auto-created.     |
| `/opt/m3tal/stack/`          | The canonical directory containing M3TAL's Docker Compose files and Traefik configurations. |
| `/docker`                    | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing Docker stacks. |
| `/docker/users.json`         | Stores M3TAL dashboard user credentials. Managed by `m3tal dashpass`. |

## Port Table

The following ports are used by M3TAL and its components:

| Port | Service        | Access                                        |
| :--- | :------------- | :-------------------------------------------- |
| 80   | Traefik        | Public access for HTTP (when using Traefik)   |
| 8080 | M3TAL API      | Host-local access                             |
| 8081 | Traefik Dashboard | Host-local access only (`127.0.0.1:8081`)    |
| 8082 | M3TAL Dashboard | Direct port access (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If you intend to expose Traefik to the public internet (e.g., for `traefik` mode), ensure that port 80 is open in your firewall. If you are using `ufw` (Uncomplicated Firewall), you can allow traffic with:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon is managed by `systemd`. You can interact with it using the following commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
    (Press `Ctrl+C` to exit the log stream)
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```