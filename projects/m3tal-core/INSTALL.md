# GET_STARTED.md

This guide provides a step-by-step setup process for first-time M3TAL users.

## 1. Prerequisites

M3TAL requires a functional Docker environment.
Docker Engine and Docker Compose V2 must be installed and operational on your system.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Example output:

```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.3-desktop.1
```

## 2. Install M3TAL via APT

Install the M3TAL CLI binary and API daemon using the APT package manager.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, configure M3TAL using the interactive wizard. This sets up essential environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt for the following settings:

*   **`DOMAIN`**: The domain name to use for services exposed via Traefik (e.g., `example.com`). Defaults to `localhost`. This will be used to construct URLs like `dash.example.com` or `api.example.com`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly accessible via a port on the host machine (e.g., `http://YOUR_IP:8082`). No Traefik configuration is required for dashboard access in this mode.
    *   `traefik`: The dashboard is exposed via the Traefik reverse proxy using a subdomain (e.g., `http://dash.YOUR_DOMAIN`). Requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The port for direct access to the dashboard if `DASHBOARD_EXPOSE_MODE` is set to `local`. Defaults to `8082`.
*   **`PUID` / `PGID`**: The User ID and Group ID that containers will run as. This ensures proper file permissions for persistent volumes. Defaults to `1000` (often the first non-root user).
*   **`TZ`**: The timezone for containers. Defaults to `America/Denver`.
*   **`DASHBOARD_SECRET`**: A secret key used by the M3TAL Dashboard for session management.
*   **`ADMIN_PASSWORD`**: The default password for the `admin` user of the M3TAL Dashboard. Defaults to `admin_pass`.
*   **`API_TOKEN`**: An authentication token for the M3TAL API.
*   **`BASE_STORAGE_PATH`**: The base directory where M3TAL stores persistent data for services. Defaults to `./data`.

## 4. Start the Routing Stack

Start the core routing services, including Traefik. The `m3tal up` command processes all Docker Compose files located in the `/docker/` directory.

```bash
m3tal up
```

This command starts:
*   `routing-compose.yml`: Deploys the Traefik reverse proxy and optional Cloudflared tunnel, enabling domain-based routing for services.
*   Any other user-defined `*-compose.yml` files present in `/docker/`.

## 5. Start the Dashboard

Start the M3TAL Dashboard container. The `m3tal dash up` command performs the following actions:
1.  Downloads the latest dashboard Docker Compose configurations.
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container, applying the appropriate Compose override (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on the `DASHBOARD_EXPOSE_MODE`.

```bash
m3tal dash up
```

## 6. Open the Dashboard in a Browser

Access the M3TAL Dashboard based on the `DASHBOARD_EXPOSE_MODE` configured in Step 3:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    Open your web browser and navigate to:
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the IP address of your host machine)
    or `http://localhost:8082` (if accessing from the server itself).
    The port `8082` is the default `DASHBOARD_PORT` you configured.

*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    Open your web browser and navigate to:
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the `DOMAIN` you configured in the wizard).
    This mode requires Traefik to be running (via `m3tal up`) and correctly configured with DNS records pointing to your server.

## 7. Log In

Log in to the M3TAL Dashboard using the default credentials:

*   **Username**: `admin`
*   **Password**: The value you set for `ADMIN_PASSWORD` during the `m3tal config wizard` (default: `admin_pass`).

To change the dashboard password for the `admin` user:

```bash
sudo m3tal dashpass admin NEW_PASSWORD
```

Replace `NEW_PASSWORD` with your desired new password.

---

## Filesystem Contract

M3TAL utilizes specific directories and files for its operation:

| Path                        | Purpose                                                                                                 |
| :-------------------------- | :------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`           | Primary configuration file. Contains environment variables and is managed by `m3tal config wizard`.       |
| `/var/lib/m3tal/state.db`   | SQLite database used by the M3TAL API daemon to store its operational state. Auto-created.                |
| `/opt/m3tal/stack/`         | Canonical directory containing M3TAL's core Docker Compose files and Traefik dynamic configurations.      |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing Docker stacks and placing custom `*-compose.yml` files. |
| `/docker/users.json`        | Stores M3TAL Dashboard user credentials. Managed by `m3tal dashpass`.                                   |

## Port Table

M3TAL components use the following network ports:

| Port | Service                                | Access                                                                                                                              |
| :--- | :------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `80` | Traefik HTTP Entry Point               | Publicly exposed by Traefik for incoming HTTP traffic, used for routing to services via domain names.                               |
| `8080` | M3TAL API Daemon (Go)                  | Accessible only on the host machine (`http://localhost:8080` or `http://host.docker.internal:8080`). Exposed externally via Traefik at `api.DOMAIN`. |
| `8081` | Traefik Dashboard (Internal)           | Accessible only on the host machine (`http://127.0.0.1:8081`). Used for monitoring Traefik.                                         |
| `8082` | M3TAL Dashboard (Python/Flask)         | Accessible directly on the host (`http://YOUR_IP:8082`) if `DASHBOARD_EXPOSE_MODE=local`, or internally via Traefik at `dash.DOMAIN` if `DASHBOARD_EXPOSE_MODE=traefik`. |

## Firewall Configuration

If you have a firewall enabled (e.g., `ufw`), you may need to allow traffic on relevant ports. If Traefik is exposed publicly on port 80, execute:

```bash
sudo ufw allow 80/tcp
```

Adjust port numbers as necessary if you've configured Traefik to listen on different ports.

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view logs using `systemctl` and `journalctl`:

*   **Check the status of the API daemon**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs for the API daemon**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the API daemon**:
    ```bash
    sudo systemctl restart m3tal-api
    ```