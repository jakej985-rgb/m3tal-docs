```markdown
# GET_STARTED.md

This guide details the setup process for M3TAL, providing operational steps for first-time users.

---

## 1. Prerequisites

M3TAL relies on Docker Engine and Docker Compose V2 for container management. These must be installed on your system before proceeding.

Verify your Docker installation:

```bash
docker --version && docker compose version
```

Expected output will show versions for Docker Engine and Docker Compose V2.

## 2. Install M3TAL via APT

M3TAL is distributed as a single Go binary via an APT repository. Execute the following commands to add the repository and install M3TAL:

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

The wizard will prompt for the following key parameters:

*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is used for Traefik routing if `DASHBOARD_EXPOSE_MODE` is set to `traefik`. Default: `localhost`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly exposed on a host port (`8082` by default). This is suitable for LAN-only access or testing without Traefik.
    *   `traefik`: The dashboard is routed via Traefik using a subdomain (e.g., `dash.DOMAIN`). Requires Traefik to be running.
*   **`PUID` / `PGID`**: User ID and Group ID for containers. These ensure containers have correct file permissions when accessing host volumes. Use `id -u` and `id -g` to find your current user's IDs. Default: `1000`.
*   **`TZ`**: Your timezone (e.g., `America/New_York`). This ensures correct timekeeping within containers. Default: `America/Denver`.
*   **`ADMIN_PASSWORD`**: The default password for the M3TAL Dashboard. You will be prompted to change this later. Default: `admin_pass`.
*   Other variables like storage paths (`BASE_STORAGE_PATH`, `CONFIG_PATH`, etc.) can be set according to your system's storage layout.

## 4. Start the Routing Stack (Traefik)

The M3TAL routing stack includes Traefik, the reverse proxy. This step deploys all Docker Compose files found in `/docker/`.

```bash
m3tal up
```

This command starts the services defined in `/docker/routing-compose.yml` (which contains Traefik and optionally Cloudflared), along with any other `*-compose.yml` files present in `/docker/`. Traefik binds to port 80 (HTTP) on the host.

## 5. Start the M3TAL Dashboard

The M3TAL Dashboard provides a web interface for managing your ecosystem.

```bash
m3tal dash up
```

This command performs the following actions:
1.  It downloads the latest dashboard Docker Compose configuration files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
2.  It reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  It starts the `m3tal-dashboard` container, applying the appropriate override file (`m3tal-compose.local.yml` for `local` mode or `m3tal-compose.traefik.yml` for `traefik` mode). This pulls the dashboard image if not already present.

## 6. Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with your machine's IP address, or `localhost` if accessing locally).

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain configured in Step 3, e.g., `http://dash.example.com`). Traefik must be running for this mode to work.

## 7. Log In to the Dashboard

The initial login credentials are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard (default: `admin_pass`).

It is strongly recommended to change the admin password immediately after logging in. You can do this via the dashboard settings or by using the CLI:

```bash
sudo m3tal dashpass
```
Follow the prompts to set a new password for the `admin` user.

---

## Filesystem Contract

M3TAL adheres to a specific filesystem layout for configuration, data, and stack management.

| Path                     | Purpose                                                                |
| :----------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for M3TAL. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the API daemon.      |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal Docker Compose files.          |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. User-facing path for managing Docker Compose stacks. Place your `*-compose.yml` files here. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.               |

---

## M3TAL Architecture Overview

M3TAL consists of several integrated components:

*   **CLI binary** (`/usr/bin/m3tal`): The command-line interface, providing a unified entry point for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on port `8080`. It manages Docker, interacts with the state database, and provides API routes for the dashboard and other clients.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container providing the web interface. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services on port 80 and routes traffic based on domain names and Docker labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container for creating secure, zero-config tunnels through Cloudflare, allowing internet access to local services without exposing ports directly.

---

## Docker / Compose Runtime Explained

M3TAL orchestrates services using **Docker Engine** and **Docker Compose V2**.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files located in `/docker/`. This allows you to place your custom Docker Compose stacks in `/docker/` and manage them alongside M3TAL's core services.
*   The `m3tal dash up` command specifically manages the M3TAL Dashboard container. As detailed in Step 5, it dynamically selects and applies the correct Docker Compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting, ensuring the dashboard starts with the appropriate network configuration.

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state and view its logs using standard `systemctl` and `journalctl` commands:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **Restart the API daemon:**
    ```bash
    sudo systemctl restart m3tal-api
    ```
*   **View real-time logs for the API daemon:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Port Map

The following ports are used by M3TAL components on the host system:

| Port | Service               | Access Level                                   |
| :--- | :-------------------- | :--------------------------------------------- |
| `80` | Traefik HTTP entry point | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)    |
| `8080` | M3TAL API daemon (Go) | Host-local (accessible by containers via `host.docker.internal:8080`) |
| `8081` | Traefik dashboard     | Host-local only (`127.0.0.1:8081`)             |
| `8082` | M3TAL Dashboard       | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Configuration

If you expose Traefik to the internet or your local network (i.e., you are using `DASHBOARD_EXPOSE_MODE=traefik` or plan to host other services via Traefik), you must allow traffic on port 80 through your firewall.

Example for `ufw` (Uncomplicated Firewall):

```bash
sudo ufw allow 80/tcp
sudo ufw enable
```

---

## Day 2 Operations: Adding a New Stack

To add your own Docker Compose stacks to the M3TAL ecosystem:

1.  **Place your compose file:** Create or copy your `*-compose.yml` file into the `/docker/` directory (e.g., `/docker/my-app-compose.yml`).
2.  **Set environment variables:** Ensure any necessary environment variables for your stack are defined in `/etc/m3tal/.env`. You can use `sudo m3tal config wizard` to add them interactively, or `sudo m3tal config set KEY value` for specific variables.
3.  **Start all stacks:** Run `m3tal up` to deploy your new stack along with the existing M3TAL services.
```