```markdown
# GETTING STARTED WITH M3TAL

This guide provides a step-by-step process for first-time installation and initial setup of the M3TAL Ecosystem.

## 1. Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.
Verify their installation by running:

```bash
docker --version && docker compose version
```

You should see output similar to:
```
Docker version 24.0.7, build afdd53b
Docker Compose version v2.24.5
```

## 2. Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the `m3tal` CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, configure M3TAL's core settings using the interactive wizard. This will create or update the primary configuration file (`/etc/m3tal/.env`).

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Here's an explanation of key prompts:

*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): The dashboard is directly accessible on a specific port (default 8082) on the host's IP. Best for local network access and initial setup without a domain.
    *   `traefik`: The dashboard is exposed via Traefik, accessible through a domain name (e.g., `dash.YOUR_DOMAIN`). Requires Traefik to be running.
*   **`DOMAIN`**: The base domain name for your services if `DASHBOARD_EXPOSE_MODE` is `traefik`, or if you plan to use other services with Traefik. Example: `example.com`.
*   **`PUID`** (User ID) and **`PGID`** (Group ID): These define the user and group permissions for containers created by M3TAL. It's recommended to use the PUID/PGID of your primary user (often `1000:1000`) for correct volume permissions.
*   **`TZ`**: Your local timezone (e.g., `America/New_York`). Used by containers for correct time display.
*   **`DASHBOARD_SECRET`**: A secret key used by the dashboard for secure session management. Generate a strong, random string.
*   **`API_TOKEN`**: A secret token for authenticating with the M3TAL API. Generate a strong, random string.
*   **`ADMIN_PASSWORD`**: The default password for the M3TAL Dashboard's administrative user. This should be changed immediately after initial login.

## 4. Start the Routing Stack

The `m3tal up` command starts all Docker Compose stacks defined in `*-compose.yml` files within the `/docker/` directory. This primarily includes the Traefik reverse proxy (from `routing-compose.yml`), which handles network routing for your services.

```bash
m3tal up
```

## 5. Start the M3TAL Dashboard

The `m3tal dash up` command specifically pulls the latest M3TAL Dashboard Docker image and starts its container based on your `DASHBOARD_EXPOSE_MODE` setting.

```bash
m3tal dash up
```

## 6. Open the Dashboard in Your Browser

Access the M3TAL Dashboard using your web browser:

*   **If `DASHBOARD_EXPOSE_MODE` is `local`:**
    Open `http://YOUR_IP:8082` (replace `YOUR_IP` with the IP address of your server, or use `localhost` if accessing from the same machine).
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Open `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured). This requires Traefik to be running (`m3tal up`).

## 7. Log In

The default login credentials for the dashboard are:

*   **Username:** `admin`
*   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard (default is `admin_pass`).

It is highly recommended to change the `ADMIN_PASSWORD` immediately after your first login for security reasons.
You can change the dashboard password using the CLI:

```bash
sudo m3tal dashpass
```

---

## Filesystem Contract

M3TAL relies on a specific filesystem layout for its operations:

| Path                        | Purpose                                                            | Managed By                                    |
| :-------------------------- | :----------------------------------------------------------------- | :-------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file, stores environment variables.          | `m3tal config wizard`, `m3tal config set`     |
| `/var/lib/m3tal/state.db`   | SQLite state database for the API daemon.                          | M3TAL API daemon (auto-created)               |
| `/opt/m3tal/stack/`         | Canonical directory for M3TAL's core Docker Compose files.         | M3TAL installation                            |
| `/docker`                   | Symlink pointing to `/opt/m3tal/stack/`. User-facing path for all Docker Compose operations and custom stacks. | M3TAL installation                            |
| `/docker/users.json`        | Dashboard credential store.                                        | `m3tal dashpass`                              |

## Docker / Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** for container management. These are hard dependencies.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files found in `/docker/`. This includes `routing-compose.yml` for Traefik and any custom user stacks.
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container. It:
    1.  Downloads the latest `m3tal-compose.yml` and its override files (`m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
    2.  Reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Starts the dashboard using the base `m3tal-compose.yml` combined with the appropriate override file (either `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).
*   To add new services, place your Docker Compose files (e.g., `my-service-compose.yml`) directly into the `/docker/` directory.

## M3TAL Dashboard Access Modes

The dashboard's accessibility is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Mode 1: Local (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   Uses the `m3tal-compose.local.yml` override file.
*   This mode adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access via:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik required.
*   **Use Cases:** Ideal for LAN-only setups, first-time users, and local development or testing.

### Mode 2: Traefik

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   Uses the `m3tal-compose.traefik.yml` override file.
*   This mode adds Traefik labels to the dashboard container, allowing Traefik to route `dash.YOUR_DOMAIN` to the dashboard on port 8082.
*   **Access via:** `http://dash.YOUR_DOMAIN` (Traefik must be running via `m3tal up`).
*   **Requirements:** Traefik must be running and configured with a domain.
*   **Use Cases:** Best for domain-based setups and integrating the dashboard behind a reverse proxy for consolidated access.

## Port Table

The following ports are typically used by M3TAL components:

| Port | Service               | Access Level                                |
| :--- | :-------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point | Public (if Traefik is exposed)              |
| 8080 | M3TAL API daemon (Go) | Host-local only                             |
| 8081 | Traefik Dashboard     | Host-local only (for Traefik's own dashboard) |
| 8082 | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If you are exposing Traefik (and therefore port 80) to the internet or your local network, ensure your firewall allows traffic on that port. For `ufw` (Uncomplicated Firewall), you can open port 80 with:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service. You can manage and monitor it using standard systemctl commands:

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
```