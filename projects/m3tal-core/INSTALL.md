# M3TAL Getting Started Guide

This guide provides a step-by-step process for first-time users to set up and start using the M3TAL Ecosystem.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation by running:

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

### Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up essential environment variables. This wizard populates the primary M3TAL configuration file located at `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Here's an explanation for the critical prompts:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   `local` (default): The M3TAL Dashboard will be directly accessible via a host port binding (e.g., `http://YOUR_IP:8082`). This mode does not require Traefik for dashboard access and is suitable for LAN-only setups or initial testing.
    *   `traefik`: The M3TAL Dashboard will be exposed via the Traefik reverse proxy using a domain name (e.g., `http://dash.YOUR_DOMAIN`). This mode requires Traefik to be running and configured with a `DOMAIN`.
*   **`DOMAIN`**:
    *   Defaults to `localhost`. If you choose `DASHBOARD_EXPOSE_MODE=traefik` or plan to use Traefik for other services, set this to your desired public domain (e.g., `example.com`). This will be used for Traefik routing rules (e.g., `dash.example.com`, `api.example.com`).
*   **`DASHBOARD_PORT`**:
    *   Defaults to `8082`. This is the internal port the dashboard container listens on. If `DASHBOARD_EXPOSE_MODE=local`, this port will be bound directly to your host's network.
*   **`PUID` / `PGID`**:
    *   Defaults to `1000`. These are the User ID (PUID) and Group ID (PGID) that containers will use to manage file permissions. It is recommended to use the PUID and PGID of the user who owns the M3TAL data directories to prevent permission issues.
*   **`TZ`**:
    *   Defaults to `America/Denver`. Set this to your local timezone (e.g., `Europe/London`, `America/New_York`).
*   **`API_TOKEN`**:
    *   A secret token used for secure communication with the M3TAL API daemon. A strong, randomly generated value is recommended.
*   **`DASHBOARD_SECRET`**:
    *   A secret key used to secure dashboard user sessions. A strong, randomly generated value is recommended.
*   **`ADMIN_PASSWORD`**:
    *   The password for the default `admin` user on the M3TAL Dashboard. Change this immediately.

### Step 4: Start the Routing Stack (Traefik)

The M3TAL CLI manages Docker Compose stacks. The `m3tal up` command starts all `*-compose.yml` files located in the `/docker/` directory. This includes the core `routing-compose.yml` which deploys Traefik, the reverse proxy.

```bash
m3tal up
```

This command will start the Traefik gateway container, exposing port 80 (and potentially 443 if configured for HTTPS) on your host.

### Step 5: Start the Dashboard

The `m3tal dash up` command specifically manages the M3TAL Dashboard container. It pulls the latest dashboard image, determines the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and then starts the dashboard container with the appropriate configuration.

```bash
m3tal dash up
```

### Step 6: Open Browser

Access the M3TAL Dashboard via your web browser:

*   **If `DASHBOARD_EXPOSE_MODE` is `local`**:
    Open `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your M3TAL host, or `localhost` if accessing locally).
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`**:
    Open `http://dash.DOMAIN` (replace `DOMAIN` with the value you configured in Step 3, e.g., `http://dash.example.com`). Traefik must be running (`m3tal up`) for this mode to work.

### Step 7: Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username**: `admin`
*   **Password**: `admin_pass` (or the password you set during the configuration wizard)

**Immediately change your password** using the `m3tal dashpass` command:

```bash
sudo m3tal dashpass
```

---

## Filesystem Contract

M3TAL relies on a specific filesystem layout for its operation:

| Path                        | Purpose                                                                                |
| :-------------------------- | :------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file, managed by `m3tal config wizard` and `m3tal config set`.   |
| `/var/lib/m3tal/state.db`   | SQLite state database, used by the M3TAL API daemon. Auto-created.                     |
| `/opt/m3tal/stack/`         | Canonical directory containing all Docker Compose files and Traefik static configuration. |
| `/docker`                   | **Symlink** to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store, managed by `m3tal dashpass`.                               |

## Port Table

The following ports are used by M3TAL components:

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public (if `TRAEFIK_WEB_PORT` is 80)        |
| 8080 | M3TAL API daemon (Go)       | Host-local only                             |
| 8081 | Traefik dashboard           | Host-local only (`127.0.0.1:8081:8080`)     |
| 8082 | M3TAL Dashboard container   | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If your M3TAL host is behind a firewall, you may need to explicitly allow traffic on certain ports.

*   To expose Traefik on port 80:
    ```bash
    sudo ufw allow 80
    ```
*   If `DASHBOARD_EXPOSE_MODE` is `local` and you need to access the dashboard from a remote machine:
    ```bash
    sudo ufw allow 8082
    ```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state using `systemctl` and view its logs with `journalctl`:

*   Check the status of the M3TAL API daemon:
    ```bash
    systemctl status m3tal-api
    ```
*   View live logs for the M3TAL API daemon:
    ```bash
    journalctl -u m3tal-api -f
    ```