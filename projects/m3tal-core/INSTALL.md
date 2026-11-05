docs/GET_STARTED.md

## M3TAL Setup Guide

This guide provides instructions for installing and setting up M3TAL for the first time.

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and version by running:

```bash
docker --version && docker compose version
```

Example expected output:
```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.3-desktop.1
```

### Step 2: Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the CLI:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

Initialize your M3TAL environment by running the configuration wizard:

```bash
sudo m3tal config wizard
```

The wizard will prompt you for the following information:

*   **DOMAIN**: Your primary domain name (e.g., `example.com`). This is used by Traefik for routing services, allowing access to `dash.example.com` or `api.example.com` if Traefik mode is enabled.
*   **DASHBOARD_EXPOSE_MODE**:
    *   `local` (default): The M3TAL Dashboard will be directly exposed on `http://YOUR_SERVER_IP:8082`. No Traefik configuration is needed for dashboard access in this mode. This is suitable for LAN-only setups or initial testing.
    *   `traefik`: The M3TAL Dashboard will be exposed via Traefik at `http://dash.YOUR_DOMAIN`. This mode requires Traefik to be running via `m3tal up`. This is suitable for domain-based setups.
*   **CLOUDFLARE_TUNNEL_TOKEN** (Optional): A Cloudflare Tunnel token. If provided, M3TAL will configure a Cloudflare Tunnel container to expose services securely to the internet without requiring direct port forwarding.
*   **API_PORT**: The port on which the M3TAL API daemon will listen (default: `8080`).
*   **DASHBOARD_PORT**: The internal port on which the M3TAL Dashboard container will listen (default: `8082`).

### Step 4: Start the Routing Stack

Start the core M3TAL routing stack, which includes Traefik:

```bash
m3tal up
```

This command scans the `/docker/` directory for `*-compose.yml` files and starts all defined Docker Compose services. This typically includes the Traefik reverse proxy (`routing-compose.yml`) which handles incoming HTTP traffic and routes it to your services.

### Step 5: Start the M3TAL Dashboard

Start the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command pulls the latest M3TAL Dashboard Docker image if not already present and starts the dashboard container. It automatically applies the appropriate Docker Compose override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE` configured in `/etc/m3tal/.env`.

### Step 6: Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard:

*   **If `DASHBOARD_EXPOSE_MODE` is `local`:**
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your M3TAL host)
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in Step 3)

### Step 7: Log In

The default login credentials for the M3TAL Dashboard are `admin`/`admin`.

To change these credentials, use the following command:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new username and password. This updates the `/docker/users.json` file.

---

### Filesystem Contract

M3TAL establishes a clear filesystem structure for its operation:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the M3TAL API daemon.           |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing directory for all Docker Compose stacks and related configurations. Place your `*-compose.yml` files here. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |

### Port Overview

The following ports are used by core M3TAL components:

| Port | Service                    | Access                                       |
| :--- | :------------------------- | :------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public (when Traefik is active and exposed)  |
| 8080 | M3TAL API daemon (Go)      | Host-local                                   |
| 8081 | Traefik dashboard          | Host-local only                              |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) |

#### Firewall Configuration

If you expose Traefik on port 80 to the internet or your local network, ensure your firewall allows incoming connections on this port. For systems using UFW (Uncomplicated Firewall), you can allow access with:

```bash
sudo ufw allow 80
```

### Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage its state using `systemctl` and view its logs with `journalctl`:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```