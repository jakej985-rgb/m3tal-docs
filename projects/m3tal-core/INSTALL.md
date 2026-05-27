# M3TAL Ecosystem: Get Started

This guide will walk you through the installation and initial setup of the M3TAL Ecosystem.

## Table of Contents

1.  [Prerequisites](#step-1-prerequisites)
2.  [Install M3TAL](#step-2-install-m3tal-via-apt)
3.  [Run Configuration Wizard](#step-3-run-the-configuration-wizard)
4.  [Start Routing Stack (Traefik)](#step-4-start-the-routing-stack-traefik)
5.  [Start Dashboard](#step-5-start-the-dashboard)
6.  [Access Dashboard](#step-6-open-browser-at-http-your_ip-8082-or-http-dashdomain-if-traefik-is-configured)
7.  [Log In](#step-7-log-in)
8.  [Filesystem Contract](#filesystem-contract)
9.  [Port Map](#port-map)
10. [Firewall Note](#firewall-note)
11. [Service Management](#service-management)

---

## Step 1: Prerequisites

Docker Engine and Docker Compose V2 must be installed. Verify your installation:

```bash
docker --version && docker compose version
```

If these are not installed, please refer to the official Docker documentation for installation instructions.

---

## Step 2: Install M3TAL via APT

Use the following commands to add the M3TAL repository and install the `m3tal` package:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

## Step 3: Run the Configuration Wizard

The `m3tal config wizard` will guide you through setting up essential configuration.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's a breakdown of what each prompt means:

*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL dashboard is exposed.
    *   `local`: Exposes the dashboard directly via a port on your host machine. This is the default and recommended for initial setup. Access will be via `http://YOUR_IP:8082`.
    *   `traefik`: Exposes the dashboard through Traefik as a reverse proxy, typically via a subdomain like `dash.YOUR_DOMAIN`. This requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The port the dashboard will listen on. The default is `8082`.
*   **`HTTP_PORT`**: The port the M3TAL API daemon will listen on. The default is `8080`.
*   **`STATE_DIR`**: The directory where M3TAL stores its state data, including the `state.db` SQLite database. The default is `./state` relative to the configuration path, but will be resolved to `/var/lib/m3tal/state` after setup.
*   **`LOG_LEVEL`**: The verbosity of logs. Options include `debug`, `info`, `warn`, `error`. `info` is the default.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. It's highly recommended to change this from the default.
*   **`API_TOKEN`**: An API token for authentication with the M3TAL API. It's highly recommended to change this from the default.
*   **`ADMIN_PASSWORD`**: The password for the dashboard administrator. It's highly recommended to change this from the default.
*   **`NETWORK_NAME`**: The name of the Docker network M3TAL will use. The default is `m3tal`.
*   **`LOCAL_IP`**: Your local network IP address. If unsure, you can often leave this blank and M3TAL will attempt to detect it.
*   **`DOMAIN`**: The primary domain you intend to use with M3TAL services. `localhost` is the default for local testing.
*   **`VPN_USER`**: Username for any VPN configurations.
*   **`VPN_PASSWORD`**: Password for any VPN configurations.
*   **`BASE_STORAGE_PATH`**: The base directory for storing M3TAL data. Defaults to `./data`.
*   **`MEDIA_PATH`**: Directory for media files. Defaults to `./data/media`.
*   **`CONFIG_PATH`**: Directory for configuration files. Defaults to `./data/config`.
*   **`DOWNLOADS_PATH`**: Directory for downloads. Defaults to `./data/downloads`.
*   **`PUID`**: The User ID for Docker containers. Defaults to `1000`.
*   **`PGID`**: The Group ID for Docker containers. Defaults to `1000`.
*   **`TZ`**: Your local timezone. Defaults to `America/Denver`.
*   **`TRAEFIK_WEB_PORT`**: The port Traefik listens on for HTTP traffic. Defaults to `80`.
*   **`TRAEFIK_WEBHTTPS_PORT`**: The port Traefik listens on for HTTPS traffic. Defaults to `443`.
*   **`TRAEFIK_DASHBOARD_PORT`**: The port Traefik uses internally for its own dashboard. Defaults to `8080`.
*   **`DEBUG_MODE`**: Enable or disable debug logging. Defaults to `false`.
*   **`METRICS_ENABLED`**: Enable or disable metrics collection. Defaults to `true`.

---

## Step 4: Start the Routing Stack (Traefik)

This command starts all Docker Compose files located in the `/docker/` directory, which includes Traefik for routing and other core M3TAL services.

```bash
m3tal up
```

This command uses Docker Compose V2 to orchestrate the containers defined in the compose files found under `/docker/`. This typically includes Traefik, which acts as the entry point for your services.

---

## Step 5: Start the Dashboard

This command specifically pulls the M3TAL dashboard image and starts its container, respecting the `DASHBOARD_EXPOSE_MODE` setting from your configuration.

```bash
m3tal dash up
```

This command will:
1.  Download the necessary `m3tal-compose.yml` and any relevant override files (like `m3tal-compose.local.yml` or `m3tal-compose.traefik.yml`) from GitHub.
2.  Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Start the `m3tal-dashboard` container using Docker Compose, applying the correct port bindings or Traefik labels based on your chosen mode.

---

## Step 6: Open Browser at `http://YOUR_IP:8082` (or `http://dash.DOMAIN` if Traefik is configured)

*   **If `DASHBOARD_EXPOSE_MODE` is set to `local`:**
    Open your web browser and navigate to `http://YOUR_IP:8082`, where `YOUR_IP` is the IP address of the machine running M3TAL. If you're accessing it from the same machine, you can use `http://localhost:8082`.

*   **If `DASHBOARD_EXPOSE_MODE` is set to `traefik`:**
    Ensure Traefik is running (`m3tal up`). You should be able to access the dashboard via the domain you configured. Navigate to `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with your actual domain, or `dash.localhost` if you're using local testing with Traefik).

---

## Step 7: Log In

When you first access the M3TAL dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default value for `ADMIN_PASSWORD` in the environment variables. **It is strongly recommended to change this immediately.**)

*   **Changing Credentials:**
    You can change the dashboard administrator password using the following command:

    ```bash
    sudo m3tal dashpass
    ```
    Follow the prompts to set a new password.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure for configuration and data storage. Understanding these paths is crucial for managing your M3TAL installation.

| Path                      | Purpose                                                                                                                                                                   |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`         | The primary configuration file for M3TAL. This file is managed by the `m3tal config wizard` and `m3tal config set` commands. It stores all environment variables.           |
| `/var/lib/m3tal/state.db` | The SQLite state database for M3TAL. This file stores operational state, service configurations, and other critical data. It is automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`       | The canonical directory containing all M3TAL Docker Compose files, Traefik configuration, and other stack-related assets.                                               |
| `/docker`                 | A symbolic link that points to `/opt/m3tal/stack/`. This is the user-facing directory for all stack operations, including adding new compose files.                           |
| `/docker/users.json`      | Stores the dashboard credential information. This file is managed by the `m3tal dashpass` command.                                                                           |

---

## Port Map

| Port   | Service            | Access                                                 |
| :----- | :----------------- | :----------------------------------------------------- |
| 80     | Traefik HTTP       | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)          |
| 8080   | M3TAL API daemon   | Host-local only (accessible by other containers/services) |
| 8081   | Traefik Dashboard  | Host-local only (accessible via `localhost:8081`)      |
| 8082   | M3TAL Dashboard    | Direct port (when `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (when `DASHBOARD_EXPOSE_MODE=traefik`) |

---

## Firewall Note

If you have a firewall enabled on your system (e.g., `ufw`), and you intend to expose services via Traefik to the public internet, you will need to allow traffic on port 80.

```bash
sudo ufw allow 80
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View logs (follow in real-time):**
    ```bash
    journalctl -u m3tal-api -f
    ```

*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```