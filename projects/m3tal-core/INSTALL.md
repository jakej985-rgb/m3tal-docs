```markdown
# GET STARTED: M3TAL First-Time Setup

This guide provides a step-by-step process for installing and configuring the M3TAL ecosystem for first-time users.

---

## Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
You can verify their installation and version using the following commands:

```bash
docker --version && docker compose version
```

## Step 2: Install M3TAL via APT

Execute the following commands in your terminal to install the M3TAL CLI binary and API daemon:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

After installation, run the M3TAL configuration wizard to set up essential environment variables. This creates or updates `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Here's an explanation of the key prompts:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   `local` (default): The dashboard will be directly accessible via `http://YOUR_IP:8082`. No Traefik configuration is required for dashboard access. This is suitable for local network access or testing.
    *   `traefik`: The dashboard will be accessible via `http://dash.YOUR_DOMAIN` using the Traefik reverse proxy. This requires Traefik to be running (`m3tal up`) and a domain configured.
*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). If `DASHBOARD_EXPOSE_MODE` is `traefik`, the dashboard will be at `dash.DOMAIN`.
*   **`DASHBOARD_PORT`**: The host port for the M3TAL Dashboard if `DASHBOARD_EXPOSE_MODE` is `local`. Default is `8082`.
*   **`PUID` / `PGID`**: The User ID and Group ID that Docker containers will use. Typically `1000` for the first user. Use `id -u` and `id -g` to find your current user's IDs.
*   **`TZ`**: Your timezone (e.g., `America/New_York`). Used by containers.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. This is critical for security; a strong, unique value should be used.
*   **`ADMIN_PASSWORD`**: The default password for the dashboard's `admin` user. **Change this immediately after initial login.**
*   **`API_TOKEN`**: A token used by the M3TAL CLI to authenticate with the M3TAL API daemon. Keep this secure.

## Step 4: Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks located in the `/docker/` directory, including the Traefik reverse proxy. Traefik handles routing for services exposed via domain names.

```bash
m3tal up
```

This command will:
*   Pull necessary Docker images (e.g., Traefik).
*   Create and start containers defined in `routing-compose.yml` and any other `*-compose.yml` files present in `/docker/`.
*   Establish the `proxy` network for inter-container communication and Traefik routing.

## Step 5: Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container.

```bash
m3tal dash up
```

This command performs the following actions:
1.  It downloads the latest dashboard Docker Compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from the official GitHub repository.
2.  It reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  It then starts the `m3tal-dashboard` container using the base `m3tal-compose.yml` along with the appropriate override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on `DASHBOARD_EXPOSE_MODE`.

## Step 6: Open Browser and Access Dashboard

The method to access the dashboard depends on your `DASHBOARD_EXPOSE_MODE` setting from Step 3.

### Mode 1: `DASHBOARD_EXPOSE_MODE=local` (Default)

Open your web browser and navigate to:

`http://YOUR_SERVER_IP:8082`

Replace `YOUR_SERVER_IP` with the actual IP address of the machine running M3TAL. If you are accessing from the same machine, you can use `http://localhost:8082`.

### Mode 2: `DASHBOARD_EXPOSE_MODE=traefik`

Open your web browser and navigate to:

`http://dash.YOUR_DOMAIN`

Replace `YOUR_DOMAIN` with the domain you configured during the wizard (e.g., `http://dash.example.com`). This mode requires Traefik to be running (`m3tal up`) and proper DNS configuration for your domain to point to your server's IP address.

## Step 7: Log In to the Dashboard

The default login credentials for the dashboard are:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during `sudo m3tal config wizard`.

**It is strongly recommended to change this password immediately after your first login.**
You can change the dashboard password for the `admin` user (or other users) using the M3TAL CLI:

```bash
sudo m3tal dashpass admin new_strong_password
```

## Filesystem Contract

M3TAL establishes a clear contract for its filesystem locations:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file, managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database, auto-created by the M3TAL API daemon.       |
| `/opt/m3tal/stack/`         | Canonical directory for M3TAL's internal Docker Compose files.     |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path where you place your custom Docker Compose files. |
| `/docker/users.json`        | Dashboard credential store, managed by `m3tal dashpass`.           |

## Port Table

The following ports are used by M3TAL components:

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public (if Traefik is exposed)              |
| 8080 | M3TAL API daemon (Go)       | Host-local only                             |
| 8081 | Traefik dashboard           | Host-local only                             |
| 8082 | M3TAL Dashboard (container) | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If you are exposing Traefik on port 80 to the internet or your local network, you may need to allow this port through your firewall. For systems using `ufw`:

```bash
sudo ufw allow 80/tcp
sudo ufw enable
```

## Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage its state using `systemctl` and view its logs with `journalctl`.

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