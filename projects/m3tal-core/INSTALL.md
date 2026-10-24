docs/GET_STARTED.md

# Getting Started with M3TAL

This guide provides the complete, step-by-step process for a first-time setup of the M3TAL ecosystem.

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and versions:

```bash
docker --version && docker compose version
```

## 2. Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the `m3tal` CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

Initialize M3TAL's primary configuration file (`/etc/m3tal/.env`) by running the interactive wizard. This will prompt you for essential environment variables.

```bash
sudo m3tal config wizard
```

Explanation of key prompts:

-   **`DOMAIN`**: The base domain name for your services. If you plan to expose M3TAL services via Traefik (e.g., `api.yourdomain.com`, `dash.yourdomain.com`), set this to your actual domain. Default is `localhost`.
-   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is accessible:
    -   `local` (default): The dashboard container will expose itself directly on a host port (default `8082`). Access via `http://YOUR_SERVER_IP:8082`. This mode does not require Traefik.
    -   `traefik`: The dashboard will be accessible via Traefik using `http://dash.DOMAIN`. This mode requires Traefik to be running.
-   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for internal security. Generate a strong, unique value.
-   **`API_TOKEN`**: An authentication token used by the `m3tal` CLI and dashboard to communicate with the M3TAL API daemon. Generate a strong, unique value.
-   **`ADMIN_PASSWORD`**: The default password for the `admin` user of the M3TAL dashboard. Change this from the default immediately.
-   **`PUID`** and **`PGID`**: User ID and Group ID that containers will use for file permissions. Typically `1000` for the first non-root user on Linux.
-   **`TZ`**: The timezone for containers (e.g., `America/Denver`).
-   **`BASE_STORAGE_PATH`**: The base directory on the host where M3TAL will store all persistent data for its services and other deployed stacks.

## 4. Start the Routing Stack

Start Traefik, the reverse proxy, and any other foundational routing services (like Cloudflared, if configured). This command initiates all Docker Compose stacks defined by `*-compose.yml` files within the `/docker/` directory, including `routing-compose.yml`.

```bash
m3tal up
```

This command typically starts Traefik, which will listen for HTTP traffic on port 80.

## 5. Start the M3TAL Dashboard

Deploy and start the M3TAL Dashboard container. This command downloads the latest dashboard compose files from GitHub, reads your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`, and starts the `m3tal-dashboard` container with the appropriate configuration. It pulls the `ghcr.io/jakej985-rgb/m3tal-godash:debug` image.

```bash
m3tal dash up
```

## 6. Access the M3TAL Dashboard

Open your web browser and navigate to the M3TAL Dashboard:

-   **If `DASHBOARD_EXPOSE_MODE` is `local`**:
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with your server's actual IP address or `localhost`).
-   **If `DASHBOARD_EXPOSE_MODE` is `traefik`**:
    `http://dash.DOMAIN` (e.g., `http://dash.example.com`). Ensure Traefik is running (`m3tal up` must have completed successfully).

## 7. Log In to the Dashboard

-   **Default Username**: `admin`
-   **Default Password**: The value configured for `ADMIN_PASSWORD` during `sudo m3tal config wizard`.

Change the default password immediately using the following CLI command:

```bash
sudo m3tal dashpass
```

## Filesystem Contract

M3TAL utilizes specific paths for its operations and persistent data:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the M3TAL API daemon.         |
| `/opt/m3tal/stack/`         | Canonical directory for M3TAL's core Docker Compose stack files.     |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations. Place your custom `*-compose.yml` files here. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.             |

## Port Map

Key ports used by M3TAL services:

| Port | Service                    | Access                                                    |
| :--- | :------------------------- | :-------------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services are exposed) |
| 8080 | M3TAL API daemon (Go)      | Host-local only                                           |
| 8081 | Traefik dashboard (internal) | Host-local only, used for Traefik's own dashboard         |
| 8082 | M3TAL Dashboard            | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Configuration

If you are exposing Traefik to the internet or your local network, ensure your firewall allows incoming connections on port 80 (and 443 if you configure HTTPS later).

Example for `ufw`:

```bash
sudo ufw allow 80/tcp
sudo ufw enable # if not already enabled
```

If you are using `DASHBOARD_EXPOSE_MODE=local` and wish to access the dashboard from other devices on your network, you might also need to allow port 8082:

```bash
sudo ufw allow 8082/tcp
```

## M3TAL API Service Management

The M3TAL API daemon runs as a systemd service. Use `systemctl` and `journalctl` for service management and logging.

-   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
-   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
-   **Restart the API daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```