## GET_STARTED.md

This guide provides a complete, step-by-step setup for first-time M3TAL users.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation by running:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

M3TAL is distributed as a single Go binary via APT. Execute the following commands to add the M3TAL repository and install the CLI:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the configuration wizard

After installation, configure M3TAL using the interactive wizard. This will create or update `/etc/m3tal/.env` with your system's settings.

```bash
sudo m3tal config wizard
```

During the wizard, you will be prompted for several configuration values:

*   **`DOMAIN`**: Sets the base domain for services routed through Traefik (e.g., `api.yourdomain.com`). Default is `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): Direct port access on `http://HOST_IP:8082` (or `http://localhost:8082`). No Traefik required.
    *   `traefik`: Access via `http://dash.YOUR_DOMAIN` through the Traefik reverse proxy. Requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The direct port for dashboard access when `DASHBOARD_EXPOSE_MODE` is `local`. Default is `8082`.
*   **`PUID` / `PGID`**: Defines the User ID and Group ID that Docker containers will run as, ensuring correct file permissions. Default is `1000` for both.
*   **`TZ`**: Sets the timezone for all Docker containers, ensuring accurate timestamps. Default is `America/Denver`.
*   **`BASE_STORAGE_PATH`**, **`CONFIG_PATH`**, **`MEDIA_PATH`**, **`DOWNLOADS_PATH`**: These define the base directories for persistent data storage, configuration files, media, and downloads, respectively. They are used for Docker volume mounts. By default, they point to subdirectories within `./data`.

Review and confirm the settings to proceed.

### Step 4: Start the routing stack (Traefik)

The M3TAL CLI provides a unified command to start your Docker Compose stacks. The primary routing stack (Traefik) is essential for exposing services.

```bash
m3tal up
```

This command executes `docker compose up -d` for all `*-compose.yml` files located in the `/docker/` directory. This includes `routing-compose.yml`, which deploys the Traefik reverse proxy.

### Step 5: Start the dashboard

Deploy the M3TAL Dashboard container. The `m3tal dash up` command handles pulling the correct image and applying the appropriate configuration based on your `DASHBOARD_EXPOSE_MODE` setting.

```bash
m3tal dash up
```

This command pulls the `m3tal-dashboard` image and starts its container, applying the correct configuration based on `DASHBOARD_EXPOSE_MODE` set in `/etc/m3tal/.env`.

### Step 6: Open browser

Access the M3TAL Dashboard in your web browser:

*   If `DASHBOARD_EXPOSE_MODE` is `local`: Navigate to `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with your host machine's IP address, or use `localhost` if accessing from the server itself).
*   If `DASHBOARD_EXPOSE_MODE` is `traefik`: Navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in Step 3).

### Step 7: Log in

The default credentials for the M3TAL Dashboard are:

*   **Username**: `admin`
*   **Password**: `admin_pass`

**Immediately change this default password** after your first login using the following command:

```bash
sudo m3tal dashpass
```

---

### Filesystem Contract

M3TAL establishes a clear filesystem contract for its operational files:

| Path                        | Purpose                                                      |
| :-------------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the API daemon.       |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.     |

### Port Table

The following ports are used by M3TAL components:

| Port | Service                    | Access                                               |
| :--- | :------------------------- | :--------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public (if Traefik is exposed)                       |
| 8080 | M3TAL API daemon (Go)      | Host-local                                           |
| 8081 | Traefik dashboard          | Host-local only                                      |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) |

### Firewall Note

If you expose Traefik on port 80 (e.g., for domain-based routing), ensure your firewall allows incoming connections on this port:

```bash
sudo ufw allow 80
```

### Service Management

The M3TAL API daemon runs as a systemd service. You can manage its status and view logs using standard `systemctl` and `journalctl` commands:

*   **Check API daemon status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View API daemon logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```