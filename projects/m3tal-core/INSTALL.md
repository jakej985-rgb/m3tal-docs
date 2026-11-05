```markdown
# M3TAL GET STARTED GUIDE

This guide will walk you through the initial setup and configuration of the M3TAL ecosystem.

## 1. Prerequisites

Before proceeding, ensure that the following software is installed on your system:

*   **Docker Engine:** The core containerization platform.
*   **Docker Compose V2:** The tool for defining and running multi-container Docker applications.

Verify your installations by running the following commands:

```bash
docker --version
docker compose version
```

If these commands do not output version information, please refer to the official Docker documentation for installation instructions.

## 2. Install M3TAL via APT

Install the M3TAL CLI binary using the provided APT repository. Execute the following three commands in sequence:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential parameters. Run the command:

```bash
sudo m3tal config wizard
```

You will be presented with a series of prompts. Here's an explanation of each:

*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. (Default: `5050`)
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible locally or via direct port binding. (Default: `8082`)
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via `DASHBOARD_PORT` (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    *   `traefik`: Exposes the dashboard through Traefik, typically accessible via a domain name (e.g., `http://dash.DOMAIN`). Requires Traefik to be running.
*   **`HTTP_PORT`**: The primary HTTP port for Traefik. (Default: `8080`)
*   **`STATE_DIR`**: The directory where M3TAL stores its state information. (Default: `./state`)
*   **`LOG_LEVEL`**: Sets the verbosity of M3TAL logs (e.g., `info`, `debug`). (Default: `info`)
*   **`DASHBOARD_SECRET`**: A secret key used for securing dashboard communication. **Change this from the default!** (Default: `change_me_immediately`)
*   **`API_TOKEN`**: A token for authenticating API requests. **Change this from the default!** (Default: `change_me_api_token`)
*   **`ADMIN_PASSWORD`**: The default password for accessing the M3TAL dashboard. **Change this!** (Default: `admin_pass`)
*   **`NETWORK_NAME`**: The name of the Docker network M3TAL uses. (Default: `m3tal`)
*   **`LOCAL_IP`**: Your server's local IP address. (Default: `127.0.0.1`)
*   **`DOMAIN`**: The primary domain for your M3TAL instance (used with Traefik). (Default: `localhost`)
*   **`VPN_USER`**: Username for VPN configuration (if applicable). (Default: `user`)
*   **`VPN_PASSWORD`**: Password for VPN configuration (if applicable). (Default: `password`)
*   **`BASE_STORAGE_PATH`**: The base directory for storing data. (Default: `./data`)
*   **`MEDIA_PATH`**: Directory for media files. (Default: `./data/media`)
*   **`CONFIG_PATH`**: Directory for configuration files. (Default: `./data/config`)
*   **`DOWNLOADS_PATH`**: Directory for downloads. (Default: `./data/downloads`)
*   **`PUID`**: User ID for container processes. (Default: `1000`)
*   **`PGID`**: Group ID for container processes. (Default: `1000`)
*   **`TZ`**: Your timezone. (Default: `America/Denver`)
*   **`TRAEFIK_WEB_PORT`**: The port Traefik listens on for HTTP. (Default: `80`)
*   **`TRAEFIK_WEBHTTPS_PORT`**: The port Traefik listens on for HTTPS. (Default: `443`)
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik's own dashboard listens on. (Default: `8080`)
*   **`DEBUG_MODE`**: Enables or disables debug logging. (Default: `false`)
*   **`METRICS_ENABLED`**: Enables or disables metrics collection. (Default: `true`)

The wizard will save your settings to `/etc/m3tal/.env`.

## 4. Start the Routing Stack (Traefik)

This command initiates the core M3TAL routing stack, primarily Traefik, which acts as a reverse proxy for your services. It processes all Docker Compose files found within the `/docker/` directory.

```bash
m3tal up
```

This command ensures that services like Traefik and any other stacks you've added are running.

## 5. Start the Dashboard

This command specifically pulls the M3TAL dashboard image if it's not already present and starts the dashboard container.

```bash
m3tal dash up
```

This command checks the `DASHBOARD_EXPOSE_MODE` setting in your `/etc/m3tal/.env` file to determine how to start the dashboard.

## 6. Access the M3TAL Dashboard

Open your web browser and navigate to one of the following addresses:

*   If `DASHBOARD_EXPOSE_MODE=local` (default):
    `http://YOUR_IP:8082`
    Replace `YOUR_IP` with your server's IP address or `localhost` if accessing from the same machine.

*   If `DASHBOARD_EXPOSE_MODE=traefik` and Traefik is running:
    `http://dash.DOMAIN`
    Replace `DOMAIN` with the value you configured during the wizard.

## 7. Log In

Use the following default credentials to log into the M3TAL dashboard:

*   **Username:** `admin`
*   **Password:** `admin_pass` (This is the default value from the wizard. It's highly recommended to change this immediately.)

To change the dashboard password, run the following command and follow the prompts:

```bash
sudo m3tal dashpass
```

## Filesystem Contract

M3TAL utilizes a specific filesystem structure to manage its configuration, state, and Docker Compose files:

| Path                      | Purpose                                                                                 |
| :------------------------ | :-------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | The primary environment file containing all M3TAL configuration variables. Modified by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's operational state. Automatically created by the API daemon. |
| `/opt/m3tal/stack/`       | The canonical directory containing M3TAL's core Docker Compose files and Traefik configuration. |
| `/docker`                 | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all M3TAL stack operations. |
| `/docker/users.json`      | Stores dashboard user credentials. Managed by `m3tal dashpass`.                       |

## Port Map

The following ports are used by M3TAL and its components:

| Port    | Service            | Access                                     |
| :------ | :----------------- | :----------------------------------------- |
| 80      | Traefik            | Public (if Traefik mode is exposed externally) |
| 8080    | M3TAL API daemon   | Host-local                                 |
| 8081    | Traefik dashboard  | Host-local only                            |
| 8082    | M3TAL Dashboard    | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If Traefik is configured to be accessible from outside your local network, ensure that port 80 is open in your firewall. For example, if you are using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
    The `-f` flag allows you to follow the log output in real-time.

*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
```