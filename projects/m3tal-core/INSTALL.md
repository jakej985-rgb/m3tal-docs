# M3TAL Getting Started Guide

This document provides a step-by-step guide to installing and configuring M3TAL for the first time.

---

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation and versions:

```bash
docker --version && docker compose version
```

## 2. Install M3TAL via APT

Execute the following commands to install the M3TAL CLI and API daemon:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

Initialize M3TAL's primary configuration file (`/etc/m3tal/.env`) by running the interactive wizard:

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several configuration values. Explanation of key prompts:

*   **`DOMAIN`**: (Default: `localhost`)
    *   The primary domain M3TAL services will use. For local setups without a domain, `localhost` is appropriate. If you plan to expose services via Traefik (e.g., `dash.example.com`), set this to your domain (e.g., `example.com`).
*   **`DASHBOARD_EXPOSE_MODE`**: (Default: `local`)
    *   Determines how the M3TAL Dashboard is exposed.
        *   `local`: The dashboard container will bind directly to a host port (default `8082`). Access via `http://YOUR_IP:8082`. This mode does not require Traefik to be running.
        *   `traefik`: The dashboard will be accessible via Traefik as `http://dash.YOUR_DOMAIN`. This mode requires Traefik to be running (`m3tal up`) and your `DOMAIN` to be correctly configured.
*   **`DASHBOARD_PORT`**: (Default: `8082`)
    *   The host port used to access the dashboard when `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **`TRAEFIK_WEB_PORT`**: (Default: `80`)
    *   The host port Traefik will listen on for HTTP traffic. This is typically port 80 for public access.
*   **`PUID` / `PGID`**: (Default: `1000`)
    *   The User ID (PUID) and Group ID (PGID) used by containers to interact with the host filesystem. Use `id -u` and `id -g` to find your current user's IDs.
*   **`TZ`**: (Default: `America/Denver`)
    *   Your timezone. This affects logging timestamps and container clocks.
*   **`BASE_STORAGE_PATH`**: (Default: `/mnt`)
    *   The root directory for M3TAL's data. User-defined stacks often mount subdirectories of this path for persistent storage (e.g., `/mnt/data`, `/mnt/config`).
*   **`DASHBOARD_SECRET`**: (Default: `change_me_immediately`)
    *   A secret key used by the M3TAL Dashboard. **Change this to a strong, unique value.**
*   **`API_TOKEN`**: (Default: `change_me_api_token`)
    *   An API token used for internal communication. **Change this to a strong, unique value.**
*   **`ADMIN_PASSWORD`**: (Default: `admin_pass`)
    *   The default password for the M3TAL Dashboard's `admin` user. **Change this to a strong, unique value immediately.**

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. This includes the `routing-compose.yml` file which defines the Traefik reverse proxy.

```bash
m3tal up
```
This command will bring up the Traefik gateway, which typically listens on port 80 for incoming HTTP requests. If you configured `DASHBOARD_EXPOSE_MODE=traefik` in the wizard, Traefik is required for dashboard access.

## 5. Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It pulls the necessary Docker image and starts the container with the correct configuration based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

```bash
m3tal dash up
```
This command downloads the latest dashboard compose files (`m3tal-compose.yml` and its overrides) and starts the `m3tal-dashboard` container.

## 6. Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` configuration:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    *   Go to: `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your M3TAL host).
    *   If you are accessing from the same machine, you can use `http://localhost:8082`.
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    *   Go to: `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured during the wizard, e.g., `http://dash.example.com`).
    *   Ensure Traefik is running (`m3tal up`) and your DNS records are configured to point `dash.YOUR_DOMAIN` to your M3TAL host's IP address.

## 7. Log In

The default login credentials for the dashboard are:

*   **Username:** `admin`
*   **Password:** The value you set for `ADMIN_PASSWORD` during the `m3tal config wizard` (default: `admin_pass`).

**It is critical to change this password immediately.** You can do so using the CLI:

```bash
sudo m3tal dashpass admin new_strong_password
```
Replace `new_strong_password` with your desired secure password.

---

## Filesystem Contract

M3TAL establishes a clear filesystem contract for its operations and user stacks:

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary M3TAL configuration file. Managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's operational state. Auto-created.       |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal stack files (compose files, Traefik config). |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for adding and managing custom Docker Compose stacks. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.               |

## Port Table

The following ports are used by M3TAL components:

| Port | Service                                | Access                                             |
| :--- | :------------------------------------- | :------------------------------------------------- |
| 80   | Traefik HTTP entry point               | Public (if Traefik is enabled and exposed)         |
| 8080 | M3TAL API daemon (Go binary)           | Host-local only (accessed by other M3TAL components) |
| 8081 | Traefik Dashboard                      | Host-local only (internal Traefik administration)  |
| 8082 | M3TAL Dashboard (Python/Flask container) | Direct port (if `local` mode) or via Traefik (if `traefik` mode) |

## Firewall Note

If Traefik is exposed on port 80 and your host machine has a firewall enabled (e.g., `ufw`), you must allow traffic on this port for public access:

```bash
sudo ufw allow 80/tcp
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view its logs using standard `systemctl` and `journalctl` commands:

*   **Check service status:**
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

You have successfully set up M3TAL. You can now proceed to deploy additional services and stacks by placing their Docker Compose files into the `/docker/` directory and running `m3tal up`.