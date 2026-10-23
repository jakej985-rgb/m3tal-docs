# M3TAL Getting Started Guide

This guide provides a step-by-step process for installing and configuring M3TAL for first-time users.

---

### Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed on your system.

Verify their installation by running:

```bash
docker --version && docker compose version
```

### Step 2: Install M3TAL via APT

Execute the following commands to add the M3TAL APT repository and install the `m3tal` CLI:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

After installation, run the configuration wizard to set up essential environment variables:

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several values. Here is an explanation of the key prompts:

*   **DOMAIN**: The primary domain for your M3TAL services (e.g., `example.com`). This is used by Traefik for routing. If you don't have a domain or are using M3TAL locally, `localhost` is a suitable default.
*   **DASHBOARD_EXPOSE_MODE**:
    *   `local` (default): The dashboard will be directly exposed on port `8082` of your host machine. This is ideal for local/LAN-only access and setups without Traefik.
    *   `traefik`: The dashboard will be exposed via Traefik at `dash.YOUR_DOMAIN`. This requires Traefik to be running.
*   **PUID / PGID**: The User ID and Group ID that Docker containers will run as. Typically, these should match your non-root user's ID (e.g., `1000:1000`).
*   **TZ**: Your system's timezone (e.g., `America/New_York`).
*   **DASHBOARD_SECRET**: A secret key used to secure the dashboard session. A random value will be generated if left blank.
*   **API_TOKEN**: An authentication token for M3TAL API access. A random value will be generated if left blank.
*   **ADMIN_PASSWORD**: The default password for the M3TAL Dashboard's `admin` user.

These values are stored in `/etc/m3tal/.env`. You can modify them later using `sudo m3tal config set KEY value`.

### Step 4: Start the Routing Stack

Start the core M3TAL services, including Traefik, the API daemon, and any other default compose files:

```bash
m3tal up
```

This command initiates Docker Compose for all `*-compose.yml` files located in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`). This includes `routing-compose.yml` for Traefik and `api-compose.yml` for the M3TAL API daemon.

### Step 5: Start the Dashboard

Start the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command performs the following actions:
1.  Downloads the latest M3TAL Dashboard Docker Compose files (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) from GitHub.
2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
3.  Starts the `m3tal-dashboard` container using the appropriate Compose override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting.

### Step 6: Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard:

*   **If `DASHBOARD_EXPOSE_MODE` is `local`**:
    Access via `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address) or `http://localhost:8082` if accessing from the server itself.
*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`**:
    Access via `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in Step 3). Traefik must be running for this mode to work.

### Step 7: Log In to the Dashboard

The default login credentials for the M3TAL Dashboard are:

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the configuration wizard (default is `admin_pass` if you skipped setting it).

**Change Dashboard Password:**
It is highly recommended to change the `admin` password immediately. You can do this using the CLI:

```bash
sudo m3tal dashpass
```

This command will prompt you to set a new password for the `admin` user.

---

### Filesystem Contract

M3TAL relies on specific file paths for its operation:

| Path                     | Purpose                                                         |
| :----------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file, managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's runtime state. Auto-created.    |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's core Docker Compose files.      |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations. Place your custom `*-compose.yml` files here. |
| `/docker/users.json`     | Stores M3TAL Dashboard user credentials. Managed by `m3tal dashpass`. |

---

### M3TAL Port Map

| Port | Service               | Access                                  |
| :--- | :-------------------- | :-------------------------------------- |
| 80   | Traefik HTTP Entry Point| Public (if `TRAEFIK_WEB_PORT` is 80)    |
| 8080 | M3TAL API Daemon (Go) | Host-local only                         |
| 8081 | Traefik Dashboard     | Host-local only (for Traefik's own dashboard) |
| 8082 | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) |

### Firewall Configuration

If you expose Traefik on port 80 (the default for `TRAEFIK_WEB_PORT`), ensure your firewall allows incoming connections on this port. For `ufw` (Uncomplicated Firewall), use:

```bash
sudo ufw allow 80/tcp
```

### M3TAL Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

### Dashboard Access Modes (Detailed)

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

1.  **Local Mode (`DASHBOARD_EXPOSE_MODE=local`)**:
    *   This is the default setting.
    *   The `m3tal dash up` command uses the `m3tal-compose.local.yml` override file.
    *   This override directly binds `8082:8082` (or `${DASHBOARD_PORT:-8082}:8082`) to the host, making the dashboard directly accessible on the host's IP address and port 8082.
    *   **Access via**: `http://YOUR_HOST_IP:8082` or `http://localhost:8082`.
    *   **Best for**: Local network setups, first-time testing, or when you don't require domain-based access via Traefik.

2.  **Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)**:
    *   The `m3tal dash up` command uses the `m3tal-compose.traefik.yml` override file.
    *   This override adds Docker labels to the dashboard container, which Traefik uses for automatic service discovery. Traefik then routes traffic from a specified domain to the dashboard container's internal port (8082).
    *   **Access via**: `http://dash.YOUR_DOMAIN` (where `YOUR_DOMAIN` is configured in `/etc/m3tal/.env`).
    *   **Requires**: Traefik to be running (`m3tal up`) and configured with the `DOMAIN` variable.
    *   **Best for**: Domain-based access, integrating with other services behind Traefik, or when you want TLS (HTTPS) via Traefik.

---

### Docker / Compose Runtime

M3TAL orchestrates services using **Docker Engine** and **Docker Compose V2**. These are core dependencies for M3TAL's functionality.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files found in the `/docker/` directory. This includes the `routing-compose.yml` (for Traefik) and the `api-compose.yml` (for the M3TAL API daemon).
*   The `m3tal dash up` command specifically manages the `m3tal-dashboard` container. It ensures the correct Compose override (local or Traefik) is applied based on your `DASHBOARD_EXPOSE_MODE` setting before starting the dashboard.
*   Your own custom Docker Compose stacks should be placed as `*-compose.yml` files within the `/docker/` directory.

---

### Day 2 Operations: Installing New Stacks

To deploy additional Docker Compose stacks with M3TAL:

1.  Create your Docker Compose file (e.g., `my-app-compose.yml`).
2.  Place the file in the `/docker/` directory.
    ```bash
    sudo mv my-app-compose.yml /docker/
    ```
3.  Ensure any environment variables required by your stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
4.  Run `m3tal up` to start all services, including your new stack:
    ```bash
    m3tal up
    ```
    This command will detect and start your new Compose file along with existing services.

---

### M3TAL System Architecture Overview

*   **CLI binary (`/usr/bin/m3tal`)**: Your primary interaction point, installed via APT.
*   **API daemon (`m3tal-api.service`)**: A Go binary running as a systemd service on port 8080. It manages Docker, the state database (`/var/lib/m3tal/state.db`), and provides API routes.
*   **Dashboard container (`m3tal-dashboard`)**: A Python/Flask application running in a Docker container (internally on port 8082). It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway (`routing-compose.yml`)**: A reverse proxy Docker container that exposes services on port 80, using domain names. It discovers services via Docker labels and loads dynamic configuration from `/etc/traefik/dynamic`.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container for secure, zero-config internet access, part of the routing stack.