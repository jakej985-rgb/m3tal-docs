```markdown
# GET_STARTED

This guide provides a complete, step-by-step setup for first-time M3TAL users.

---

## 1. Prerequisites

M3TAL requires a functional Docker environment.
Docker Engine and Docker Compose V2 must be installed on your system.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Example expected output:
```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.0-desktop.1
```

## 2. Install M3TAL via APT

Install the M3TAL CLI binary and API daemon using the APT package manager:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the M3TAL configuration wizard to set up essential environment variables. This creates the primary configuration file at `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Pay close attention to these key prompts:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   `local` (default): The M3TAL Dashboard will be directly accessible via `http://YOUR_IP:8082`. No Traefik configuration is required for the dashboard. This is recommended for first-time setups and LAN-only access.
    *   `traefik`: The M3TAL Dashboard will be exposed via the Traefik reverse proxy at `http://dash.YOUR_DOMAIN`. This requires Traefik to be running and a `DOMAIN` to be configured.
*   **`DOMAIN`**: Used for Traefik-based routing (e.g., `dash.DOMAIN`, `api.DOMAIN`). If you selected `traefik` for `DASHBOARD_EXPOSE_MODE`, set this to your desired domain (e.g., `example.com`). If you selected `local`, `localhost` is generally sufficient.
*   **`PUID` (User ID)** and **`PGID` (Group ID)**: These determine the user and group Docker containers will run as, ensuring proper file permissions for mounted volumes. Typically `1000` for the first non-root user. You can find your current IDs with `id -u` and `id -g`.
*   **`TZ` (Time Zone)**: Set your local time zone (e.g., `America/New_York`). This ensures logs and scheduled tasks are aligned with your local time.
*   **`DASHBOARD_SECRET`**: A strong, random string used for session management in the dashboard. Change this from the default.
*   **`ADMIN_PASSWORD`**: The initial password for the M3TAL Dashboard. Change this from the default.

## 4. Start the Routing Stack

M3TAL uses Traefik as a reverse proxy for exposing services by domain name. The `m3tal up` command starts all Docker Compose files located in the `/docker/` directory, including the `routing-compose.yml` which deploys Traefik and optionally Cloudflared.

```bash
m3tal up
```

This command will:
*   Create the `proxy` Docker network if it doesn't exist.
*   Pull the `traefik` Docker image (and `cloudflare/cloudflared` if enabled).
*   Start the Traefik and Cloudflared containers. Traefik will listen on port 80.

## 5. Start the Dashboard

The M3TAL Dashboard is a Docker container. The `m3tal dash up` command manages its deployment:

```bash
m3tal dash up
```

This command will:
1.  Download the latest `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
2.  Read the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Start the `m3tal-dashboard` container, applying the appropriate override:
    *   If `DASHBOARD_EXPOSE_MODE=local`, it applies `m3tal-compose.local.yml` to bind port `8082` directly to the host.
    *   If `DASHBOARD_EXPOSE_MODE=traefik`, it applies `m3tal-compose.traefik.yml` to add Traefik labels, allowing Traefik to route `dash.DOMAIN` to the dashboard.
4.  The dashboard container will pull its image (`ghcr.io/jakej985-rgb/m3tal-godash:debug`) and start. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.

## 6. Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard using the address appropriate for your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your server). You can also use `http://localhost:8082` if accessing from the server itself.

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured in step 3). Ensure your DNS records are correctly configured to point `dash.DOMAIN` to your server's IP address.

## 7. Log In

Use the credentials configured during the `m3tal config wizard` to log in:

*   **Username**: `admin`
*   **Password**: The value you set for `ADMIN_PASSWORD` during the wizard.

**Important:** It is highly recommended to change the dashboard password immediately if you used the default value during setup.

To change the dashboard password:

```bash
sudo m3tal dashpass
```
You will be prompted to enter and confirm a new password. This command updates the `/docker/users.json` file.

---

## Filesystem Contract

M3TAL uses a specific filesystem layout for its operation and configuration:

