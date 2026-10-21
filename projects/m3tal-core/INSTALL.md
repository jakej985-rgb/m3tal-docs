# docs/GET_STARTED.md

## Getting Started with M3TAL

This guide provides a complete, step-by-step setup for first-time M3TAL users.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify your installation:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

Execute the following commands to install the M3TAL CLI and API daemon:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up your primary M3TAL environment variables.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for the following information:

*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is used for Traefik routing. Defaults to `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly accessible on `http://YOUR_IP:8082`. No Traefik configuration is needed for the dashboard itself.
    *   `traefik`: The dashboard is accessible via `http://dash.YOUR_DOMAIN` through Traefik. Requires Traefik to be running.
*   **`PUID` (User ID) & `PGID` (Group ID)**: The user and group ID that Docker containers will run as inside the host system. This is crucial for file permissions. You can find your current user's IDs with `id -u` and `id -g`. Defaults to `1000`.
*   **`TZ` (Timezone)**: Your desired timezone (e.g., `America/New_York`). Defaults to `America/Denver`.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for session management. A strong, random string is recommended. This will be automatically generated if left blank.
*   **`API_TOKEN`**: An authentication token for the M3TAL API. A strong, random string is recommended. This will be automatically generated if left blank.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user on the M3TAL Dashboard. Defaults to `admin_pass`. You will change this later.

### Step 4: Start the Routing Stack (Traefik)

The M3TAL routing stack consists of Traefik, the reverse proxy, and optionally Cloudflared for secure tunnels. This command starts all Docker Compose files located in `/docker/` (which is a symlink to `/opt/m3tal/stack/`).

```bash
m3tal up
```

This command initiates the containers defined in the core stack files, including `routing-compose.yml`, which deploys Traefik and Cloudflared. Traefik will typically bind to port 80 on your host system.

### Step 5: Start the Dashboard

This command specifically manages the M3TAL Dashboard container. It pulls the latest dashboard image, then starts the container using the appropriate configuration based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

```bash
m3tal dash up
```

### Step 6: Open Browser

Access the M3TAL Dashboard in your web browser:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    Open `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your server). If you are on the same machine, `http://localhost:8082` will also work.

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Open `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in the wizard, e.g., `http://dash.example.com`). Traefik must be running for this mode to work.

### Step 7: Log In

The default login credentials for the M3TAL Dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard (default is `admin_pass`).

**To change the dashboard password:**
Execute the following command on your server:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new password for the `admin` user.

---

## Filesystem Contract

M3TAL relies on specific file paths for its operation and configuration:

| Path                        | Purpose                                                                                                 |
| :-------------------------- | :------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`           | Primary configuration file. Contains environment variables for the M3TAL ecosystem. Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Stores M3TAL's internal state, such as service configurations. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all M3TAL-managed Docker Compose files and Traefik configuration.               |
| `/docker`                   | A symbolic link that points to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Contains encrypted user passwords for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Port Map

| Port | Service                | Access                                     |
| :--- | :--------------------- | :----------------------------------------- |
| 80   | Traefik HTTP entry point | Public (if `DASHBOARD_EXPOSE_MODE=traefik` and exposing other services) |
| 8080 | M3TAL API daemon (Go)  | Host-local only (accessed by other M3TAL components) |
| 8081 | Traefik dashboard      | Host-local only (for monitoring Traefik itself) |
| 8082 | M3TAL Dashboard        | Direct port (`local` mode) or via Traefik (`traefik` mode) |

### Firewall Note

If Traefik is exposed publicly (e.g., to handle domain-based routing for your services), you may need to open port 80 on your firewall:

```bash
sudo ufw allow 80/tcp
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view its logs using standard `systemctl` and `journalctl` commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the API daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```