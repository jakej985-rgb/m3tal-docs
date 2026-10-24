```markdown
# M3TAL - Getting Started Guide

This guide provides step-by-step instructions to install and run M3TAL for the first time.

## 1. Prerequisites

Before proceeding, ensure the following software is installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

Verify their installation by running:

```bash
docker --version && docker compose version
```

If these are not installed, please refer to the official Docker documentation for installation instructions.

## 2. Install M3TAL via APT

Install M3TAL by adding our APT repository and installing the package.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

The configuration wizard will guide you through setting up essential M3TAL parameters.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   **`local`** (default): Exposes the dashboard directly via a port on your host machine. This is recommended for initial setup and LAN-only access. Access via `http://YOUR_IP:8082`.
    *   **`traefik`**: Exposes the dashboard through Traefik, allowing access via a domain name (e.g., `dash.YOUR_DOMAIN`). This requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name for your M3TAL setup. If using `traefik` mode, this is used to construct internal routing rules (e.g., `dash.DOMAIN`). Defaults to `localhost`.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. Defaults to `8082`.
*   **`TRAEFIK_WEB_PORT`**: The public-facing port for Traefik's HTTP entrypoint. Defaults to `80`.
*   **`PUID` / `PGID`**: The User ID (UID) and Group ID (GID) that the Docker containers will run as. It's recommended to use your current user's IDs (find with `id -u` and `id -g`).
*   **`BASE_STORAGE_PATH`**: The root directory for M3TAL's data and configuration. Defaults to `./data` within the M3TAL installation directory.

## 4. Start the Routing Stack (Traefik)

This command starts the Traefik reverse proxy, which is essential for routing external traffic to your M3TAL services, especially if you're using `traefik` mode for the dashboard.

```bash
m3tal up
```

This command reads all `.yml` files in the `/docker/` directory and starts the corresponding Docker Compose services. This includes Traefik.

## 5. Start the Dashboard

This command specifically pulls the latest M3TAL dashboard image and starts its container.

```bash
m3tal dash up
```

This command will download the necessary Docker image for the dashboard and start the container, applying the configuration based on your `DASHBOARD_EXPOSE_MODE` setting.

## 6. Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard URL:

*   If `DASHBOARD_EXPOSE_MODE` is set to **`local`**:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. If running locally, you can also use `http://localhost:8082`)

*   If `DASHBOARD_EXPOSE_MODE` is set to **`traefik`** (and Traefik is running):
    `http://dash.YOUR_DOMAIN`
    (Replace `YOUR_DOMAIN` with the domain you configured. Ensure DNS is set up correctly if needed.)

## 7. Log In to the Dashboard

Upon accessing the dashboard, you will be presented with a login screen.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: `admin_pass`

**Important:** It is highly recommended to change the default password immediately after your first login. You can do this via the command line:

```bash
sudo m3tal dashpass
```

This command will prompt you to set a new password for the dashboard.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem layout for its configuration and state.

| Path                      | Purpose                                                    | Managed By                |
| :------------------------ | :--------------------------------------------------------- | :------------------------ |
| `/etc/m3tal/.env`         | Primary M3TAL configuration file.                          | `m3tal config wizard`     |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's operational state.         | M3TAL API daemon          |
| `/opt/m3tal/stack/`       | Canonical directory for M3TAL's Docker Compose stacks.     | M3TAL installation        |
| `/docker`                 | Symlink pointing to `/opt/m3tal/stack/`. User-facing path for Docker Compose files. | M3TAL installation        |
| `/docker/users.json`      | Stores dashboard credentials.                              | `m3tal dashpass`          |

## Port Map

| Port | Service / Protocol | Access Method                               |
| :--- | :----------------- | :------------------------------------------ |
| 80   | Traefik (HTTP)     | Public (if `traefik` mode is active)        |
| 8080 | M3TAL API Daemon   | Host-local                                  |
| 8081 | Traefik Dashboard  | Host-local only (for Traefik management)    |
| 8082 | M3TAL Dashboard    | Direct port (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If Traefik is exposed to the public internet (port 80), ensure your firewall allows incoming connections on that port. If you are using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```
    (Press `Ctrl+C` to exit log viewing.)
```