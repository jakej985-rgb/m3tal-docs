# M3TAL System Documentation

This guide will walk you through the initial setup of the M3TAL system.

## GETTING STARTED

This document provides a step-by-step guide for new users to install and configure M3TAL.

### Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running the following commands:

```bash
docker --version
docker compose version
```

If these commands do not return version information, please refer to the official Docker documentation for installation instructions.

### Step 2: Install M3TAL via APT

Install M3TAL using the provided APT repository. This process involves adding the M3TAL signing key and repository, followed by the package installation.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

### Step 3: Run the Configuration Wizard

Initialize M3TAL's configuration by running the setup wizard. This wizard will guide you through setting essential parameters for your M3TAL instance.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions:

*   **`Enter your desired domain name [localhost]:`**: This is the domain name that will be used for accessing your M3TAL services, particularly when Traefik is configured. For local testing, `localhost` is suitable.
*   **`Enter the timezone for your server [America/Denver]:`**: Select your server's timezone. This is important for accurate logging and scheduling.
*   **`Enter the PUID (Process User ID) [1000]:`**: This is the User ID that Docker containers will run as. It's typically your primary user's ID.
*   **`Enter the PGID (Process Group ID) [1000]:`**: This is the Group ID that Docker containers will run as. It's typically your primary user's group ID.
*   **`Enter the base path for configuration files [/etc/m3tal/]:`**: This is where M3TAL stores its primary configuration file (`.env`).
*   **`Enter the base path for Docker compose files [/opt/m3tal/stack/]:`**: This is the canonical location for M3TAL's Docker Compose files. A symlink will be created at `/docker`.
*   **`Enter the base path for M3TAL state and data [/var/lib/m3tal/]:`**: This directory will be used for storing M3TAL's state database and other runtime data.
*   **`Enter the port for the M3TAL API [8080]:`**: The port on which the M3TAL API daemon will listen.
*   **`Enter the port for the M3TAL Dashboard [8082]:`**: The port on which the M3TAL dashboard will be accessible.
*   **`Dashboard expose mode (local/traefik) [local]:`**:
    *   `local`: Exposes the dashboard directly on the specified `DASHBOARD_PORT`. This is ideal for local network access and initial setup.
    *   `traefik`: Exposes the dashboard through Traefik, accessible via a subdomain (e.g., `dash.YOUR_DOMAIN`). This requires Traefik to be running.
*   **`Enter a strong password for the dashboard [change_me_immediately]:`**: Set a secure password for accessing the M3TAL dashboard. **It is highly recommended to change this from the default.**
*   **`Enter an API token for the M3TAL API [change_me_api_token]:`**: This token is used for API authentication. **Change this from the default for security.**

### Step 4: Start the Routing Stack (Traefik)

M3TAL utilizes Traefik as a reverse proxy to manage incoming traffic to your services. The `m3tal up` command orchestrates all Docker Compose files located within the `/docker/` directory, starting Traefik and any other configured services.

```bash
m3tal up
```

This command will:
1.  Identify all `*-compose.yml` files within `/docker/`.
2.  Execute `docker compose up -d` for each identified compose file, bringing up all services in detached mode. This includes Traefik, which will be configured to route traffic.

### Step 5: Start the Dashboard

The M3TAL dashboard provides a user interface for managing your M3TAL system. The `m3tal dash up` command specifically manages the dashboard container.

```bash
m3tal dash up
```

This command will:
1.  Download the necessary Docker Compose files for the dashboard from GitHub.
2.  Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Pull the `m3tal-godash:debug` Docker image if it's not already present.
4.  Start the `m3tal-dashboard` container using the appropriate configuration (either direct port binding for `local` mode or Traefik labels for `traefik` mode).

### Step 6: Access the Dashboard

You can now access the M3TAL dashboard through your web browser. The access method depends on your `DASHBOARD_EXPOSE_MODE` setting.

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):**
    Open your browser to: `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. If accessing from the same machine, you can use `http://localhost:8082`.)

*   **If `DASHBOARD_EXPOSE_MODE=traefik`:**
    Open your browser to: `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain name you configured during the wizard. Traefik must be running for this to work.)

### Step 7: Log In

Upon accessing the dashboard, you will be presented with a login screen.

*   **Username:** `admin`
*   **Password:** The password you set during the `sudo m3tal config wizard` step (default is `admin_pass`).

**To change the dashboard password after initial setup:**

```bash
sudo m3tal dashpass
```

This command will prompt you to enter a new password for the dashboard.

---

## FILESYSTEM CONTRACT

This section details the critical file locations and their purposes within the M3TAL ecosystem.

| Path                      | Purpose                                                                       | Managed By                               |
| :------------------------ | :---------------------------------------------------------------------------- | :--------------------------------------- |
| `/etc/m3tal/.env`         | Primary configuration file for M3TAL. Stores environment variables.           | `m3tal config wizard`, `m3tal config set` |
| `/var/lib/m3tal/state.db` | SQLite database for M3TAL's operational state.                                | M3TAL API daemon                         |
| `/opt/m3tal/stack/`       | Canonical directory for M3TAL's Docker Compose files and related configurations. | M3TAL system installation                |
| `/docker`                 | Symlink to `/opt/m3tal/stack/`. User-facing path for all stack operations.    | M3TAL system installation                |
| `/docker/users.json`      | Stores dashboard credentials.                                                 | `m3tal dashpass`                         |

---

## PORT TABLE

The following table outlines the ports used by M3TAL services.

| Port   | Service          | Access                                           | Notes                                                |
| :----- | :--------------- | :----------------------------------------------- | :--------------------------------------------------- |
| `80`   | Traefik HTTP     | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)    | Traefik's main entry point for HTTP traffic.         |
| `443`  | Traefik HTTPS    | Public (if HTTPS is configured and enabled)      | Traefik's main entry point for HTTPS traffic.        |
| `8080` | M3TAL API daemon | Host-local                                       | The core API service.                                |
| `8081` | Traefik Dashboard| Host-local only                                  | Traefik's own administrative dashboard.              |
| `8082` | M3TAL Dashboard  | Direct port (`local` mode) or via Traefik (`traefik` mode) | The M3TAL user interface.                            |

---

## FIREWALL NOTE

If your M3TAL instance is exposed to the internet and you are using Traefik (`DASHBOARD_EXPOSE_MODE=traefik`), ensure that port 80 (and 443 if HTTPS is configured) is open in your firewall.

For systems using `ufw` (Uncomplicated Firewall):

```bash
sudo ufw allow 80
# If you have configured HTTPS with Traefik, also allow port 443:
# sudo ufw allow 443
```

---

## SERVICE MANAGEMENT

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

**Checking the status of the M3TAL API service:**

```bash
systemctl status m3tal-api
```

**Viewing real-time logs for the M3TAL API service:**

```bash
journalctl -u m3tal-api -f
```