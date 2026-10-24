```markdown
# M3TAL Getting Started Guide

This guide will walk you through the installation and initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

- **Docker Engine**: The core containerization platform.
- **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

Verify your installation by running the following commands:

```bash
docker --version
docker compose version
```

If these commands do not produce version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed via an APT repository for straightforward installation. Execute the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through essential setup options. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a brief explanation of each:

*   **`API_PORT`**: The port the M3TAL API daemon will listen on. (Default: `5050`)
*   **`DASHBOARD_PORT`**: The port the M3TAL dashboard will use. (Default: `8082`)
*   **`DASHBOARD_EXPOSE_MODE`**: How the dashboard is exposed.
    *   `local`: Exposes the dashboard directly on `DASHBOARD_PORT`. Best for LAN-only access or initial setup.
    *   `traefik`: Exposes the dashboard via Traefik, requiring Traefik to be running and configured with a domain.
*   **`HTTP_PORT`**: The HTTP port Traefik will listen on. (Default: `8080` for internal API, `80` for Traefik entrypoint)
*   **`STATE_DIR`**: Directory for storing M3TAL's state data. (Default: `./state`)
*   **`LOG_LEVEL`**: Sets the verbosity of M3TAL logs (`info`, `debug`, etc.). (Default: `info`)
*   **`DASHBOARD_SECRET`**: A secret key for securing the dashboard. **Change this from the default.**
*   **`API_TOKEN`**: A token for authenticating with the M3TAL API. **Change this from the default.**
*   **`ADMIN_PASSWORD`**: The password for the dashboard's administrator account.
*   **`NETWORK_NAME`**: The Docker network name M3TAL will use. (Default: `m3tal`)
*   **`LOCAL_IP`**: The local IP address of your server. (Default: `127.0.0.1`)
*   **`DOMAIN`**: The primary domain for your M3TAL setup, used by Traefik. (Default: `localhost`)
*   **`VPN_USER`**: Username for VPN integration (if applicable).
*   **`VPN_PASSWORD`**: Password for VPN integration (if applicable).
*   **`BASE_STORAGE_PATH`**: The base directory for storing data for various M3TAL services. (Default: `./data`)
*   **`MEDIA_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for media files. (Default: `./data/media`)
*   **`CONFIG_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for configuration files. (Default: `./data/config`)
*   **`DOWNLOADS_PATH`**: Subdirectory within `BASE_STORAGE_PATH` for downloads. (Default: `./data/downloads`)
*   **`PUID`**: The User ID for running containers. (Default: `1000`)
*   **`PGID`**: The Group ID for running containers. (Default: `1000`)
*   **`TZ`**: Your server's timezone (e.g., `America/New_York`). (Default: `America/Denver`)
*   **`TRAEFIK_WEB_PORT`**: The external HTTP port for Traefik. (Default: `80`)
*   **`TRAEFIK_WEBHTTPS_PORT`**: The external HTTPS port for Traefik. (Default: `443`)
*   **`TRAEFIK_DASHBOARD_PORT`**: The internal port Traefik listens on for its own dashboard. (Default: `8080`)
*   **`DEBUG_MODE`**: Enable or disable debug logging. (Default: `false`)
*   **`METRICS_ENABLED`**: Enable or disable metrics collection. (Default: `true`)

## Step 4: Start the Routing Stack (Traefik)

The `m3tal up` command orchestrates all necessary Docker Compose files located in the `/docker/` directory. This will start the core M3TAL services, including Traefik, which acts as your reverse proxy gateway.

```bash
m3tal up
```

This command will pull images and start containers defined in files like `routing-compose.yml` and others within `/docker/`.

## Step 5: Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL dashboard container. It will download the necessary configuration and start the dashboard container based on your `DASHBOARD_EXPOSE_MODE` setting from the wizard.

```bash
m3tal dash up
```

This command ensures the `m3tal-dashboard` container is running and accessible according to your chosen configuration.

## Step 6: Access the M3TAL Dashboard

You can now access the M3TAL dashboard through your web browser.

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), open your browser to:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. If accessing from the same machine, `http://localhost:8082` may also work.)

*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, and Traefik is correctly configured with a domain name, you can access it via:
    `http://dash.YOUR_DOMAIN`
    (Replace `YOUR_DOMAIN` with the domain you configured.)

## Step 7: Log In to the Dashboard

Upon first access, you will be prompted to log in.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: The password you set for `ADMIN_PASSWORD` during the `m3tal config wizard`.

**Changing Your Dashboard Password:**

To change your dashboard password after initial setup, use the following command:

```bash
sudo m3tal dashpass
```
This will prompt you to enter a new password for the `admin` user.

## Filesystem Contract

M3TAL utilizes a specific filesystem layout for configuration and state management:

| Path                      | Purpose                                                                     |
| :------------------------ | :-------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | The primary environment file containing all M3TAL configuration variables. |
| `/var/lib/m3tal/state.db` | The SQLite database used by the M3TAL API daemon to store system state.     |
| `/docker`                 | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing directory where all Docker Compose files for your stacks reside. |
| `/docker/users.json`      | Stores the dashboard user credentials, managed by `m3tal dashpass`.         |

## Port Map

The following ports are utilized by M3TAL and its components:

| Port   | Service        | Access                                   |
| :----- | :------------- | :--------------------------------------- |
| 80     | Traefik        | Public (if `DASHBOARD_EXPOSE_MODE=traefik` and exposed) |
| 8080   | M3TAL API      | Host-local                               |
| 8081   | Traefik UI     | Host-local only                          |
| 8082   | M3TAL Dashboard| Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Configuration

If your server is running a firewall and you intend to expose Traefik to the public internet (e.g., for `traefik` mode dashboard access or other services), ensure port 80 is allowed.

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

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
```