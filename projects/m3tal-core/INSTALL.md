# M3TAL Getting Started Guide

This guide provides operational steps for the initial setup of a M3TAL ecosystem.

---

### Step 1: Prerequisites

M3TAL requires **Docker Engine** and **Docker Compose V2** to be installed on your system.
Verify their installation by running:

```bash
docker --version && docker compose version
```

Example output:
```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.23.3-desktop.1
```

If either is not installed or the version is older, please install or update them according to the official Docker documentation.

---

### Step 2: Install M3TAL via APT

Execute the following commands in your terminal to install the M3TAL CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

### Step 3: Run the Configuration Wizard

After installation, run the M3TAL configuration wizard to set up essential environment variables. This creates the `/etc/m3tal/.env` file.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Below is an explanation for key prompts:

*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard will be accessible.
    *   `local` (default): The dashboard will be directly exposed on port `8082` of the host. Access via `http://YOUR_IP:8082`. This mode does not require Traefik for dashboard access and is suitable for local network use or initial setup.
    *   `traefik`: The dashboard will be accessible via a subdomain using Traefik. Access via `http://dash.DOMAIN` (where `DOMAIN` is set below). This requires Traefik to be running and configured.
*   **`DOMAIN`**: The base domain for your services (e.g., `example.com`). This is primarily used by Traefik for routing services via subdomains (e.g., `dash.example.com`, `api.example.com`). If you selected `DASHBOARD_EXPOSE_MODE=local`, you can leave this as `localhost` for now.
*   **`PUID` / `PGID`**: The User ID and Group ID that containers will run as. Use `id -u` and `id -g` to find your current user's IDs. This is important for ensuring proper file permissions for mounted volumes.
*   **`TZ`**: Your system's timezone (e.g., `America/Denver`).
*   **`DASHBOARD_SECRET`**: A strong, randomly generated string used for securing dashboard sessions. Generate a complex string and keep it secure.
*   **`ADMIN_PASSWORD`**: The default password for the M3TAL Dashboard's `admin` user. Choose a strong password.
*   **`API_TOKEN`**: A secure token for authenticating requests to the M3TAL API daemon. Generate a strong, unique value.

---

### Step 4: Start the Routing Stack

The `m3tal up` command starts all Docker Compose files found in the `/docker/` directory. This includes the `routing-compose.yml` which deploys Traefik, the reverse proxy gateway.

```bash
m3tal up
```

This command uses `docker compose` to bring up the services defined in your stack files, such as Traefik, which will bind to port 80 on your host.

---

### Step 5: Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It pulls the necessary Docker image, checks your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.

```bash
m3tal dash up
```

This command handles downloading `m3tal-compose.yml` and its overrides, then starts the `m3tal-dashboard` container.

---

### Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard using the URL corresponding to your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    Go to `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with your server's actual IP address or `localhost` if accessing from the server itself).
*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    Go to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in Step 3). Traefik must be running for this mode to work.

---

### Step 7: Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the configuration wizard (Step 3).

To change the dashboard password for the `admin` user at any time, use the command:

```bash
sudo m3tal dashpass
```

This command will prompt you to enter and confirm a new password.

---

## M3TAL Filesystem Contract

M3TAL establishes a clear contract for its filesystem layout:

| Path                     | Purpose                                                         |
| :----------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the API daemon.          |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal Docker Compose files.  |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations. Place your custom `*-compose.yml` files here. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.        |

---

## M3TAL Port Map

The following ports are used by M3TAL components:

| Port | Service               | Access                                                 |
| :--- | :-------------------- | :----------------------------------------------------- |
| 80   | Traefik HTTP entry point | Public (if Traefik is exposed)                         |
| 8080 | M3TAL API daemon (Go) | Host-local only (or via Traefik at `api.DOMAIN`)         |
| 8081 | Traefik dashboard     | Host-local only (accessed via `http://localhost:8081`) |
| 8082 | M3TAL Dashboard       | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

---

## Firewall Note

If your server has a firewall (e.g., `ufw`) enabled, you may need to allow traffic on the ports you wish to expose.

*   To expose Traefik (for `dash.DOMAIN` or other proxied services):
    ```bash
    sudo ufw allow 80
    ```
*   To expose the M3TAL Dashboard directly (if `DASHBOARD_EXPOSE_MODE=local`):
    ```bash
    sudo ufw allow 8082
    ```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using standard `systemctl` commands:

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View API service logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```