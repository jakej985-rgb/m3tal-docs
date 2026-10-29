```markdown
# GET_STARTED.md: M3TAL Ecosystem Setup Guide

This guide provides a step-by-step procedure for first-time M3TAL users to set up their system.

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

Initialize your M3TAL environment and configure essential settings using the interactive wizard:

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values to populate `/etc/m3tal/.env`. Key prompts include:

*   **`DOMAIN`**: This is your primary domain name (e.g., `example.com`). If you plan to access services via subdomains like `dash.example.com`, set this accordingly. For local testing without a domain, `localhost` is a valid entry.
*   **`DASHBOARD_EXPOSE_MODE`**:
    *   Enter `local` if you want to access the M3TAL Dashboard directly via an IP address and port (e.g., `http://YOUR_IP:8082`). This is recommended for first-time users or LAN-only setups.
    *   Enter `traefik` if you intend to route dashboard access through the Traefik reverse proxy via a domain (e.g., `http://dash.YOUR_DOMAIN`). This requires Traefik to be running.
*   **`PUID`** and **`PGID`**: These are your user and group IDs, respectively. Docker containers will use these IDs for file permissions within bind mounts. Typically, `1000` is the default for the first non-root user. You can find yours with `id -u` and `id -g`.
*   **`TZ`**: Your timezone (e.g., `America/New_York`). This ensures logs and scheduled tasks operate with correct timestamps.
*   **`DASHBOARD_SECRET`**: A strong, random string used for secure communication within the dashboard. The wizard will generate one if left blank.
*   **`ADMIN_PASSWORD`**: The default password for the M3TAL Dashboard's `admin` user. Ensure you set a strong password.

### Step 4: Start the Routing Stack

Start the core routing components, including Traefik. This command reads all `*-compose.yml` files located in the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`) and brings up their defined services. This specifically includes `routing-compose.yml` for Traefik.

```bash
m3tal up
```

This command will:
*   Create an external Docker network named `proxy` if it doesn't exist, which all M3TAL services will join for internal communication.
*   Start the Traefik reverse proxy container, listening on port 80 (and 443 if configured).
*   Optionally start `cloudflared` if configured in the wizard.

### Step 5: Start the M3TAL Dashboard

Deploy and start the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command performs the following actions:
1.  Downloads the latest M3TAL Dashboard compose files (`m3tal-compose.yml` and its overrides) from GitHub.
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env` to determine the dashboard's access method.
3.  Starts the `m3tal-dashboard` container, applying the appropriate `local` or `traefik` override.
    *   If `DASHBOARD_EXPOSE_MODE=local`, a direct port binding (`8082:8082`) is added.
    *   If `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels are added to route `dash.YOUR_DOMAIN` to the dashboard.

### Step 6: Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` setting from Step 3:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    `http://YOUR_SERVER_IP:8082` (Replace `YOUR_SERVER_IP` with the actual IP address of your server).
    Alternatively, if accessing from the server itself: `http://localhost:8082`

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    `http://dash.YOUR_DOMAIN` (Replace `YOUR_DOMAIN` with the domain configured in your `.env` file).
    *This requires Traefik to be running (Step 4).*

### Step 7: Log In to the Dashboard

At the login screen:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the configuration wizard (default is `admin_pass` if not changed).

**Change Dashboard Password**:
It is strongly recommended to change the default password immediately. You can do this via the CLI:

```bash
sudo m3tal dashpass
```

---

## Filesystem Contract

The M3TAL system relies on specific paths for configuration and data storage:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for M3TAL's internal Docker Compose files.     |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. User-facing path for stack operations and adding custom compose files. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.           |

## Port Table

The following ports are used by M3TAL components:

| Port | Service               | Access                                                 |
| :--- | :-------------------- | :----------------------------------------------------- |
| 80   | Traefik HTTP          | Public (if `TRAEFIK_WEB_PORT` is exposed)              |
| 8080 | M3TAL API daemon (Go) | Host-local only (accessed by dashboard and Traefik)    |
| 8081 | Traefik Dashboard     | Host-local only (e.g., `http://127.0.0.1:8081`)        |
| 8082 | M3TAL Dashboard       | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Note

If your server has a firewall enabled (e.g., `ufw`), you may need to open ports to allow external access:

*   To expose Traefik (for `dash.YOUR_DOMAIN` or other services via port 80):
    ```bash
    sudo ufw allow 80/tcp
    ```
*   To expose the M3TAL Dashboard directly (if `DASHBOARD_EXPOSE_MODE=local`):
    ```bash
    sudo ufw allow 8082/tcp
    ```
*   Remember to reload or enable your firewall after making changes:
    ```bash
    sudo ufw reload # or sudo ufw enable
    ```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view its logs using `systemctl` and `journalctl`:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## M3TAL Dashboard Access Modes

The dashboard's accessibility is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`, configured during the wizard.

### Mode 1: Local (Default)
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: M3TAL uses `m3tal-compose.local.yml` to add a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container.
*   **Access**: `http://YOUR_SERVER_IP:8082` or `http://localhost:8082`.
*   **Requires**: No Traefik configuration needed for dashboard access.
*   **Best for**: LAN-only setups, initial testing, or when you don't use a domain.

### Mode 2: Traefik
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: M3TAL uses `m3tal-compose.traefik.yml` to apply Traefik labels to the dashboard container. These labels instruct Traefik to route requests for `dash.${DOMAIN}` to the dashboard's internal port 8082.
*   **Access**: `http://dash.YOUR_DOMAIN`.
*   **Requires**: Traefik must be running via `m3tal up` and configured to listen on port 80.
*   **Best for**: Domain-based access, integrating with other services behind Traefik.

---

## Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** for container orchestration. These are fundamental dependencies for the system's operation.

*   The `m3tal up` command executes `docker compose` operations against all `*-compose.yml` files located within the `/docker/` directory (e.g., `routing-compose.yml`). This orchestrates your entire M3TAL stack and any custom user stacks you deploy.
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container:
    1.  It downloads the necessary Docker Compose configuration for the dashboard.
    2.  It reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  It then starts the dashboard container, applying the correct compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) to ensure it's exposed according to your configuration.

---

## Deployment Lifecycle: Adding User Stacks

To add new Docker Compose-based services to your M3TAL ecosystem:

1.  **Place Compose File**: Create your Docker Compose file (e.g., `my-app-compose.yml`) and place it in the `/docker/` directory.
2.  **Configure Variables**: Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` to add them.
3.  **Start Stack**: Run `m3tal up` to deploy and start your new services alongside the existing M3TAL components. Traefik will automatically discover and route traffic to services with appropriate Docker labels.
```