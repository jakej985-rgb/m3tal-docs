# Getting Started with M3TAL

This guide covers the initial setup and configuration of your M3TAL ecosystem. Follow these steps to get your M3TAL instance operational.

## 1. Prerequisites

M3TAL requires a functional Docker environment. Ensure **Docker Engine** and **Docker Compose V2** are installed on your system.

You can verify their installation and versions with the following commands:

```bash
docker --version && docker compose version
```

## 2. Install M3TAL via APT

Install the M3TAL CLI and API daemon using the provided APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, run the configuration wizard to set up your M3TAL instance. This wizard will guide you through setting essential environment variables in `/etc/m3tal/.env`.

```bash
sudo m3tal config wizard
```

During the wizard, you will be prompted for various settings:

*   **DOMAIN**: The primary domain for your services (e.g., `example.com`). This is used for Traefik routing. Defaults to `localhost`.
*   **DASHBOARD_EXPOSE_MODE**: Choose `local` or `traefik`.
    *   `local`: Dashboard is accessible directly via `http://YOUR_IP:8082`. Best for initial setup or LAN-only access.
    *   `traefik`: Dashboard is accessible via `http://dash.YOUR_DOMAIN`. Requires Traefik to be running.
*   **PUID/PGID**: User and Group IDs for container processes to ensure correct file permissions. Defaults to `1000`.
*   **TZ**: Your timezone (e.g., `America/Denver`).
*   **BASE_STORAGE_PATH**: The base directory where M3TAL stores data, configurations, and media. Defaults to `/mnt`.
*   **DASHBOARD_SECRET**: A secret key used by the dashboard for session management. Generated automatically.
*   **API_TOKEN**: An authentication token for the M3TAL API. Generated automatically.
*   **ADMIN_PASSWORD**: The initial password for the M3TAL Dashboard administrator account.

Review each prompt and provide appropriate values. Once complete, your `/etc/m3tal/.env` file will be populated.

## 4. Start the Routing Stack (Traefik)

Start the core routing services, including the Traefik reverse proxy. This command initializes and runs all Docker Compose files found in the `/docker/` directory (which symlinks to `/opt/m3tal/stack/`).

```bash
m3tal up
```

This will start services defined in `routing-compose.yml` (Traefik, and optionally Cloudflared). Traefik will listen on port 80 (HTTP).

## 5. Start the M3TAL Dashboard

Deploy the M3TAL Dashboard container. This command pulls the necessary Docker image, reads your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration override.

```bash
m3tal dash up
```

## 6. Access the Dashboard

Open your web browser and navigate to the M3TAL Dashboard based on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    `http://YOUR_SERVER_IP:8082` (replace `YOUR_SERVER_IP` with the actual IP address of your M3TAL host).
    Alternatively, if accessing from the host itself: `http://localhost:8082`.
*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured in the wizard). Traefik must be running (`m3tal up` executed).

## 7. Log In

Use the `admin` username and the `ADMIN_PASSWORD` you set during the `m3tal config wizard` to log in.

**Change Default Credentials:**
It is highly recommended to change the default admin password immediately. Use the following command:

```bash
sudo m3tal dashpass
```

This command will prompt you to set a new password for the `admin` user.

## Filesystem Contract

M3TAL adheres to a defined filesystem structure for its operation and configuration.

| Path                     | Purpose                                                                 |
| :----------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.           |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the M3TAL API daemon.            |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal stack Docker Compose files.    |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. Place your custom compose files here. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.                |

## Port Map

The following ports are used by M3TAL components:

| Port | Service            | Access                                        |
| :--- | :----------------- | :-------------------------------------------- |
| 80   | Traefik HTTP       | Public (if Traefik is exposed)                |
| 8080 | M3TAL API daemon   | Host-local only (internal communication)      |
| 8081 | Traefik Dashboard  | Host-local only (for Traefik's own dashboard) |
| 8082 | M3TAL Dashboard    | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Configuration

If you have a firewall enabled (e.g., `ufw`), you may need to allow traffic on the ports used by M3TAL.

*   **For Traefik (if `DASHBOARD_EXPOSE_MODE=traefik` or if exposing other services):**
    ```bash
    sudo ufw allow 80/tcp
    ```
*   **For direct dashboard access (if `DASHBOARD_EXPOSE_MODE=local`):**
    ```bash
    sudo ufw allow 8082/tcp
    ```
    (Ensure this port is not blocked if you need external access to the dashboard in `local` mode.)

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using standard `systemctl` commands.

*   **Check API service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View API daemon logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```

## Understanding M3TAL Dashboard Access Modes

M3TAL offers two primary ways to access its dashboard, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Mode 1: Local Access (`DASHBOARD_EXPOSE_MODE=local`)

*   **Configuration:** Uses `m3tal-compose.local.yml` as an override. This file directly binds port `8082` from the host to the dashboard container's internal port `8082`.
*   **Access URL:** `http://YOUR_SERVER_IP:8082` or `http://localhost:8082`.
*   **Use Case:** Ideal for initial setup, environments where you prefer direct IP-based access, or when running M3TAL on a home server for LAN-only usage. Traefik is not required for dashboard access in this mode.

### Mode 2: Traefik Access (`DASHBOARD_EXPOSE_MODE=traefik`)

*   **Configuration:** Uses `m3tal-compose.traefik.yml` as an override. This file adds specific Docker labels to the dashboard container, which Traefik uses for automatic service discovery and routing. Traefik will then route requests for `dash.${DOMAIN}` to the dashboard container.
*   **Access URL:** `http://dash.YOUR_DOMAIN`.
*   **Use Case:** Recommended for production deployments, domain-based access, and when integrating the M3TAL dashboard into a larger ecosystem managed by Traefik. Requires the Traefik routing stack to be active (`m3tal up`).

The `m3tal dash up` command intelligently applies the correct Docker Compose override based on your configured `DASHBOARD_EXPOSE_MODE`.

## Understanding Docker/Compose Runtime

M3TAL leverages **Docker Engine** and **Docker Compose V2** as its underlying container orchestration tools. These are fundamental dependencies.

*   The `m3tal up` command acts as a wrapper around `docker compose`. It executes `docker compose --file <file1> --file <file2> ... up -d` across all `*-compose.yml` files located in the `/docker/` directory (e.g., `routing-compose.yml`, user-defined stacks).
*   The `m3tal dash up` command specifically manages the M3TAL Dashboard container:
    1.  It downloads the latest `m3tal-compose.yml` (base configuration), `m3tal-compose.local.yml` (local access override), and `m3tal-compose.traefik.yml` (Traefik access override) from the M3TAL GitHub repository.
    2.  It reads the `DASHBOARD_EXPOSE_MODE` variable from your `/etc/m3tal/.env` file.
    3.  It then starts the dashboard container using the base `m3tal-compose.yml` and the appropriate override file (either `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`).

## Day 2 Operations: Deploying User Stacks

Once M3TAL is set up, you can easily deploy additional Docker Compose stacks:

1.  **Create your Docker Compose file:** Place your `my-stack-compose.yml` file into the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`).
2.  **Configure Environment Variables:** If your stack requires specific environment variables, define them in `/etc/m3tal/.env`. You can use `sudo m3tal config wizard` to set common variables or `sudo m3tal config set KEY value` for individual variables.
3.  **Start all stacks:** Run `m3tal up`. This command will detect your new compose file in `/docker/` and start your services alongside the existing M3TAL and routing components.