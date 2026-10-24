# docs/GET_STARTED.md

As DocSmith, the M3TAL Ecosystem Documentation Architect, this guide provides a complete, step-by-step setup for first-time users. Follow these operational steps to get your M3TAL system running.

---

## 1. Prerequisites

M3TAL relies on Docker Engine and Docker Compose V2 for container management. These must be installed on your system before proceeding.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Expected output will show the installed versions of Docker Engine and Docker Compose.

## 2. Install M3TAL via APT

M3TAL is distributed as a single Go binary (`m3tal`) via an APT repository. Execute the following commands to install:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the configuration wizard to set up essential environment variables. This creates or updates `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Here's an explanation of key prompts:

*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is primarily used by Traefik for routing if you expose services by domain name. If you only plan to access services by IP, you can leave this as `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Controls how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard will be directly accessible via `http://YOUR_IP:8082`. This is recommended for first-time users or LAN-only setups.
    *   `traefik`: The dashboard will be accessible via `http://dash.YOUR_DOMAIN` using Traefik as a reverse proxy. This requires Traefik to be running.
*   **`PUID` / `PGID`**: These are the User ID (UID) and Group ID (GID) that M3TAL's containers will use for file permissions. It's recommended to set these to your current user's IDs to avoid permission issues. You can find your PUID/PGID with `id -u` and `id -g`.
*   **`TZ`**: The timezone for your containers (e.g., `America/Denver`).
*   **`BASE_STORAGE_PATH`**: The base directory where M3TAL will store all persistent data (e.g., `/mnt`). This is where `/mnt` in container volumes will map to.
*   **`DASHBOARD_SECRET` / `API_TOKEN` / `ADMIN_PASSWORD`**: These are security credentials. While defaults are provided, it is critical to change them immediately after initial setup for production environments.

## 4. Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose stacks found in the `/docker/` directory. By default, this includes `routing-compose.yml`, which deploys the Traefik reverse proxy.

```bash
m3tal up
```

This command initiates the Docker containers defined in all `*-compose.yml` files in `/docker/`. Traefik will start on port 80 and prepare to route traffic.

## 5. Start the Dashboard

The M3TAL Dashboard is a separate container managed specifically by `m3tal dash up`. This command will pull the necessary Docker image and start the dashboard container, applying the correct configuration based on your `DASHBOARD_EXPOSE_MODE` setting from Step 3.

```bash
m3tal dash up
```

This command downloads the latest dashboard compose files from GitHub, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard container with either `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml` override.

## 6. Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your server).
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in `/etc/m3tal/.env`). This requires Traefik to be running (Step 4).

## 7. Log In to the Dashboard

The default login credentials for the dashboard are:

*   **Username:** `admin`
*   **Password:** `admin_pass` (as set in `ADMIN_PASSWORD` in `/etc/m3tal/.env`)

**Change the default password immediately:**

```bash
sudo m3tal dashpass
```

This command will prompt you to set a new password for the `admin` user, updating the `/docker/users.json` file.

---

## Filesystem Contract

M3TAL adheres to a specific filesystem layout for its operation and configuration.

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file, managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`| SQLite state database, auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`      | Canonical stack directory containing core compose files and Traefik config. |
| `/docker`                | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. Place your custom `*-compose.yml` files here. |
| `/docker/users.json`     | Dashboard credential store, managed by `m3tal dashpass`.               |

---

## Port Table

M3TAL components expose services on specific ports. Ensure these ports are available and configured correctly in your firewall if public access is desired.

| Port | Service                                  | Access                                     |
| :--- | :--------------------------------------- | :----------------------------------------- |
| 80   | Traefik HTTP entry point                 | Public (if Traefik is exposed)             |
| 8080 | M3TAL API daemon (Go)                    | Host-local (accessible via `localhost:8080` or `host.docker.internal:8080` by containers) |
| 8081 | Traefik dashboard (admin interface)      | Host-local only (`127.0.0.1:8081`)         |
| 8082 | M3TAL Dashboard (Python/Flask container) | Direct port (local mode) or via Traefik (traefik mode) |

---

## Firewall Configuration

If you have a firewall (e.g., `ufw`) enabled on your system and wish to access Traefik or the dashboard from outside the local machine, you must allow incoming connections on the relevant ports.

To allow public HTTP access via Traefik (port 80):

```bash
sudo ufw allow 80/tcp
```

If you are using the `local` dashboard mode and need remote access to port 8082:

```bash
sudo ufw allow 8082/tcp
```

---

## Service Management

The M3TAL API daemon runs as a systemd service (`m3tal-api.service`). You can manage this service using standard `systemctl` commands.

*   **Check the status of the API daemon:**

    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the API daemon:**

    ```bash
    journalctl -u m3tal-api -f
    ```