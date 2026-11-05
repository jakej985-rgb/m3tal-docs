```markdown
# docs/GET_STARTED.md

This guide provides a complete, step-by-step setup for M3TAL.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.

Verify your Docker and Docker Compose V2 installation:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

Install the M3TAL CLI binary and API daemon using the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the configuration wizard

Initialize your M3TAL environment and set essential variables:

```bash
sudo m3tal config wizard
```

The wizard will guide you through setting up core environment variables in `/etc/m3tal/.env`. Key prompts include:

*   **`DOMAIN`**: Your primary domain name (e.g., `example.com`). This is used for Traefik routing.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (default): Dashboard is exposed directly on a port (e.g., `8082`). No Traefik configuration needed for dashboard access.
    *   `traefik`: Dashboard is routed via Traefik using `dash.YOUR_DOMAIN`. Requires Traefik to be running.
*   **Other variables**: The wizard may prompt for other service-specific variables or API tokens as needed.

### Step 4: Start the routing stack (Traefik)

This command starts all Docker Compose files (ending in `-compose.yml`) found in the `/docker/` directory, including the Traefik reverse proxy defined in `routing-compose.yml`.

```bash
m3tal up
```

### Step 5: Start the dashboard

This command specifically manages the M3TAL Dashboard. It pulls the latest `m3tal-dashboard` image, and starts its container based on the `DASHBOARD_EXPOSE_MODE` setting from Step 3.

```bash
m3tal dash up
```

### Step 6: Open browser

Access the M3TAL Dashboard in your web browser:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    `http://YOUR_SERVER_IP:8082` (e.g., `http://192.168.1.100:8082` or `http://localhost:8082`)
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    `http://dash.YOUR_DOMAIN` (e.g., `http://dash.example.com`). Traefik must be running for this to work.

### Step 7: Log in

The M3TAL Dashboard does not have default login credentials. You must set them initially.

To set or change the dashboard password:

```bash
sudo m3tal dashpass
```

Follow the prompts to create your dashboard username and password. After setting them, use these credentials to log into the dashboard.

---

### Filesystem Contract

The following paths are essential to the M3TAL system:

| Path                    | Purpose                                                                                             |
| :---------------------- | :-------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`       | Primary configuration file. Contains environment variables. Managed by `m3tal config wizard`.       |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created and managed by the M3TAL API daemon.                            |
| `/opt/m3tal/stack/`     | Canonical stack directory. Contains core compose files and Traefik configuration.                   |
| `/docker`               | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations and custom compose files. |
| `/docker/users.json`    | Dashboard credential store. Managed by `m3tal dashpass`.                                            |

### Port Table

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public (if `DASHBOARD_EXPOSE_MODE=traefik` or other services exposed) |
| 8080 | M3TAL API daemon (Go)       | Host-local only (accessed by other services like dashboard) |
| 8081 | Traefik Dashboard           | Host-local only (access to Traefik's own dashboard) |
| 8082 | M3TAL Dashboard container   | Direct port (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (if `DASHBOARD_EXPOSE_MODE=traefik`) |

### Firewall Note

If you expose Traefik on port 80 or the M3TAL Dashboard on port 8082 to the internet, ensure your firewall allows incoming connections on these ports.

Example for Traefik (port 80):

```bash
sudo ufw allow 80/tcp
```

Example for M3TAL Dashboard (port 8082) in `local` mode:

```bash
sudo ufw allow 8082/tcp
```

### Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   Check the status of the M3TAL API daemon:
    ```bash
    systemctl status m3tal-api
    ```
*   View live logs from the M3TAL API daemon:
    ```bash
    journalctl -u m3tal-api -f
    ```

---

### Dashboard Access Modes (Detailed)

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

1.  **`local` mode (default)**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
    *   **Mechanism**: The dashboard container has a direct port binding, typically `8082:8082`.
    *   **Access**: Open your browser to `http://YOUR_SERVER_IP:8082` or `http://localhost:8082`.
    *   **Usage**: Ideal for LAN-only setups, first-time users, or local development where a full domain setup is not required. No Traefik configuration is involved for the dashboard itself.

2.  **`traefik` mode**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
    *   **Mechanism**: The dashboard container is configured with Traefik labels, allowing Traefik to route `dash.YOUR_DOMAIN` to the dashboard's internal port `8082`.
    *   **Access**: Open your browser to `http://dash.YOUR_DOMAIN`.
    *   **Usage**: Best for domain-based setups where you manage multiple services behind a single reverse proxy (Traefik). Requires Traefik to be running via `m3tal up`.

### Docker / Compose Runtime

M3TAL leverages Docker Engine and Docker Compose V2 for managing its services and user-defined stacks.

*   The `m3tal up` command acts as a wrapper around `docker compose up -d`, applying it to all `*-compose.yml` files located in the `/docker/` directory. This includes core services like Traefik (`routing-compose.yml`) and any user-deployed stacks.
*   The `m3tal dash up` command is specifically for the M3TAL Dashboard. It performs the following actions:
    1.  Downloads the necessary Compose files for the dashboard (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`).
    2.  Reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the appropriate Compose override file (`m3tal-compose.local.yml` for `local` mode, or `m3tal-compose.traefik.yml` for `traefik` mode).
*   To add your own Docker Compose stacks, simply place your `*-compose.yml` files directly into the `/docker/` directory.

### Traefik Routing Architecture

Traefik acts as the central reverse proxy for M3TAL. It is deployed as a container via `routing-compose.yml` and performs the following functions:

*   **Entry Point**: Binds to port 80 on the host system to serve HTTP traffic.
*   **Service Discovery**: Automatically discovers services configured with Docker labels (e.g., the M3TAL Dashboard in `traefik` mode).
*   **File Provider**: Loads additional dynamic configuration from the `/docker/dynamic/` directory, allowing for hot-reloading of routing rules.
*   **API Routing**: Routes `api.YOUR_DOMAIN` to the M3TAL API daemon, which runs on `http://host.docker.internal:8080`.
*   **Dashboard Routing**: Routes `dash.YOUR_DOMAIN` to the M3TAL Dashboard container when `DASHBOARD_EXPOSE_MODE` is set to `traefik`.