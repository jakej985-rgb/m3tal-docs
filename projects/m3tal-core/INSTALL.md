```markdown
# M3TAL Getting Started Guide

This guide provides operational steps for first-time users to set up and start the M3TAL ecosystem.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and version:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the `m3tal` CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up essential environment variables. This creates or updates the `/etc/m3tal/.env` file.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values:

*   **`DASHBOARD_EXPOSE_MODE`** (default: `local`):
    *   `local`: The M3TAL Dashboard will be directly accessible via `http://YOUR_IP:8082`. This is suitable for local network access or if you are not using Traefik.
    *   `traefik`: The M3TAL Dashboard will be exposed via the Traefik reverse proxy, accessible at `http://dash.YOUR_DOMAIN`. This requires Traefik to be running and a configured `DOMAIN`.
*   **`DOMAIN`** (default: `localhost`): Your primary domain for Traefik-routed services (e.g., `example.com`). If you set `DASHBOARD_EXPOSE_MODE` to `traefik`, the dashboard will be at `dash.YOUR_DOMAIN`.
*   **`PUID`** (default: `1000`): The User ID (UID) for containers that require specific user permissions. This usually matches your unprivileged user's UID.
*   **`PGID`** (default: `1000`): The Group ID (GID) for containers that require specific group permissions. This usually matches your unprivileged user's GID.
*   **`TZ`** (default: `America/Denver`): Your local timezone, used by containers.
*   **`DASHBOARD_SECRET`** (default: `change_me_immediately`): A unique secret key for the M3TAL Dashboard for enhanced security. **Change this to a strong, random value immediately.**
*   **`API_TOKEN`** (default: `change_me_api_token`): An API token for the M3TAL API daemon. **Change this to a strong, random value immediately.**
*   **`ADMIN_PASSWORD`** (default: `admin_pass`): The default password for the M3TAL Dashboard `admin` user. **Change this to a strong password immediately.**
*   **`BASE_STORAGE_PATH`** (default: `./data`): The base directory for all M3TAL data volumes. This will typically be `/mnt/m3tal_data` or similar.
*   **`CONFIG_PATH`** (default: `./data/config`): The base directory for M3TAL configuration volumes.
*   Other prompts may appear for specific service configurations (e.g., `VPN_USER`, `VPN_PASSWORD`, `TRAEFIK_WEB_PORT`). Provide values as appropriate for your setup.

### Step 4: Start the Routing Stack

Start the core M3TAL routing stack, including Traefik. This command reads all Docker Compose files located in `/docker/` and brings up their services.

```bash
m3tal up
```

This command will start containers defined in `routing-compose.yml` (Traefik, Cloudflared) and any other compose files placed in the `/docker/` directory.

### Step 5: Start the Dashboard

Start the M3TAL Dashboard container. This command handles downloading the necessary compose files, and starting the dashboard with the correct expose mode.

```bash
m3tal dash up
```

This command will:
1.  Download `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` to `/docker/`.
2.  Read the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Start the `m3tal-dashboard` container using the base compose file and the appropriate override (`m3tal-compose.local.yml` for `local` mode, or `m3tal-compose.traefik.yml` for `traefik` mode).

### Step 6: Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your server).
    Example: `http://192.168.1.100:8082` or `http://localhost:8082`
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in `/etc/m3tal/.env`).
    Example: `http://dash.example.com`

### Step 7: Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username:** `admin`
*   **Password:** The value you set for `ADMIN_PASSWORD` during the `m3tal config wizard`, or `admin_pass` if you accepted the default.

**To change the admin password:**
After logging in, or at any time from the CLI, you can update the dashboard password:

```bash
sudo m3tal dashpass
```

---

### Filesystem Contract

The following paths are critical to the M3TAL ecosystem:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains base Docker Compose files.         |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations (e.g., adding custom compose files). |
| `/docker/users.json`        | M3TAL Dashboard credential store. Managed by `m3tal dashpass`.         |

---

### Port Map

M3TAL utilizes the following ports:

| Port | Service                    | Access                                        |
| :--- | :------------------------- | :-------------------------------------------- |
| 80   | Traefik HTTP Entry Point   | Public (if Traefik is exposed)                |
| 8080 | M3TAL API Daemon           | Host-local only                               |
| 8081 | Traefik Dashboard          | Host-local only (accessed via `127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard Container  | Direct (local mode) or via Traefik (traefik mode) |

### Firewall Note

If you are exposing Traefik to the internet or your local network, you will need to allow traffic on port 80 (and potentially 443 for HTTPS) through your firewall. For `ufw`:

```bash
sudo ufw allow 80/tcp
sudo ufw reload
```

### Service Management

The M3TAL API daemon runs as a systemd service. Use `systemctl` for management and `journalctl` for logs:

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View live API service logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
```