```markdown
# docs/GET_STARTED.md

This guide details the steps for setting up M3TAL on a new system for the first time.

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and versions:

```bash
docker --version && docker compose version
```

Expected output will show versions for Docker Engine and Docker Compose V2. If not installed, please refer to the official Docker documentation for your operating system.

## 2. Install M3TAL via APT

M3TAL is distributed via an APT repository for easy installation and updates. Execute the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.list stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

This will install the `m3tal` CLI binary to `/usr/bin/m3tal` and the `m3tal-api.service` systemd daemon.

## 3. Run the Configuration Wizard

After installation, configure M3TAL using the interactive wizard. This will generate the primary configuration file at `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several details:

*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is used by Traefik for routing services like `dash.example.com` or `api.example.com`. Default is `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard will be directly accessible via `http://YOUR_IP:8082`. No Traefik configuration is needed for dashboard access. This is ideal for initial setup or LAN-only deployments.
    *   `traefik`: The dashboard will be accessible via `http://dash.YOUR_DOMAIN` using Traefik as a reverse proxy. This requires Traefik to be running.
*   **`PUID` / `PGID`**: The User ID and Group ID that containers will run as. Typically `1000` for a standard user. This ensures proper file permissions for volumes.
*   **`TZ`**: Your timezone (e.g., `America/New_York`). Used by containers for correct timekeeping.
*   **`DASHBOARD_SECRET`**: A secret key used by the M3TAL Dashboard for session management. A strong, random value is recommended.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user on the M3TAL Dashboard. **Change this immediately after setup.**

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. This includes the core routing stack managed by `routing-compose.yml`, which deploys Traefik.

```bash
m3tal up
```

This command executes `docker compose` operations for all `*-compose.yml` files located in `/docker/` (which is a symlink to `/opt/m3tal/stack/`). The `routing-compose.yml` file configures the Traefik reverse proxy to listen on port 80.

## 5. Start the M3TAL Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container.

```bash
m3tal dash up
```

This command performs the following actions:
1.  Downloads the latest `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
2.  Reads the `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container, applying the appropriate compose override based on `DASHBOARD_EXPOSE_MODE`. It pulls the necessary Docker image (`ghcr.io/jakej985-rgb/m3tal-godash:debug`) if not already present.

## 6. Open the M3TAL Dashboard in Your Browser

Access the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` configuration:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    Open your browser and navigate to `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your server) or `http://localhost:8082` if accessing from the server itself.

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Open your browser and navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in step 3). This requires Traefik to be running (as started in step 4).

## 7. Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the `m3tal config wizard` (default is `admin_pass` if not changed).

**It is highly recommended to change the admin password immediately.**
You can change the dashboard password using the M3TAL CLI:

```bash
sudo m3tal dashpass
```
Follow the prompts to set a new password for the `admin` user. This updates the `/docker/users.json` file.

## Filesystem Contract

M3TAL establishes a clear contract for its operational files:

*   `/etc/m3tal/.env`: The primary configuration file, managed by `m3tal config wizard` and `m3tal config set`.
*   `/var/lib/m3tal/state.db`: The SQLite database storing M3TAL's state, automatically created and managed by the API daemon.
*   `/opt/m3tal/stack/`: The canonical directory where M3TAL stores its core Docker Compose files (`routing-compose.yml`, `m3tal-compose.yml`, etc.) and Traefik configurations.
*   `/docker`: A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations, allowing you to place custom Docker Compose files for new services (e.g., `/docker/my-app-compose.yml`).
*   `/docker/users.json`: The JSON file storing dashboard user credentials. Managed by `m3tal dashpass`.

## Important Ports

| Port | Service                               | Access                                    |
| :--- | :------------------------------------ | :---------------------------------------- |
| 80   | Traefik HTTP Entrypoint               | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services exposed) |
| 8080 | M3TAL API Daemon (Go)                 | Host-local only (internal communication)  |
| 8081 | Traefik Dashboard                     | Host-local only (`127.0.0.1:8081`)        |
| 8082 | M3TAL Dashboard (Python/Flask)        | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Note

If Traefik is enabled and exposed to the public internet (i.e., you intend to access services via port 80/443 from outside your local network), you may need to open port 80 (and 443 for HTTPS, if configured) on your firewall. For `ufw`:

```bash
sudo ufw allow 80/tcp
```

## Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service. You can manage its state and view its logs using standard systemd commands:

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View API service logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```

*   **Restart the API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
```