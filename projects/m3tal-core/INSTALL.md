# Getting Started with M3TAL

This guide provides a complete, step-by-step setup for first-time users of the M3TAL Ecosystem.

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify your Docker installation by running:

```bash
docker --version && docker compose version
```

## 2. Install M3TAL via APT

Execute the following commands in your terminal to install the M3TAL CLI binary and API daemon:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

This installs the `/usr/bin/m3tal` CLI binary and sets up the `m3tal-api.service` systemd daemon.

## 3. Run the Configuration Wizard

Initialize M3TAL's core configuration using the wizard:

```bash
sudo m3tal config wizard
```

The wizard will guide you through essential settings. Here's an explanation of key prompts:

*   **DASHBOARD_EXPOSE_MODE (local/traefik)**:
    *   `local` (default): The M3TAL Dashboard will be directly exposed on a specific port (default 8082) on your host machine. This is ideal for quick setups, LAN-only access, or when you don't use Traefik for domain-based routing.
    *   `traefik`: The M3TAL Dashboard will be accessible via a domain (e.g., `dash.yourdomain.com`) through the Traefik reverse proxy. This requires Traefik to be running and assumes you have DNS configured for your domain.
*   **DOMAIN (e.g., example.com)**: If you selected `traefik` mode for the dashboard, this defines the base domain for your services (e.g., `dash.example.com`, `api.example.com`). Default is `localhost`.
*   **PUID / PGID**: These define the User ID and Group ID that containers will run as, primarily for file permissions. Using your current user's PUID/PGID (typically `1000`) is recommended to avoid permission issues with mounted volumes.
*   **TZ (Timezone, e.g., America/New_York)**: Sets the timezone for containers.
*   **DASHBOARD_PORT (default 8082)**: The port on your host machine where the dashboard will be accessible if `DASHBOARD_EXPOSE_MODE` is `local`.
*   **Storage Paths (BASE_STORAGE_PATH, CONFIG_PATH, MEDIA_PATH, DOWNLOADS_PATH)**: These define the base directories for M3TAL's data, configurations, and user-managed media/download folders. Defaults usually point to subdirectories under `/mnt`.
*   **DASHBOARD_SECRET, API_TOKEN, ADMIN_PASSWORD**: These are critical security credentials. **Change these from their defaults immediately.** The `ADMIN_PASSWORD` is the initial password for the M3TAL Dashboard.

The wizard creates or updates the `/etc/m3tal/.env` file, which is the primary configuration file for your M3TAL ecosystem.

## 4. Start the Routing Stack (Traefik)

The M3TAL ecosystem uses Traefik as its default reverse proxy for exposing services. Start Traefik and any other core components:

```bash
m3tal up
```

This command executes `docker compose` operations across all `*-compose.yml` files located in the `/docker/` directory. By default, it will start the `routing-compose.yml` which deploys Traefik and optionally Cloudflared. Traefik will listen on port 80 and route traffic to other services based on Docker labels and dynamic configuration. It also routes `api.YOUR_DOMAIN` to the M3TAL API daemon running on port 8080.

The `/docker` directory is a symlink to `/opt/m3tal/stack/`, which is the canonical location for M3TAL's compose files.

## 5. Start the M3TAL Dashboard

Deploy the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command pulls the `m3tal-dashboard` Docker image and starts its container. It intelligently applies the correct configuration based on the `DASHBOARD_EXPOSE_MODE` set in `/etc/m3tal/.env`:

*   If `DASHBOARD_EXPOSE_MODE=local`, it uses `m3tal-compose.local.yml` to bind the dashboard container's port 8082 directly to your host's configured `DASHBOARD_PORT` (default 8082).
*   If `DASHBOARD_EXPOSE_MODE=traefik`, it uses `m3tal-compose.traefik.yml` to add Traefik labels, allowing Traefik to route requests for `dash.YOUR_DOMAIN` to the dashboard container's internal port 8082.

The dashboard communicates with the M3TAL API daemon (which runs on the host at `http://host.docker.internal:8080`).

## 6. Open the M3TAL Dashboard in Your Browser

Access the M3TAL Dashboard using your web browser:

*   **If `DASHBOARD_EXPOSE_MODE` is `local`:**
    Navigate to `http://YOUR_HOST_IP:8082` (replace `YOUR_HOST_IP` with the actual IP address of your server, or use `localhost` if accessing from the same machine).
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in the wizard). Traefik must be running for this mode to work.

## 7. Log In to the Dashboard

When prompted, log in to the M3TAL Dashboard.

*   **Default Credentials:** The default username is `admin`, and the initial password is the value set for `ADMIN_PASSWORD` during the `m3tal config wizard` (default `admin_pass` if not changed).
*   **Changing Credentials:** It is highly recommended to change the default password. You can do this using the CLI:
    ```bash
    sudo m3tal dashpass
    ```
    This command updates the credential store located at `/docker/users.json`.

---

## Filesystem Contract

The M3TAL ecosystem adheres to a clear filesystem contract for its essential files and directories:

| Path                        | Purpose                                                                                |
| :-------------------------- | :------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.                          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon.            |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains M3TAL's core Docker Compose files and Traefik config. |
| `/docker`                   | Symlink that points to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | M3TAL Dashboard credential store. Managed by `m3tal dashpass`.                         |

## Port Table

These are the default ports used by M3TAL and its components:

| Port | Service               | Access                                                         |
| :--- | :-------------------- | :------------------------------------------------------------- |
| 80   | Traefik HTTP entry point | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services exposed) |
| 8080 | M3TAL API daemon (Go) | Host-local (accessed by containers via `host.docker.internal`) |
| 8081 | Traefik dashboard     | Host-local only (`127.0.0.1:8081`)                               |
| 8082 | M3TAL Dashboard       | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Note

If you are exposing Traefik (e.g., for domain-based routing or public access), you will need to allow incoming connections on port 80 (and optionally 443 for HTTPS) in your firewall. For `ufw` (Uncomplicated Firewall):

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using standard systemctl and journalctl commands:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```