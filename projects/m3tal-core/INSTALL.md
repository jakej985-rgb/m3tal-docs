# Getting Started with M3TAL

This guide provides step-by-step instructions for setting up M3TAL on your system.

## Step 1: Prerequisites

Before you begin, ensure you have the following software installed:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running these commands:

```bash
docker --version
docker compose version
```

If either command returns an error or shows an older version, please refer to the official Docker documentation for installation and upgrade instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed via an APT repository. Run the following three commands to add the repository and install the M3TAL package:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The `m3tal config wizard` will guide you through essential initial configurations.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions:

*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL dashboard is accessed.
    *   `local`: Exposes the dashboard directly via a port on your host machine (e.g., `http://YOUR_IP:8082`). This is the default and recommended for beginners.
    *   `traefik`: Exposes the dashboard through Traefik, typically via a subdomain (e.g., `http://dash.DOMAIN`). Requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name you will use to access M3TAL services. Defaults to `localhost`.
*   **`DASHBOARD_PORT`**: The port the M3TAL dashboard will listen on. Defaults to `8082`.
*   **`HTTP_PORT`**: The port for the M3TAL API. Defaults to `8080`.
*   **`PUID` / `PGID`**: The User ID and Group ID for running Docker containers. Defaults to `1000`.
*   **`TZ`**: Your local timezone (e.g., `America/New_York`).

The wizard will save your choices to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

M3TAL uses Traefik as its reverse proxy to manage access to various services. To start the Traefik routing stack, run:

```bash
m3tal up
```

This command orchestrates all Docker Compose files found in the `/docker/` directory, including the Traefik reverse proxy configuration.

## Step 5: Start the Dashboard

The M3TAL dashboard provides a graphical interface for managing your services. To start the dashboard container:

```bash
m3tal dash up
```

This command will pull the latest M3TAL dashboard image from the registry and start the container according to your `DASHBOARD_EXPOSE_MODE` configuration.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's URL:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default):
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's local IP address. You can often use `localhost` if you are on the same machine).

*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured in Step 3. Ensure Traefik is running and DNS is correctly configured for this subdomain).

## Step 7: Log In to the Dashboard

Upon accessing the dashboard, you will be prompted to log in.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default value for `ADMIN_PASSWORD` in your `.env` file. **It is highly recommended to change this immediately.**)

To change your dashboard password:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new secure password.

## Filesystem Contract

M3TAL relies on a specific filesystem structure for its configuration and data:

| Path                     | Purpose                                                                           | Notes                                             |
| :----------------------- | :-------------------------------------------------------------------------------- | :------------------------------------------------ |
| `/etc/m3tal/.env`        | Primary configuration file. Stores all environment variables.                     | Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Stores application state and configurations.             | Auto-created by the API daemon.                   |
| `/docker`                | Symlink to the canonical stack directory.                                         | Points to `/opt/m3tal/stack/`. User stacks are added here. |
| `/docker/users.json`     | Dashboard credential store (username/password hash).                              | Managed by `m3tal dashpass`.                      |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains core M3TAL compose files and Traefik config. | This is where the `/docker` symlink points.       |

## Port Map

The following ports are used by M3TAL and its components:

| Port | Service                  | Access                                            |
| :--- | :----------------------- | :------------------------------------------------ |
| 80   | Traefik HTTP Entrypoint  | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)     |
| 8080 | M3TAL API Daemon (Go)    | Host-local only                                   |
| 8081 | Traefik Dashboard        | Host-local only (internal Traefik UI)             |
| 8082 | M3TAL Dashboard          | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If your server is accessible from the internet and you are using Traefik (`DASHBOARD_EXPOSE_MODE=traefik`), ensure that port 80 is open in your firewall. If you are using `ufw` (Uncomplicated Firewall):

```bash
sudo ufw allow 80
```

## Service Management

M3TAL's API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```

*   **Restart the M3TAL API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```