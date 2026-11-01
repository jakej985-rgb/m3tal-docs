# M3TAL: Getting Started Guide

This guide will walk you through the initial setup and configuration of the M3TAL ecosystem.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system.

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installation by running:

```bash
docker --version && docker compose version
```

If either command fails, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed as a Debian package. Execute the following commands to add the M3TAL repository and install the `m3tal` binary.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential parameters for your environment.

```bash
sudo m3tal config wizard
```

You will be presented with several prompts:

*   **`DASHBOARD_EXPOSE_MODE`**: This determines how the M3TAL dashboard will be accessible.
    *   `local`: The dashboard will be directly accessible via its port (default 8082). This is the recommended setting for beginners and LAN-only setups.
    *   `traefik`: The dashboard will be exposed via Traefik, allowing access via a domain name (e.g., `dash.yourdomain.com`). This requires Traefik to be running.
*   **`DOMAIN`**: Your primary domain name. This is used for configuring Traefik if `DASHBOARD_EXPOSE_MODE` is set to `traefik`. Defaults to `localhost`.
*   **`HTTP_PORT`**: The port the M3TAL API daemon will listen on. Defaults to `8080`.
*   **`DASHBOARD_PORT`**: The port the M3TAL dashboard container will listen on internally. Defaults to `8082`.
*   **`API_PORT`**: The port the M3TAL API daemon will listen on internally. Defaults to `5050`.
*   **`PUID` / `PGID`**: The User ID and Group ID for running Docker containers. Use `id -u` and `id -g` respectively to find these for your primary user. Defaults to `1000`.
*   **`TZ`**: Your timezone. Use standard timezone names (e.g., `America/New_York`). Defaults to `America/Denver`.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. **Change this from the default `change_me_immediately` for security.**
*   **`API_TOKEN`**: A token for API access. **Change this from the default `change_me_api_token` for security.**
*   **`ADMIN_PASSWORD`**: The password for the dashboard administrator. **Change this from the default `admin_pass` for security.**

## Step 4: Start the Routing Stack (Traefik)

This command initiates the M3TAL Docker stack, including the Traefik reverse proxy.

```bash
m3tal up
```

This command orchestrates all Docker Compose files found in the `/docker/` directory. Traefik will start and begin listening for incoming requests on port 80.

## Step 5: Start the Dashboard

This command specifically pulls the M3TAL dashboard image and starts its container.

```bash
m3tal dash up
```

This command will:
1. Download the necessary Docker Compose files for the dashboard.
2. Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3. Start the dashboard container with the appropriate configuration based on your chosen mode.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address.

*   If you chose `DASHBOARD_EXPOSE_MODE=local` (default), access it at:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. You can also use `http://localhost:8082` if accessing from the same machine.)

*   If you chose `DASHBOARD_EXPOSE_MODE=traefik` and have configured a domain, access it at:
    `http://dash.YOUR_DOMAIN`
    (Replace `YOUR_DOMAIN` with the domain you configured.)

## Step 7: Log In

When prompted, use the following default credentials:

*   **Username**: `admin`
*   **Password**: `admin_pass` (This is the default value for `ADMIN_PASSWORD`. It's highly recommended to change this immediately.)

**To change the dashboard password:**

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new administrator password.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure to manage its configuration and state.

| Path                        | Purpose                                                                     |
| :-------------------------- | :-------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file for M3TAL. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite database storing M3TAL's state. Automatically created by the API.    |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing directory for all M3TAL Docker Compose files. |
| `/docker/users.json`        | Stores dashboard credentials. Managed by `m3tal dashpass`.                  |

## Port Map

The following ports are used by M3TAL and its components:

| Port   | Service                | Access                                         |
| :----- | :--------------------- | :--------------------------------------------- |
| 80     | Traefik (HTTP Entry)   | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)    |
| 8080   | M3TAL API Daemon (Go)  | Host-local                                     |
| 8081   | Traefik Dashboard      | Host-local only (`http://127.0.0.1:8081`)      |
| 8082   | M3TAL Dashboard        | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If you are exposing Traefik to the public internet (e.g., for `DASHBOARD_EXPOSE_MODE=traefik` to work externally), ensure that port 80 is open in your firewall.

If you are using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check the status of the API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View real-time logs from the API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```