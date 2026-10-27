# M3TAL Setup Guide

This guide will walk you through the process of installing and setting up M3TAL for the first time.

## Step 1: Prerequisites

Before you begin, ensure you have the following software installed on your system:

*   **Docker Engine**: The containerization platform.
*   **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

You can verify your installation by running the following commands in your terminal:

```bash
docker --version
docker compose version
```

If these commands do not output version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

M3TAL is distributed as a Debian package for easy installation. Follow these exact commands to add the M3TAL repository and install the `m3tal` package.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

After installation, you need to configure M3TAL. The built-in wizard will guide you through the essential settings.

Run the following command:

```bash
sudo m3tal config wizard
```

The wizard will present you with several prompts. Here's an explanation of each:

*   **`DASHBOARD_EXPOSE_MODE`**: This determines how the M3TAL dashboard is accessed.
    *   `local`: The dashboard will be directly accessible via its port (default 8082) on your host's IP address. This is the simplest mode for local network access.
    *   `traefik`: The dashboard will be accessible through Traefik, typically via a subdomain (e.g., `dash.yourdomain.com`). This requires Traefik to be running.
*   **`DOMAIN`**: The primary domain name for accessing your M3TAL services if you are using Traefik for routing. If you are using `local` mode for the dashboard, this value is less critical for the dashboard itself but might be used by other services.
*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. The default is 5050.
*   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will listen internally. The default is 8082.
*   **`HTTP_PORT`**: The port Traefik will use for its HTTP entry point. The default is 80.
*   **`PUID` / `PGID`**: The User ID (UID) and Group ID (GID) that Docker containers will run as. Typically, these should match your primary user's UID/GID to ensure proper file permissions. You can find your UID/GID by running `id -u` and `id -g` respectively.
*   **`TZ`**: Your system's Timezone. This is important for accurate logging and scheduling.
*   **`DASHBOARD_SECRET`**: A secret key for securing the dashboard. It's highly recommended to change this from the default.
*   **`API_TOKEN`**: A token for authenticating with the M3TAL API. Change this from the default.
*   **`ADMIN_PASSWORD`**: The password for the default administrator user in the M3TAL dashboard. **Strongly recommended to change this.**
*   **`BASE_STORAGE_PATH`**: The base directory where M3TAL will store persistent data, configuration, and media. Defaults to `./data` relative to where commands are run, but will be set to an absolute path during setup.
*   **`MEDIA_PATH`**: The path for media files.
*   **`CONFIG_PATH`**: The path for configuration files.
*   **`DOWNLOADS_PATH`**: The path for downloaded files.

The wizard will save these settings to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

The routing stack, primarily managed by Traefik, is essential for exposing your services.

To start the Traefik reverse proxy and other routing components, run:

```bash
m3tal up
```

This command orchestrates Docker Compose files located in the `/docker/` directory. It starts Traefik and any other services defined in the relevant compose files, making them ready to route traffic.

## Step 5: Start the Dashboard

Now, start the M3TAL dashboard container.

```bash
m3tal dash up
```

This command will:
1.  Fetch the necessary Docker Compose configuration for the dashboard.
2.  Consider your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Pull the latest `m3tal-dashboard` Docker image if it's not already present.
4.  Start the dashboard container.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` in `/etc/m3tal/.env`, access it at: `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's actual IP address, or use `http://localhost:8082` if on the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is configured correctly with your domain, you can access it at: `http://dash.DOMAIN` (replace `DOMAIN` with your configured domain).

## Step 7: Log In

You will be presented with the M3TAL dashboard login screen.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: `admin_pass` (This is the default value of `ADMIN_PASSWORD` from the wizard. **It is critical to change this.**)

Once logged in, you can manage your M3TAL services. To change the administrator password, use the following command:

```bash
sudo m3tal dashpass admin new_password_here
```

Replace `new_password_here` with your desired strong password.

---

## Filesystem Contract

M3TAL utilizes a defined filesystem structure for its configuration and state. Understanding these paths is crucial for managing and troubleshooting your M3TAL installation.

| Path                      | Purpose                                                                                             |
| :------------------------ | :-------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | The primary M3TAL configuration file. This is populated and managed by the `m3tal config wizard` and `m3tal config set` commands. It stores all environment variables for M3TAL services. |
| `/var/lib/m3tal/state.db` | An SQLite database file used by the M3TAL API daemon to store its operational state, service configurations, and other persistent data. This file is auto-created by the API daemon if it doesn't exist. |
| `/docker`                 | This directory acts as a symbolic link (symlink) pointing to `/opt/m3tal/stack/`. All Docker Compose files (`.yml`) and related configurations for user-defined stacks and the core M3TAL services should reside here. This is the user-facing path for stack operations via the `m3tal` CLI. |
| `/opt/m3tal/stack/`       | The canonical directory containing the M3TAL system's Docker Compose files, Traefik configurations, and other core assets. The `/docker` symlink points here. |
| `/docker/users.json`      | Stores the M3TAL dashboard's user credentials. This file is managed by the `m3tal dashpass` command. |

---

## Port Map

The following ports are used by M3TAL and its components. Ensure these ports are not in conflict with other services running on your system.

| Port   | Service                | Access                                                                               |
| :----- | :--------------------- | :----------------------------------------------------------------------------------- |
| 80     | Traefik (HTTP)         | Exposed to the public/network if `DASHBOARD_EXPOSE_MODE=traefik` or other services are configured for external access. |
| 8080   | M3TAL API daemon (Go)  | Accessible only from within the host system (or other Docker networks).           |
| 8081   | Traefik Dashboard      | Accessible locally on the host system only (`127.0.0.1:8081`).                      |
| 8082   | M3TAL Dashboard        | Directly accessible via host IP in `local` mode, or via Traefik in `traefik` mode. |

---

## Firewall Note

If Traefik is configured to expose services publicly (e.g., if `DASHBOARD_EXPOSE_MODE=traefik` and you want external access to services routed by Traefik), you will need to open port 80 (and potentially 443 for HTTPS) in your firewall.

If you are using `ufw` (Uncomplicated Firewall), you can allow traffic on port 80 with:

```bash
sudo ufw allow 80
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

### Checking Service Status

To check the status of the M3TAL API service:

```bash
systemctl status m3tal-api
```

### Viewing Logs

To follow the real-time logs of the M3TAL API service:

```bash
journalctl -u m3tal-api -f
```