*   `/etc/m3tal/.env`: The primary configuration file, managed by `m3tal config wizard`.
*   `/var/lib/m3tal/state.db`: The SQLite state database used by the M3TAL API daemon. This file is auto-created.
*   `/opt/m3tal/stack/`: The canonical directory where M3TAL stores its core Docker Compose files (e.g., `routing-compose.yml`, `m3tal-compose.yml`).
*   `/docker`: A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker stack operations. When you place a `*-compose.yml` file here, M3TAL detects and manages it.
*   `/docker/users.json`: Stores M3TAL Dashboard user credentials. Managed by `m3tal dashpass`.

## Core M3TAL Components & Architecture

*   **CLI binary** (`/usr/bin/m3tal`): The unified Go binary, installed via APT. This is your single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a `systemd` service on host port 8080. It manages Docker interactions, the state database, and provides API routes for the dashboard and other services.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port 8082. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services by domain name on host port 80. It uses Docker labels and a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for zero-configuration internet access, part of the routing stack.

### Docker / Compose Runtime Explanation

M3TAL relies on **Docker Engine** and **Docker Compose V2** as hard dependencies.
*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files present in the `/docker/` directory. This includes the `routing-compose.yml` (for Traefik) and any other user-defined stacks.
*   The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It ensures the dashboard's compose files are up-to-date by downloading them from GitHub, then starts the container with the correct configuration based on `DASHBOARD_EXPOSE_MODE` (using `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` overrides).

## Dashboard Access Modes

The dashboard's accessibility is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Mode 1: local (Default)
*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the `m3tal-compose.local.yml` override, which directly binds the dashboard's internal port 8082 to the host's `DASHBOARD_PORT` (default 8082).
*   **Access via**: `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requires**: No Traefik setup for dashboard access.
*   **Best for**: LAN-only setups, initial testing, users who prefer direct port access.

### Mode 2: traefik
*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. These labels instruct Traefik to route `dash.${DOMAIN}` to the dashboard on its internal port 8082.
*   **Access via**: `http://dash.DOMAIN` (requires Traefik to be running via `m3tal up`).
*   **Requires**: Traefik (routing stack) running, and `DOMAIN` configured in `/etc/m3tal/.env`.
*   **Best for**: Domain-based access, integrating the dashboard with other services behind a reverse proxy.

## Port Map

| Port | Service                               | Access                                    |
| :--- | :------------------------------------ | :---------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (if Traefik is exposed)            |
| 8080 | M3TAL API daemon (Go)                 | Host-local only                           |
| 8081 | Traefik dashboard (admin interface)   | Host-local only (e.g., `http://localhost:8081`) |
| 8082 | M3TAL Dashboard container             | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Configuration

If you have a firewall enabled (e.g., `ufw`), you may need to allow traffic on relevant ports.

To expose Traefik (and thus any services it routes, like `dash.DOMAIN`):

```bash
sudo ufw allow 80/tcp
```

If you are using `DASHBOARD_EXPOSE_MODE=local` and want to access the dashboard from another machine on your network:

```bash
sudo ufw allow 8082/tcp
```

## Service Management (M3TAL API Daemon)

The M3TAL API daemon runs as a `systemd` service named `m3tal-api`.

*   **Check its status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

---

## Day 2 Operations: Deploying User Stacks

To add your own Docker Compose applications to the M3TAL ecosystem:

1.  **Place your compose file:** Create or copy your Docker Compose file (e.g., `my-app-compose.yml`) into the `/docker/` directory.
    ```bash
    sudo cp /path/to/my-app-compose.yml /docker/
    ```
2.  **Set required variables:** If your compose file uses environment variables, ensure they are defined in `/etc/m3tal/.env`. You can edit the file directly or use the M3TAL CLI:
    ```bash
    sudo m3tal config set MY_APP_VAR my_value
    ```
3.  **Start all stacks:** Run `m3tal up` to deploy all compose files found in `/docker/`, including your new stack.
    ```bash
    m3tal up
    ```

M3TAL will automatically discover and manage your `my-app-compose.yml` along with its core services.
```