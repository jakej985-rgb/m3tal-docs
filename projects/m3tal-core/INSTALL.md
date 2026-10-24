# M3TAL - Getting Started

This guide will walk you through setting up M3TAL for the first time.

## Step 1: Prerequisites

Before you begin, ensure you have the following installed:

- **Docker Engine**: The containerization platform.
- **Docker Compose V2**: For orchestrating Docker containers.

You can verify your installation by running:

```bash
docker --version && docker compose version
```

If these commands do not return version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed as a Debian package for easy installation. Run the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential parameters. Execute the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of what each prompt means:

*   **`[+] API_PORT (default: 5050)`**: The port the M3TAL API daemon will listen on. This is usually accessed locally.
*   **`[+] DASHBOARD_PORT (default: 8082)`**: The port the M3TAL dashboard will be accessible on.
*   **`[+] DASHBOARD_EXPOSE_MODE (default: local)`**: Determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via its port. Ideal for LAN access.
    *   `traefik`: Exposes the dashboard through Traefik (requires `m3tal up`). Ideal for domain-based access.
*   **`[+] HTTP_PORT (default: 8080)`**: The port for general HTTP services, often used by the API.
*   **`[+] STATE_DIR (default: ./state)`**: The directory where M3TAL stores its state data.
*   **`[+] LOG_LEVEL (default: info)`**: Sets the verbosity of M3TAL logs (`debug`, `info`, `warn`, `error`).
*   **`[+] DASHBOARD_SECRET (default: change_me_immediately)`**: A secret key for securing the dashboard. **Change this immediately.**
*   **`[+] API_TOKEN (default: change_me_api_token)`**: A token for authenticating with the M3TAL API. **Change this immediately.**
*   **`[+] ADMIN_PASSWORD (default: admin_pass)`**: The password for the dashboard's administrative user. **Change this immediately.**
*   **`[+] NETWORK_NAME (default: m3tal)`**: The Docker network name M3TAL will use.
*   **`[+] LOCAL_IP (default: 127.0.0.1)`**: Your local machine's IP address.
*   **`[+] DOMAIN (default: localhost)`**: The domain name you will use to access M3TAL services (e.g., `dash.localhost`).
*   **`[+] VPN_USER (default: user)`**: Username for VPN connections (if configured).
*   **`[+] VPN_PASSWORD (default: password)`**: Password for VPN connections (if configured).
*   **`[+] BASE_STORAGE_PATH (default: ./data)`**: The base directory for M3TAL storage.
*   **`[+] MEDIA_PATH (default: ./data/media)`**: Directory for media files.
*   **`[+] CONFIG_PATH (default: ./data/config)`**: Directory for configuration files.
*   **`[+] DOWNLOADS_PATH (default: ./data/downloads)`**: Directory for downloads.
*   **`[+] PUID (default: 1000)`**: The user ID to run containers as.
*   **`[+] PGID (default: 1000)`**: The group ID to run containers as.
*   **`[+] TZ (default: America/Denver)`**: Your local timezone.
*   **`[+] TRAEFIK_WEB_PORT (default: 80)`**: The host port Traefik listens on for HTTP traffic.
*   **`[+] TRAEFIK_WEBHTTPS_PORT (default: 443)`**: The host port Traefik listens on for HTTPS traffic.
*   **`[+] TRAEFIK_DASHBOARD_PORT (default: 8080)`**: The port Traefik's own dashboard is exposed on.
*   **`[+] DEBUG_MODE (default: false)`**: Enables debug logging for M3TAL.
*   **`[+] METRICS_ENABLED (default: true)`**: Enables Prometheus metrics collection.

## Step 4: Start the Routing Stack (Traefik)

The routing stack, primarily Traefik, handles incoming network traffic and directs it to the appropriate services. To start it, run:

```bash
m3tal up
```

This command will orchestrate all Docker Compose files found in the `/docker/` directory, starting services like Traefik.

## Step 5: Start the Dashboard

Now, start the M3TAL dashboard. This command will pull the necessary Docker image and start the dashboard container:

```bash
m3tal dash up
```

The dashboard's startup behavior is determined by the `DASHBOARD_EXPOSE_MODE` setting in your configuration.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (the default), access the dashboard at: `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or `localhost` if on the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is running, access the dashboard at: `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured).

## Step 7: Log In

When prompted, use the following default credentials:

*   **Username:** `admin`
*   **Password:** The password you set during the configuration wizard (default: `admin_pass`).

**It is strongly recommended to change your dashboard password immediately after logging in.** You can do this via the dashboard interface or by running the following command:

```bash
sudo m3tal dashpass new_password
```

Replace `new_password` with your desired strong password.

---

## Filesystem Contract

M3TAL relies on a specific filesystem structure for configuration and state management.

| Path                        | Purpose                                                                                                                              |
| :-------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary environment file for M3TAL. This file is managed by the `m3tal config wizard` and `m3tal config set` commands.            |
| `/var/lib/m3tal/state.db`   | The SQLite database storing M3TAL's operational state. This file is automatically created and managed by the M3TAL API daemon.       |
| `/docker` (symlink)         | This directory is a symbolic link to `/opt/m3tal/stack/`. All M3TAL Docker Compose files and related configurations reside within `/opt/m3tal/stack/`. User-added stack Compose files should be placed in this symlinked directory. |

## Port Map

The following ports are used by M3TAL and its components:

| Port | Service                | Access                                              |
| :--- | :--------------------- | :-------------------------------------------------- |
| 80   | Traefik HTTP entrypoint | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)       |
| 8080 | M3TAL API daemon (Go)  | Host-local only                                     |
| 8081 | Traefik dashboard      | Host-local only                                     |
| 8082 | M3TAL Dashboard        | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If Traefik is exposed to the public internet (i.e., `DASHBOARD_EXPOSE_MODE=traefik`), ensure that port 80 is open in your firewall:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs (follow live):**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```