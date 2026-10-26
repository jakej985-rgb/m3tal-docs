# docs/GET_STARTED.md

This document provides a complete, step-by-step guide for setting up M3TAL for the first time.

---

### Step 1: Prerequisites

M3TAL requires Docker Engine and Docker Compose V2. Verify their installation and version:

```bash
docker --version && docker compose version
```

If these are not installed, refer to the official Docker documentation for your operating system.

### Step 2: Install M3TAL via APT

Execute the following commands in your terminal to add the M3TAL APT repository and install the `m3tal` CLI:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

Initialize M3TAL's core configuration using the wizard. This process creates the `/etc/m3tal/.env` file with essential environment variables.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for several configuration settings. Below are explanations for common prompts:

*   **`DOMAIN`**: The primary domain name for services exposed via Traefik. Default: `localhost`. Example: `example.com`.
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL Dashboard is exposed.
    *   `local` (Default): The dashboard is accessible directly via a host port binding (e.g., `http://YOUR_IP:8082`). This mode does not require Traefik for dashboard access.
    *   `traefik`: The dashboard is routed through Traefik (e.g., `http://dash.YOUR_DOMAIN`). This mode requires Traefik to be running.
*   **`PUID` (User ID), `PGID` (Group ID)**: The user and group IDs that containers will run as, crucial for file permissions. You can find your current user's IDs with `id -u` and `id -g`. Default: `1000`.
*   **`TZ` (Timezone)**: Your local timezone (e.g., `America/New_York`). Default: `America/Denver`.
*   **`ADMIN_PASSWORD`**: The initial password for the default `admin` user of the M3TAL Dashboard. **Change this immediately after setup.** Default: `admin_pass`.
*   **`API_TOKEN`**: A secret token used for authenticating with the M3TAL API daemon. **Change this immediately after setup.** Default: `change_me_api_token`.
*   **Storage Paths (e.g., `BASE_STORAGE_PATH`, `CONFIG_PATH`, `MEDIA_PATH`, `DOWNLOADS_PATH`)**: These define the base directories for persistent data storage on your host system. Default: paths under `/mnt`.

### Step 4: Start the Routing Stack

Start the core routing services, including the Traefik reverse proxy. This command uses Docker Compose V2 to start all `*-compose.yml` files found in the `/docker/` directory.

```bash
m3tal up
```

This will bring up the `routing` stack, which typically includes Traefik and optionally Cloudflared. Traefik handles HTTP requests on port 80 and routes them to your services.

### Step 5: Start the Dashboard

Deploy the M3TAL Dashboard container. The `m3tal dash up` command downloads the latest dashboard compose files from GitHub, reads your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.

```bash
m3tal dash up
```

### Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local`**:
    *   Access via: `http://YOUR_SERVER_IP:8082` or `http://localhost:8082`
*   **If `DASHBOARD_EXPOSE_MODE=traefik`**:
    *   Access via: `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the value configured in `DOMAIN` during the wizard). Traefik must be running for this mode to work.

### Step 7: Log in

The default login credentials for the M3TAL Dashboard are:

*   **Username**: `admin`
*   **Password**: The value you set for `ADMIN_PASSWORD` during the `config wizard` (default: `admin_pass`).

To change the dashboard password for the `admin` user:

```bash
sudo m3tal dashpass
```

---

### Filesystem Contract

M3TAL uses specific locations for its configuration and state data:

| Path                        | Purpose                                                                                                 |
| :-------------------------- | :------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`           | Primary configuration file containing environment variables. Managed by `m3tal config wizard`.            |
| `/var/lib/m3tal/state.db`   | SQLite database storing M3TAL's internal state. Auto-created by the API daemon.                         |
| `/opt/m3tal/stack/`         | Canonical directory where M3TAL stores its core compose files and Traefik dynamic configurations.       |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing Docker stacks.|
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                                                |

---

### Port Table

M3TAL utilizes the following network ports:

| Port | Service                                  | Access                                     |
| :--- | :--------------------------------------- | :----------------------------------------- |
| 80   | Traefik HTTP entry point                 | Public (if Traefik is exposed)             |
| 8080 | M3TAL API daemon (Go)                    | Host-local                                 |
| 8081 | Traefik Dashboard (internal access only) | Host-local only (`127.0.0.1:8081`)         |
| 8082 | M3TAL Dashboard (Python/Flask)           | Direct port (local mode) or via Traefik (traefik mode) |

### Firewall Note

If you intend to access your M3TAL dashboard or any services routed through Traefik from outside your local network, you may need to open port 80 on your host's firewall. For `ufw` (Uncomplicated Firewall), use:

```bash
sudo ufw allow 80
```

### Service Management

The M3TAL API daemon runs as a `systemd` service. You can manage and monitor it using standard `systemctl` commands:

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View live logs**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```

---

### Understanding M3TAL's Docker Runtime

M3TAL leverages **Docker Engine + Docker Compose V2** for container orchestration.

*   The `m3tal up` command executes `docker compose` operations across all `*-compose.yml` files found within the `/docker/` directory. This is how M3TAL manages your entire stack, including its core components and any user-defined services.
*   The `m3tal dash up` command is specialized for the M3TAL Dashboard. It performs the following actions:
    1.  Downloads the latest `m3tal-compose.yml`, `m3tal-compose.local.yml`, and `m3tal-compose.traefik.yml` from the official GitHub repository.
    2.  Reads the `DASHBOARD_EXPOSE_MODE` variable from `/etc/m3tal/.env`.
    3.  Starts the dashboard container using the base `m3tal-compose.yml` along with the appropriate override file (`m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) to configure its network exposure.

### Deploying New Services (Day 2 Operations)

After initial setup, you can add new services to your M3TAL ecosystem:

1.  **Place your compose file**: Create a Docker Compose file (e.g., `my-service-compose.yml`) in the `/docker/` directory.
2.  **Configure environment variables**: If your new stack requires specific environment variables, set them in `/etc/m3tal/.env` using `m3tal config wizard` or `m3tal config set KEY value`.
3.  **Start all stacks**: Run `m3tal up` to deploy your new service along with any existing stacks. M3TAL will automatically discover and manage the new compose file.