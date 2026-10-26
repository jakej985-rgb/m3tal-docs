# M3TAL Getting Started Guide

This guide provides the necessary steps to set up M3TAL for the first time.

---

### 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and versions:

```bash
docker --version && docker compose version
```

---

### 2. Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the `m3tal` CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

### 3. Run the Configuration Wizard

Initialize the M3TAL configuration with the interactive wizard. This will create or update the primary configuration file at `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several configuration values:

-   **`DOMAIN`**: The base domain for Traefik routing (e.g., `example.com`). If not using a custom domain and only accessing services via IP, `localhost` is a suitable default.
-   **`DASHBOARD_EXPOSE_MODE`**:
    -   `local`: Exposes the M3TAL Dashboard directly on a host port (default `8082`). Access via `http://YOUR_IP:8082`. No Traefik configuration is required for the dashboard itself in this mode.
    -   `traefik`: Exposes the M3TAL Dashboard via Traefik on a subdomain (e.g., `dash.YOUR_DOMAIN`). Requires Traefik to be running.
-   **`DASHBOARD_PORT`**: The host port to bind for the dashboard when `DASHBOARD_EXPOSE_MODE` is `local`. Default is `8082`.
-   **`PUID` / `PGID`**: User ID and Group ID for Docker containers to ensure correct file permissions on mounted volumes. These typically match your user's UID and GID (e.g., `1000`).
-   **`TZ`**: Your local timezone (e.g., `America/Denver`).
-   **`ADMIN_PASSWORD`**: The password for the `admin` user to log into the M3TAL Dashboard.
-   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for session management.
-   **`API_TOKEN`**: A token for API authentication.

---

### 4. Start the Routing Stack

Start all Docker Compose stacks defined by `*-compose.yml` files in the `/docker/` directory. This primarily includes the Traefik reverse proxy and any optional routing services like Cloudflared.

```bash
m3tal up
```

This command invokes `docker compose up -d` for all recognized compose files, ensuring your routing infrastructure (including Traefik on port 80) is operational.

---

### 5. Start the M3TAL Dashboard

Deploy the M3TAL Dashboard container. This command handles downloading the latest dashboard compose files and starts the container based on your `DASHBOARD_EXPOSE_MODE` setting.

```bash
m3tal dash up
```

This process:
1.  Downloads the necessary `m3tal-compose.yml` and override files from GitHub.
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Applies the corresponding Docker Compose override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).
4.  Pulls the `m3tal-dashboard` Docker image (if not already present).
5.  Starts the `m3tal-dashboard` container.

---

### 6. Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard using the appropriate address based on your `DASHBOARD_EXPOSE_MODE`:

-   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your M3TAL host)
    or `http://localhost:8082` if accessing from the host machine.

-   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the `DOMAIN` you configured in the wizard).
    *(Note: Traefik must be running for this mode to work.)*

---

### 7. Log In to the Dashboard

The default username for the dashboard is `admin`.
The password is the `ADMIN_PASSWORD` you set during the configuration wizard (e.g., `admin_pass`).

To change the dashboard password for the `admin` user after initial setup, use:

```bash
sudo m3tal dashpass
```

---

### Filesystem Contract

The following paths are central to the M3TAL ecosystem:

-   **/etc/m3tal/.env**: The primary configuration file, managed by `m3tal config wizard`. This file stores all environment variables used by M3TAL.
-   **/var/lib/m3tal/state.db**: The SQLite database where the M3TAL API daemon stores its operational state. This file is automatically created by the API.
-   **/opt/m3tal/stack/**: The canonical directory for M3TAL's internal Docker Compose files and Traefik configuration.
-   **/docker**: A symlink that points to `/opt/m3tal/stack/`. This is the user-facing path where all Docker Compose files (`*-compose.yml`) should be placed for `m3tal up` to discover them.
-   **/docker/users.json**: Stores the M3TAL Dashboard user credentials. Managed by `m3tal dashpass`.

---

### Port Map

M3TAL utilizes the following ports by default:

| Port | Service                                | Access                 |
| :--- | :------------------------------------- | :--------------------- |
| 80   | Traefik HTTP entry point               | Public (traefik mode)  |
| 8080 | M3TAL API daemon (Go binary)           | Host-local             |
| 8081 | Traefik dashboard (Traefik's internal) | Host-local only        |
| 8082 | M3TAL Dashboard (Python/Flask)         | Direct port (local mode) or via Traefik (traefik mode) |

---

### Firewall Configuration

If you are exposing Traefik (port 80) to the public internet, you may need to adjust your firewall. For `ufw` (Uncomplicated Firewall), execute:

```bash
sudo ufw allow 80
```

---

### Service Management

The M3TAL API daemon runs as a systemd service. Use `systemctl` to manage it:

-   **Check the status of the M3TAL API daemon:**
    ```bash
    systemctl status m3tal-api
    ```

-   **View real-time logs for the M3TAL API daemon:**
    ```bash
    journalctl -u m3tal-api -f
    ```