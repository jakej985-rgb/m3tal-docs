```markdown
# M3TAL Get Started Guide

This document provides a complete, step-by-step guide for first-time M3TAL users.

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

M3TAL is distributed via an APT repository for easy installation and updates.

Execute the following commands to install M3TAL:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the M3TAL configuration wizard to set up essential environment variables. This creates and populates `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values:

*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is used for Traefik routing. Defaults to `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly exposed on a host port (`8082` by default). Ideal for local-only access or when Traefik is not yet configured.
    *   `traefik`: The dashboard is exposed via Traefik using `dash.${DOMAIN}`. Requires Traefik to be running.
*   **`PUID` / `PGID`**: User and Group IDs for Docker containers to ensure correct file permissions. Defaults to `1000`.
*   **`TZ`**: Your system's timezone (e.g., `America/Denver`). Used by containers.
*   **`API_TOKEN`**: A secret token for the M3TAL API. Change this from the default for security.
*   **`DASHBOARD_SECRET`**: A secret key used by the M3TAL Dashboard for session management. Change this from the default for security.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user on the M3TAL Dashboard. Change this immediately.

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. This includes the `routing-compose.yml` which deploys Traefik, the reverse proxy gateway.

```bash
m3tal up
```

This command will download the necessary Docker images and start the containers defined in `/docker/routing-compose.yml` (which includes Traefik) and any other compose files placed in `/docker/`.

**Firewall Note:** If you are exposing Traefik to the internet (which is the default if using `traefik` expose mode or setting `TRAEFIK_WEB_PORT` to `80`), you may need to allow port 80 through your firewall:

```bash
sudo ufw allow 80
```

## 5. Start the M3TAL Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It pulls the latest dashboard image and starts it, applying the correct configuration based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

```bash
m3tal dash up
```

This command:
1.  Downloads the latest dashboard compose files from GitHub.
2.  Reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container with the appropriate Docker Compose override (either `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).

## 6. Access the M3TAL Dashboard

The method for accessing the dashboard depends on the `DASHBOARD_EXPOSE_MODE` you selected during the configuration wizard:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    Open your web browser and navigate to `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your M3TAL host, or use `http://localhost:8082` if accessing from the host itself).

*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    Open your web browser and navigate to `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured in the wizard, e.g., `http://dash.example.com`). This requires Traefik to be running (`m3tal up` must have been executed).

## 7. Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during `sudo m3tal config wizard`.

**To change the dashboard password:**

```bash
sudo m3tal dashpass
```
This command will prompt you to set a new password for the dashboard's `admin` user.

---

## Filesystem Contract

M3TAL adheres to a clear filesystem contract for its configuration and data.

| Path                         | Purpose                                                                |
| :--------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`    | SQLite state database. Auto-created by the M3TAL API daemon.           |
| `/opt/m3tal/stack/`          | Canonical directory for M3TAL-managed compose files and Traefik config.|
| `/docker`                    | Symlink to `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`         | M3TAL Dashboard credential store. Managed by `m3tal dashpass`.         |

## Port Table

The following ports are used by M3TAL components:

| Port | Service                    | Access                                          |
| :--- | :------------------------- | :---------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services exposed) |
| 8080 | M3TAL API daemon (Go)      | Host-local only (or via Traefik as `api.DOMAIN`) |
| 8081 | Traefik Dashboard (admin)  | Host-local only (`127.0.0.1:8081`)              |
| 8082 | M3TAL Dashboard (Python)   | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view logs using standard systemd commands:

*   **Check service status:**
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

---
```