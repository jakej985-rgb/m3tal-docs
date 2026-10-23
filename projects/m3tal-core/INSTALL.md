This document provides a step-by-step guide for first-time users to set up and configure the M3TAL ecosystem.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation by running:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

Execute the following commands in your terminal to install the M3TAL CLI binary and API daemon:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up essential environment variables. This wizard creates or updates the `/etc/m3tal/.env` file.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Below is an explanation for each:

*   **`DOMAIN`**: The base domain name for your services (e.g., `example.com`). This is used by Traefik for routing services like the dashboard (`dash.DOMAIN`) and the M3TAL API (`api.DOMAIN`). For local setups without a domain, `localhost` is a suitable default.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   **`local` (default)**: The dashboard will be directly accessible via a host port, typically `http://YOUR_IP:8082`. No Traefik configuration is required for dashboard access in this mode.
    *   **`traefik`**: The dashboard will be accessible via `http://dash.DOMAIN` through the Traefik reverse proxy. This requires Traefik to be running.
*   **`DASHBOARD_SECRET`**: A secret key used for securing dashboard user sessions. This should be a long, random string.
*   **`API_TOKEN`**: A token used to secure communication with the M3TAL API daemon. This should be a long, random string.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user on the M3TAL Dashboard. Change this immediately after first login.
*   **`PUID`** (User ID) / **`PGID`** (Group ID): These define the user and group IDs that containers will run as, ensuring proper file permissions on mounted volumes. For most users, your own user's UID/GID (e.g., `id -u` and `id -g`) are appropriate.
*   **`TZ`**: Your local timezone (e.g., `America/New_York`). This ensures correct time synchronization within containers.
*   **`BASE_STORAGE_PATH`**: The base directory for all M3TAL-managed storage, such as `/mnt/data`.
*   **`CONFIG_PATH`**: The base directory for M3TAL-managed configuration files, typically inside `BASE_STORAGE_PATH`.

### Step 4: Start the Routing Stack (Traefik)

The M3TAL API daemon (`m3tal-api.service`) starts automatically after installation via systemd. Now, start the Docker-based routing stack, which includes Traefik.

```bash
m3tal up
```

This command scans the `/docker/` directory for all `*-compose.yml` files and brings them up using Docker Compose V2. This includes `routing-compose.yml`, which deploys the Traefik reverse proxy.

### Step 5: Start the Dashboard

Deploy the M3TAL Dashboard container. The CLI will download the necessary compose files and start the dashboard based on your `DASHBOARD_EXPOSE_MODE` setting from Step 3.

```bash
m3tal dash up
```

This command pulls the latest dashboard image (e.g., `ghcr.io/jakej985-rgb/m3tal-godash:debug`) and starts its container. It intelligently selects between `m3tal-compose.local.yml` (for `local` mode, adding a direct port binding) and `m3tal-compose.traefik.yml` (for `traefik` mode, adding Traefik labels) based on the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Step 6: Open Browser

Access the M3TAL Dashboard through your web browser. The URL depends on your `DASHBOARD_EXPOSE_MODE` configuration:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    Open your browser to `http://YOUR_IP:8082`.
    (Replace `YOUR_IP` with the actual IP address of your M3TAL server, e.g., `192.168.1.100`. You can find your server's IP with `ip a`).
*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    Open your browser to `http://dash.DOMAIN`.
    (Replace `DOMAIN` with the value you configured in Step 3, e.g., `http://dash.example.com`).

### Step 7: Log In

Upon accessing the dashboard, you will be prompted to log in.

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you configured during the `m3tal config wizard` in Step 3.

**It is strongly recommended to change the default password immediately.** You can do this via the dashboard's user management interface or by using the CLI:

```bash
sudo m3tal dashpass admin new_strong_password
```

---

### Filesystem Contract

M3TAL utilizes specific locations on your filesystem for its operations:

| Path                     | Purpose                                                                                |
| :----------------------- | :------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file, storing environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database, used by the API daemon to persist application state. Auto-created. |
| `/opt/m3tal/stack/`      | Canonical directory where M3TAL stores its core Docker Compose files (e.g., `routing-compose.yml`). |
| `/docker`                | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path for placing and managing your Docker Compose stacks. |
| `/docker/users.json`     | The credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`.             |

### Port Map

M3TAL services use the following network ports:

| Port | Service                               | Access                                                                  |
| :--- | :------------------------------------ | :---------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (if Traefik is exposed and routing HTTP traffic)                 |
| 8080 | M3TAL API daemon (Go binary)          | Host-local. Traefik routes to it if `DOMAIN` is configured.             |
| 8081 | Traefik dashboard                     | Host-local only (accessed via `http://localhost:8081` on the server)    |
| 8082 | M3TAL Dashboard (Python/Flask)        | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`). |

### Firewall Note

If you expose Traefik (Port 80) to the internet or your local network, ensure your firewall allows incoming connections on the necessary ports. For example, to allow HTTP traffic through Traefik:

```bash
sudo ufw allow 80/tcp
```

### Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using standard systemd commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```