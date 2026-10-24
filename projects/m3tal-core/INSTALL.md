# M3TAL GET STARTED

This guide provides step-by-step instructions for setting up M3TAL for the first time.

## Step 1: Prerequisites

Before proceeding, ensure you have the following installed:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installations by running:

```bash
docker --version
docker compose version
```

If these commands do not return version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

Install M3TAL using the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through essential setup. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of what each prompt means:

*   **`DASHBOARD_EXPOSE_MODE`**: This determines how the M3TAL dashboard is accessed.
    *   `local`: The dashboard will be accessible directly via its port (default 8082). This is ideal for local access or simple LAN setups.
    *   `traefik`: The dashboard will be exposed through Traefik, allowing access via a domain name (e.g., `dash.yourdomain.com`). This requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name for your M3TAL instance. If you are using `traefik` mode for the dashboard, this will be used to construct its access URL. Defaults to `localhost`.
*   **`API_PORT`**: The port on which the M3TAL API daemon listens.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard container runs internally.
*   **`STATE_DIR`**: The directory where M3TAL stores its persistent state, including the `state.db` file.
*   **`DASHBOARD_SECRET`**: A secret key for securing the dashboard. It's highly recommended to change this from the default.
*   **`ADMIN_PASSWORD`**: The password for the default administrator account in the M3TAL dashboard.
*   **`PUID` / `PGID`**: The User ID and Group ID for running Docker containers. Ensure these match the user who owns your M3TAL data directories.
*   **`TZ`**: Your timezone. This is important for accurate logging and scheduling.
*   **`TRAEFIK_WEB_PORT`**: The host port Traefik will bind to for HTTP traffic.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The host port Traefik will bind to for HTTPS traffic (if configured).

## Step 4: Start the Routing Stack (Traefik)

M3TAL uses Docker Compose to manage its services. The `m3tal up` command orchestrates all compose files located in the `/docker/` directory. To start the core routing stack (which includes Traefik), run:

```bash
m3tal up
```

This command will pull necessary Docker images and start the containers defined in `/docker/*.yml`, including Traefik, which acts as your primary reverse proxy.

## Step 5: Start the Dashboard

To start the M3TAL dashboard container, execute:

```bash
m3tal dash up
```

This command will:
1.  Fetch the latest `m3tal-compose.yml` and any relevant override files (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
2.  Pull the `m3tal-dashboard` Docker image if it's not already present.
3.  Start the `m3tal-dashboard` container with the appropriate configuration.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's URL. The address depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), access the dashboard at:
    `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or `http://localhost:8082` if accessing from the server itself).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, and Traefik is running, access the dashboard via the domain you configured:
    `http://dash.DOMAIN` (replace `DOMAIN` with your configured domain).

## Step 7: Log In

Use the following default credentials to log into the M3TAL dashboard:

*   **Username:** `admin`
*   **Password:** `admin_pass` (or the password you set during the wizard)

It is **strongly recommended** to change the default password immediately after logging in for security. You can change the dashboard password using the following command:

```bash
sudo m3tal dashpass admin <new_password>
```

Replace `<new_password>` with your desired new password.

## Filesystem Contract

M3TAL utilizes specific file locations for configuration and state management:

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the API daemon.               |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's core Docker compose files and configs. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations and custom compose files. |
| `/docker/users.json`     | Stores dashboard credential hashes. Managed by `m3tal dashpass`.       |

## Port Map

The following ports are used by M3TAL and its associated services:

| Port | Service          | Access                                       |
| :--- | :--------------- | :------------------------------------------- |
| 80   | Traefik HTTP     | Public (if Traefik is exposed)               |
| 8080 | M3TAL API Daemon | Host-local (internal to the host)            |
| 8081 | Traefik Dashboard| Host-local only (unless exposed explicitly) |
| 8082 | M3TAL Dashboard  | Direct port access (`local` mode) or via Traefik (`traefik` mode) |

## Firewall Note

If you are exposing Traefik (and thus potentially other services) to the public internet, ensure that port 80 (HTTP) is allowed through your firewall. If using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   Check the status of the API service:
    ```bash
    systemctl status m3tal-api
    ```
*   View real-time logs for the API service:
    ```bash
    journalctl -u m3tal-api -f
    ```