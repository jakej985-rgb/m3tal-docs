# GET_STARTED.md

This guide provides a complete, operational setup for M3TAL for first-time users.

---

### Step 1: Prerequisites

M3TAL relies on Docker Engine and Docker Compose V2. Ensure both are installed on your system before proceeding.

Verify your Docker installation:

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

After installation, run the configuration wizard to set up essential environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for the following key settings:

*   **`DOMAIN`**: The domain or IP address your M3TAL instance will use (e.g., `example.com` or `192.168.1.100`). Defaults to `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is accessed.
    *   `local` (Default): The dashboard is directly exposed on a host port (default `8082`). Ideal for local network access or testing.
    *   `traefik`: The dashboard is routed via Traefik using a subdomain (e.g., `dash.DOMAIN`). Requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The host port to expose the M3TAL Dashboard on if `DASHBOARD_EXPOSE_MODE` is `local`. Defaults to `8082`.
*   **`PUID`** (User ID) / **`PGID`** (Group ID): These define the user and group IDs used by containers for file permissions. Defaults to `1000`, corresponding to the first user created on most Linux systems. Use `id -u` and `id -g` to find your current user's IDs.
*   **`TZ`**: Your timezone (e.g., `America/Denver`). This ensures containers operate with correct time.
*   **`ADMIN_PASSWORD`**: The default login password for the M3TAL Dashboard. **Change this from the default.**
*   Other variables like `DASHBOARD_SECRET` and `API_TOKEN` are automatically generated for security.

### Step 4: Start the Routing Stack (Traefik)

The `m3tal up` command initializes and starts all Docker Compose stacks found in the `/docker/` directory, including the Traefik reverse proxy. Traefik handles routing for services by default on port 80.

```bash
m3tal up
```

This command iterates through all `*-compose.yml` files located in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`) and starts the Docker services defined within them using `docker compose up -d`. This includes `routing-compose.yml`, which deploys Traefik.

### Step 5: Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container.

```bash
m3tal dash up
```

This command performs the following actions:
1.  Downloads the latest M3TAL Dashboard Docker Compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
2.  Reads the `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container with the appropriate Docker Compose override (`m3tal-compose.local.yml` for `local` mode or `m3tal-compose.traefik.yml` for `traefik` mode). The dashboard container communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.

### Step 6: Open the Dashboard in Your Browser

Access the M3TAL Dashboard based on your configured `DASHBOARD_EXPOSE_MODE`:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    Open your web browser and navigate to:
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your server).
    If on the same machine, `http://localhost:8082` will also work.

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    Open your web browser and navigate to:
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in the wizard, e.g., `http://dash.example.com`).
    This mode requires Traefik to be running via `m3tal up` and proper DNS records pointing `dash.YOUR_DOMAIN` to your server's IP address.

### Step 7: Log In

Log in to the M3TAL Dashboard using the credentials configured during the wizard:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set in Step 3.

**Important**: Change the default `ADMIN_PASSWORD` immediately if you did not do so during the wizard. You can change the dashboard password using the CLI:

```bash
sudo m3tal dashpass
```

---

### Filesystem Contract

M3TAL adheres to a strict filesystem contract to ensure consistent operation and simplify management.

| Path                     | Purpose                                                                                                                                                                                                                                                                |
| :----------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | **Primary Configuration File**. Contains environment variables loaded by M3TAL CLI and API. Managed by `m3tal config wizard` and `m3tal config set`.                                                                                                                      |
| `/var/lib/m3tal/state.db`| **SQLite State Database**. Automatically created and managed by the `m3tal-api.service`. Stores M3TAL's internal state, such as registered Docker services and their configurations.                                                                                     |
| `/opt/m3tal/stack/`      | **Canonical Stack Directory**. This is the internal directory where M3TAL stores its core Docker Compose files (e.g., for routing, dashboard) and where you should place your custom `*-compose.yml` files for user-defined stacks.                                    |
| `/docker`                | **User-Facing Symlink to `/opt/m3tal/stack/`**. This symlink provides a convenient and standardized path for users to interact with their Docker Compose stacks. All `m3tal` commands operating on compose files resolve to this directory.                                 |
| `/docker/users.json`     | **Dashboard Credential Store**. This file, located within the M3TAL stack directory, stores encrypted credentials for the M3TAL Dashboard. It is managed by the `m3tal dashpass` command.                                                                               |

---

### Port Map

M3TAL uses the following ports for its core services. Ensure these ports are available and, if necessary, configured in your firewall.

| Port | Service                                   | Access                                                                  |
| :--- | :---------------------------------------- | :---------------------------------------------------------------------- |
| 80   | Traefik HTTP Entry Point                  | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services exposed) |
| 443  | Traefik HTTPS Entry Point                 | Public (if HTTPS is enabled for Traefik)                                |
| 8080 | M3TAL API Daemon (Go binary)              | Host-local only (Dashboard and other internal components connect here)  |
| 8081 | Traefik Dashboard (Web UI)                | Host-local only (`127.0.0.1:8081`)                                      |
| 8082 | M3TAL Dashboard (Python/Flask container)  | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik           |

---

### Firewall Note

If your server has a firewall (e.g., UFW) enabled, you may need to open ports to allow external access, especially for Traefik if you plan to expose services to the internet or access the M3TAL Dashboard externally in `local` mode.

For Traefik on port 80:

```bash
sudo ufw allow 80
```

If you plan to use HTTPS (port 443) with Traefik:

```bash
sudo ufw allow 443
```

If you configured `DASHBOARD_EXPOSE_MODE=local` and wish to access it from another machine on your local network:

```bash
sudo ufw allow 8082
```

---

### Service Management

The M3TAL API daemon runs as a systemd service, managing the Docker environment and the state database.

To check the status of the M3TAL API daemon:

```bash
systemctl status m3tal-api
```

To view the live logs of the M3TAL API daemon:

```bash
journalctl -u m3tal-api -f
```