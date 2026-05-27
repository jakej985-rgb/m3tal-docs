# M3TAL Ecosystem Setup Guide

This document details the steps required to install and configure the M3TAL ecosystem for first-time users.

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

This installs the `m3tal` CLI binary to `/usr/bin/m3tal` and the `m3tal-api.service` systemd daemon.

### Step 3: Run the configuration wizard

The configuration wizard guides you through setting up the primary environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

You will be prompted for several configuration values. Here's what each prompt means:

*   **`DOMAIN`**: The base domain name for your services (e.g., `example.com`). If you don't have a domain or are setting up locally, `localhost` is the default. This is used by Traefik for routing.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard will be accessible.
    *   `local` (default): The dashboard will be directly exposed on port `8082` of your host. Access via `http://YOUR_IP:8082`. No Traefik configuration is needed for the dashboard itself in this mode.
    *   `traefik`: The dashboard will be exposed via Traefik, accessible at `http://dash.YOUR_DOMAIN`. This requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The direct port on the host to expose the dashboard if `DASHBOARD_EXPOSE_MODE` is set to `local`. Default is `8082`.
*   **`PUID`** and **`PGID`**: The User ID and Group ID that Docker containers will use. This ensures file permissions are correctly set for volumes mounted from your host. It is recommended to use the `id -u` and `id -g` of your current user.
*   **`TZ`**: Your timezone (e.g., `America/New_York`). Used by containers for correct time synchronization.
*   **`BASE_STORAGE_PATH`**: The base directory on your host where M3TAL will manage storage for all services. Defaults to `/mnt` if not changed. You can set this to `/var/lib/m3tal/data` or any other suitable path.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for session management. It is automatically generated during the wizard.
*   **`API_TOKEN`**: A token used to authenticate with the M3TAL API. It is automatically generated during the wizard.
*   **`ADMIN_PASSWORD`**: The initial password for the M3TAL Dashboard's default admin user. Change this immediately after setup.

### Step 4: Start the routing stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks defined in the `/docker/` directory. This includes Traefik, the primary reverse proxy for the M3TAL ecosystem.

```bash
m3tal up
```

This command initiates the routing stack (Traefik and optionally Cloudflared) as defined in `/docker/routing-compose.yml`, which will listen on port 80 (HTTP).

### Step 5: Start the dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container.

```bash
m3tal dash up
```

This command performs the following actions:
1.  Downloads the necessary `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container, applying the appropriate compose override (either for direct port exposure or Traefik routing).

### Step 6: Open browser

Once the dashboard container is running, access it via your web browser:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    Open `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your server).
*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    Open `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in `DOMAIN` variable during the wizard). This requires Traefik to be running via `m3tal up`.

### Step 7: Log in

The default login credentials for the M3TAL Dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during `sudo m3tal config wizard`.

**To change the admin password:**
After logging in, or at any time via CLI, you can change the dashboard password using:

```bash
sudo m3tal dashpass
```

This command will prompt you for a new password and update the `/docker/users.json` file.

---

### Filesystem Contract

The M3TAL ecosystem uses a defined filesystem structure for its operation:

| Path                        | Purpose                                                                                                                                                                                                                                                                       |
| :-------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file, containing environment variables for the M3TAL system and its services. Managed by `m3tal config wizard` and `m3tal config set`.                                                                                                                |
| `/var/lib/m3tal/state.db`   | The SQLite state database used by the M3TAL API daemon to store service states, configurations, and other operational data. This file is automatically created by the API daemon.                                                                                              |
| `/opt/m3tal/stack/`         | The canonical directory for M3TAL's internal Docker Compose files and Traefik dynamic configurations.                                                                                                                                                                           |
| `/docker`                   | A symbolic link that points to `/opt/m3tal/stack/`. This user-facing path is where all Docker Compose stack files (`*-compose.yml`) should be placed for `m3tal up` to discover and manage.                                                                                    |
| `/docker/users.json`        | The credential store for the M3TAL Dashboard. Contains hashed usernames and passwords. Managed by `m3tal dashpass`.                                                                                                                                                           |

### Port Table

The following network ports are used by M3TAL components:

| Port | Service                               | Access                                                                             |
| :--- | :------------------------------------ | :--------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point              | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services are public)           |
| 8080 | M3TAL API daemon (Go)                 | Host-local only; internal communication with dashboard and other M3TAL components. |
| 8081 | Traefik Dashboard                     | Host-local only (accessed via `http://127.0.0.1:8081` on the host).                |
| 8082 | M3TAL Dashboard                       | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `traefik` mode). |

### Firewall Note

If Traefik is exposed publicly on port 80, ensure your firewall allows incoming connections on that port. For systems using `ufw`, you can enable it with:

```bash
sudo ufw allow 80/tcp
```

### Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check the status of the M3TAL API daemon:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs of the M3TAL API daemon:**
    ```bash
    journalctl -u m3tal-api -f
    ```

### Next Steps: Deploying New Stacks

To add new services or applications to your M3TAL ecosystem:

1.  Create a Docker Compose file for your new stack (e.g., `my-app-compose.yml`).
2.  Place this file within the `/docker/` directory.
3.  Ensure any necessary environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
4.  Run `m3tal up` to start all services, including your new stack.