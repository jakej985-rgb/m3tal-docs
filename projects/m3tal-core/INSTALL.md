# M3TAL System - Get Started

This guide will walk you through the initial setup and configuration of the M3TAL system.

## 1. Prerequisites

Before you begin, ensure you have the following software installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running these commands:

```bash
docker --version
docker compose version
```

If either command fails, please refer to the official Docker documentation for installation instructions.

## 2. Install M3TAL

M3TAL can be installed on Debian-based systems (like Ubuntu) using our APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## 3. Run the Configuration Wizard

After installation, you will run the configuration wizard to set up M3TAL's core settings.

```bash
sudo m3tal config wizard
```

The wizard will prompt you for the following information:

*   **`DASHBOARD_EXPOSE_MODE`**:
    *   `local`: Exposes the dashboard directly via a port binding on your host machine. This is the default and recommended for initial setup on a local network.
    *   `traefik`: Configures the dashboard to be accessible via Traefik as a reverse proxy, typically using a domain name. Requires Traefik to be running.
*   **`DASHBOARD_PORT`**: The port on which the dashboard will be accessible. Defaults to `8082` for `local` mode.
*   **`DOMAIN`**: The domain name you intend to use for accessing M3TAL services (e.g., `localhost`, `m3tal.yourdomain.com`). This is primarily used when `DASHBOARD_EXPOSE_MODE` is set to `traefik`.
*   **`API_PORT`**: The port on which the M3TAL API daemon will run. Defaults to `8080`.
*   **`HTTP_PORT`**: The port for Traefik's HTTP entrypoint. Defaults to `80`.
*   **`STATE_DIR`**: The directory where M3TAL will store its persistent state (e.g., the SQLite database). Defaults to `./state` within the M3TAL installation directory.
*   **`LOG_LEVEL`**: Controls the verbosity of M3TAL logs. Options include `debug`, `info`, `warn`, `error`. Defaults to `info`.
*   **`DASHBOARD_SECRET`**: A secret key for securing your dashboard. **It is highly recommended to change this from the default.**
*   **`API_TOKEN`**: A token for authenticating with the M3TAL API. **It is highly recommended to change this from the default.**
*   **`ADMIN_PASSWORD`**: The password for the dashboard's admin user. **It is highly recommended to change this from the default.**
*   **`PUID` / `PGID`**: The User ID and Group ID that Docker containers will run as. Defaults to `1000`.
*   **`TZ`**: Your timezone (e.g., `America/Denver`).

These settings will be saved to `/etc/m3tal/.env`.

## 4. Start the Routing Stack (Traefik)

The routing stack, powered by Traefik, acts as your main entry point for services. Running `m3tal up` will orchestrate all Docker Compose files located within the `/docker/` directory, which includes the Traefik reverse proxy.

```bash
m3tal up
```

This command will pull necessary Docker images and start all defined services, including Traefik.

## 5. Start the Dashboard

Now, start the M3TAL dashboard. This command will pull the latest dashboard image and launch the container according to your `DASHBOARD_EXPOSE_MODE` setting.

```bash
m3tal dash up
```

## 6. Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard URL:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local`: `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address or `localhost`).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is running: `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured).

## 7. Log In

You will be presented with a login screen.

*   **Default Username**: `admin`
*   **Default Password**: `admin_pass` (or whatever you set for `ADMIN_PASSWORD` during the wizard).

**Important:** It is critical to change the default password immediately after logging in for security. You can do this via the dashboard's settings or by running:

```bash
sudo m3tal dashpass admin NEW_PASSWORD
```

Replace `NEW_PASSWORD` with your desired secure password.

## Filesystem Contract

M3TAL utilizes a defined filesystem structure for configuration and data storage.

| Path                     | Purpose                                                                 |
| :----------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for M3TAL. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's operational state.                     |
| `/docker`                | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations and compose files. |
| `/docker/users.json`     | Stores dashboard user credentials. Managed by `m3tal dashpass`.         |

## Port Map

| Port   | Service/Component     | Access Method                                      |
| :----- | :-------------------- | :------------------------------------------------- |
| `80`   | Traefik (HTTP)        | Public access if Traefik is exposed.               |
| `8080` | M3TAL API daemon (Go) | Host-local access (used internally by other services). |
| `8082` | M3TAL Dashboard       | Direct port access (`local` mode) or via Traefik.    |

## Firewall Note

If you are exposing Traefik to the public internet, ensure that port 80 (and potentially 443 if you configure HTTPS) is allowed through your firewall. For example, if you are using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`:

*   **Check status**:
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs in real-time**:
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service**:
    ```bash
    sudo systemctl restart m3tal-api
    ```