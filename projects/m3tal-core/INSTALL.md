# Getting Started with M3TAL

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before you begin, ensure you have the following software installed on your system:

*   **Docker Engine**: The core containerization platform.
*   **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

To verify your installation, run the following commands:

```bash
docker --version
docker compose version
```

If either command fails, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed via a dedicated APT repository. Use the following commands to add the repository and install the `m3tal` package:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The `m3tal config wizard` command will guide you through setting up essential configuration options.

```bash
sudo m3tal config wizard
```

You will be presented with a series of prompts. Here's an explanation of each:

*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL dashboard is accessible.
    *   `local`: Exposes the dashboard directly via a host port (default: 8082). Recommended for initial setup and LAN-only access.
    *   `traefik`: Exposes the dashboard via Traefik using a domain name. Requires Traefik to be running.
*   **`DOMAIN`**: The base domain name for accessing services if `DASHBOARD_EXPOSE_MODE` is set to `traefik`. Defaults to `localhost`.
*   **`DASHBOARD_PORT`**: The port on which the dashboard will listen. Defaults to `8082`.
*   **`HTTP_PORT`**: The port for the M3TAL API daemon. Defaults to `8080`.
*   **`PUID` / `PGID`**: The User ID (UID) and Group ID (GID) to run containers with. These typically match your user's UID/GID for file permissions. You can find your UID/GID by running `id -u` and `id -g`.
*   **`TZ`**: Your local timezone (e.g., `America/New_York`). This ensures accurate logging and scheduling.
*   **`STATE_DIR`**: The directory where M3TAL stores its state, including the database. Defaults to `./state` within the configuration path.
*   **`BASE_STORAGE_PATH`**: The base directory for storing data and configurations for various M3TAL services. Defaults to `./data`.
*   **`CONFIG_PATH`**: The directory where M3TAL stores its configuration files. Defaults to `./data/config`.
*   **`MEDIA_PATH`**: The directory for media files.
*   **`DOWNLOADS_PATH`**: The directory for downloads.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for security. A default value is provided; it is highly recommended to change this to a unique, strong secret.
*   **`API_TOKEN`**: An API token for authenticating with the M3TAL API. A default value is provided; change this for security.
*   **`ADMIN_PASSWORD`**: The password for the M3TAL dashboard admin user. A default value is provided; change this immediately.

After completing the wizard, the configuration will be saved to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

The Traefik reverse proxy handles incoming traffic and routes it to the appropriate services. To start Traefik and other core routing components, run:

```bash
m3tal up
```

This command will orchestrate all Docker Compose files found in the `/docker/` directory, including the configuration for Traefik.

## Step 5: Start the Dashboard

The M3TAL dashboard provides a web interface for managing your services. To start the dashboard container:

```bash
m3tal dash up
```

This command will pull the latest `m3tal-dashboard` image if it's not already present and start the container according to your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's URL:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local`:
    `http://YOUR_IP:8082` (Replace `YOUR_IP` with your server's IP address, or use `http://localhost:8082` if accessing from the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`:
    `http://dash.DOMAIN` (Replace `DOMAIN` with the domain you configured, e.g., `http://dash.localhost`).

## Step 7: Log In

Upon accessing the dashboard for the first time, you will be prompted to log in.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: The value you set for `ADMIN_PASSWORD` during the wizard (or the default `admin_pass` if you didn't change it).

**Important**: For security reasons, immediately change the default password after your first login. You can also change the password from the command line using:

```bash
sudo m3tal dashpass
```

This command will prompt you to enter a new password for the dashboard.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure to manage its configuration and state.

| Path                   | Purpose                                                                            |
| :--------------------- | :--------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`      | The primary M3TAL configuration file. Modified by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | The SQLite database storing M3TAL's operational state. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`    | The canonical directory containing M3TAL's core Docker Compose files and Traefik configuration. |
| `/docker`              | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing Docker Compose files for all stacks. |
| `/docker/users.json`   | Stores dashboard user credentials. Managed by `m3tal dashpass`.                       |

## Port Map

The following ports are used by M3TAL and its components:

| Port   | Service             | Access                                           |
| :----- | :------------------ | :----------------------------------------------- |
| 80     | Traefik             | Public entry point (when exposed via Traefik)    |
| 8080   | M3TAL API Daemon    | Host-local access                                |
| 8081   | Traefik Dashboard   | Host-local access only                           |
| 8082   | M3TAL Dashboard     | Direct port access (local mode) or via Traefik (traefik mode) |

## Firewall Note

If Traefik is exposed to the public internet (e.g., `DASHBOARD_EXPOSE_MODE=traefik` and you intend to access services from outside your local network), ensure that port 80 is allowed through your firewall. If you are using `ufw`, run:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs (real-time)**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```