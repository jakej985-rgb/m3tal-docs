# Getting Started with M3TAL

This guide provides step-by-step instructions for installing and setting up the M3TAL ecosystem for the first time.

---

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system.

- **Docker Engine**: The containerization platform.
- **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

To verify your installation, run the following command in your terminal:

```bash
docker --version && docker compose version
```

If you encounter errors, please refer to the official Docker documentation for installation instructions.

---

## Step 2: Install M3TAL via APT

M3TAL is distributed via a dedicated APT repository. Use the following commands to add the repository and install the M3TAL CLI binary.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## Step 3: Run the Configuration Wizard

The `m3tal config wizard` command will guide you through the initial setup and configuration of your M3TAL environment.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of what each prompt means:

*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. This is used for internal communication between M3TAL components. (Default: `5050`)
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. (Default: `8082`)
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is made accessible.
    *   `local`: Exposes the dashboard directly via its port (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    *   `traefik`: Exposes the dashboard through Traefik, allowing access via a domain name (e.g., `http://dash.DOMAIN`). Requires Traefik to be running.
*   **`HTTP_PORT`**: The port used by Traefik for HTTP traffic. (Default: `8080` - note: Traefik itself typically uses port `80` for its entrypoint, this might refer to an internal API port or a separate service).
*   **`STATE_DIR`**: The directory where M3TAL stores its state information. This includes the `state.db` SQLite database. (Default: `./state` relative to where the command is run, but will be placed in `/var/lib/m3tal/state` by default after initial setup).
*   **`LOG_LEVEL`**: Controls the verbosity of M3TAL logs. Options include `debug`, `info`, `warn`, `error`. (Default: `info`)
*   **`DASHBOARD_SECRET`**: A secret key used for securing the M3TAL dashboard. **It is highly recommended to change this from the default.**
*   **`API_TOKEN`**: An API token for authenticating with the M3TAL API. **It is highly recommended to change this from the default.**
*   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard as an administrator. **It is highly recommended to change this from the default.**
*   **`NETWORK_NAME`**: The name of the Docker network M3TAL services will join. (Default: `m3tal`)
*   **`LOCAL_IP`**: Your server's local IP address. (Default: `127.0.0.1`)
*   **`DOMAIN`**: The domain name to use for accessing services if `traefik` mode is enabled. (Default: `localhost`)
*   **`VPN_USER`**: Username for a VPN connection (if configured). (Default: `user`)
*   **`VPN_PASSWORD`**: Password for a VPN connection (if configured). (Default: `password`)
*   **`BASE_STORAGE_PATH`**: The base directory for storing M3TAL related data, including media and downloads. (Default: `./data`)
*   **`MEDIA_PATH`**: Path for media storage within `BASE_STORAGE_PATH`. (Default: `./data/media`)
*   **`CONFIG_PATH`**: Path for configuration files within `BASE_STORAGE_PATH`. (Default: `./data/config`)
*   **`DOWNLOADS_PATH`**: Path for download storage within `BASE_STORAGE_PATH`. (Default: `./data/downloads`)
*   **`PUID` / `PGID`**: User and group IDs for running containers. (Default: `1000`)
*   **`TZ`**: Your timezone, used for consistent logging and scheduling. (Default: `America/Denver`)
*   **`TRAEFIK_WEB_PORT`**: The port Traefik listens on for HTTP traffic. (Default: `80`)
*   **`TRAEFIK_WEBHTTPS_PORT`**: The port Traefik listens on for HTTPS traffic. (Default: `443`)
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik's own dashboard is accessible on (typically `127.0.0.1:8080`). (Default: `8080`)
*   **`DEBUG_MODE`**: Enable or disable debug logging for M3TAL. (Default: `false`)
*   **`METRICS_ENABLED`**: Enable or disable metrics collection. (Default: `true`)

---

## Step 4: Start the Routing Stack (Traefik)

This command starts the core routing infrastructure, primarily Traefik, which acts as a reverse proxy for your services.

```bash
m3tal up
```

This command processes all `*-compose.yml` files found in the `/docker/` directory and starts the corresponding Docker containers. This includes Traefik and potentially other core routing components.

---

## Step 5: Start the Dashboard

The M3TAL dashboard provides a web interface for managing your M3TAL ecosystem.

```bash
m3tal dash up
```

This command will:
1.  Download the latest `m3tal-compose.yml` and relevant override files (e.g., `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting.
2.  Start the `m3tal-dashboard` Docker container.

---

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard URL. The exact URL depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    Navigate to `http://YOUR_IP:8082`
    Replace `YOUR_IP` with the IP address of your server running M3TAL. If you are accessing it from the same machine, `http://localhost:8082` may also work.

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Navigate to `http://dash.DOMAIN`
    Replace `DOMAIN` with the domain you configured during the wizard (e.g., `dash.localhost` or `dash.yourdomain.com`). Traefik must be running and configured for this to work.

---

## Step 7: Log In to the Dashboard

You will be presented with a login screen.

*   **Username:** `admin`
*   **Password:** The password you set during the configuration wizard (or `admin_pass` if you did not change it).

**To change your dashboard password after logging in:**

Use the `m3tal dashpass` command:

```bash
sudo m3tal dashpass admin NEW_PASSWORD
```
Replace `NEW_PASSWORD` with your desired new password.

---

## Filesystem Contract

M3TAL utilizes specific directories and files for its operation. Understanding this contract is crucial for maintenance and troubleshooting.

| Path                     | Purpose                                                                                                                                    |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Stores all environment variables defined during `m3tal config wizard` or manually.                             |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's operational state, including configuration details and service information. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`      | The canonical directory for M3TAL's Docker Compose files, Traefik configurations, and other stack-related assets.                             |
| `/docker`                | A symbolic link that points to `/opt/m3tal/stack/`. User-facing operations (like `m3tal up`) interact with this directory.                  |
| `/docker/users.json`     | Stores dashboard credentials. Managed by the `m3tal dashpass` command.                                                                      |

---

## Port Map

The following ports are used by M3TAL components.

| Port     | Service          | Access                                                 |
| :------- | :--------------- | :----------------------------------------------------- |
| 80       | Traefik          | Public (if `DASHBOARD_EXPOSE_MODE=traefik` and exposed) |
| 8080     | M3TAL API Daemon | Host-local                                             |
| 8081     | Traefik Dashboard| Host-local only (access via `127.0.0.1:8081`)          |
| 8082     | M3TAL Dashboard  | Direct port (`local` mode) or via Traefik (`traefik` mode) |

---

## Firewall Note

If Traefik is exposed to the public internet (e.g., running on a server accessible from outside your local network), ensure that port 80 is allowed through your firewall.

```bash
sudo ufw allow 80
```

---

## Service Management (API Daemon)

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check the status of the API daemon:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the API daemon:**
    ```bash
    journalctl -u m3tal-api -f
    ```
    Press `Ctrl+C` to exit the log stream.