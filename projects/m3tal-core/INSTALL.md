# M3TAL: Getting Started Guide

This guide will walk you through the initial setup of the M3TAL system.

## Step 1: Prerequisites

Before proceeding, ensure you have the following installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installations by running:

```bash
docker --version && docker compose version
```

If Docker or Docker Compose are not installed, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL

Install M3TAL using the official APT repository. Run the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential system parameters. Run the command:

```bash
sudo m3tal config wizard
```

You will be prompted with the following questions:

*   **`API_PORT`**: The port on which the M3TAL API daemon will listen. The default is `5050`. You can change this if the default port is already in use.
*   **`API_TOKEN`**: A secret token used for authenticating API requests. It's recommended to set a strong, unique token. This token will be used by the dashboard and any other integrations.
*   **`M3TAL_STATE_DB`**: The path to the SQLite database where M3TAL stores its state. The default is typically automatically determined. You can specify a custom path if desired.
*   **`M3TAL_PLUGINS_DIR`**: The directory where M3TAL will look for plugin configurations. The default is typically automatically determined.

## Step 4: Start the Routing Stack (Traefik)

This command starts the core M3TAL routing stack, primarily Traefik, which acts as your reverse proxy. It will read all `*-compose.yml` files present in the `/docker/` directory and start the defined services.

```bash
m3tal up
```

## Step 5: Start the Dashboard

This command specifically pulls the latest M3TAL dashboard image and starts its container. The dashboard provides a web interface to manage your M3TAL services.

```bash
m3tal dash up
```

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address. The exact URL depends on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env` and your Traefik configuration:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), access the dashboard via: `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or `localhost` if accessing from the same machine).
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is configured with a domain, access it via: `http://dash.DOMAIN` (replace `DOMAIN` with your configured domain).

## Step 7: Log In to the Dashboard

When you first access the dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `admin`

**Important:** It is highly recommended to change these default credentials immediately for security. You can change the dashboard password by running the following command on your server:

```bash
sudo m3tal dashpass
```

This command will prompt you to enter a new password for the dashboard user.

## Filesystem Contract

M3TAL utilizes a specific filesystem structure for configuration and state management:

| Path                    | Purpose                                                                                                 |
| :---------------------- | :------------------------------------------------------------------------------------------------------ |
| `/etc/m3tal/.env`       | The primary configuration file for M3TAL. Managed by the `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | The SQLite database where M3TAL stores its operational state. Auto-created by the API daemon.         |
| `/docker`               | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing directory for all stack operations. |
| `/docker/users.json`    | Stores the dashboard credentials. Managed by the `m3tal dashpass` command.                              |

## Port Map

The following ports are used by M3TAL services:

| Port | Service        | Access                                   |
| :--- | :------------- | :--------------------------------------- |
| 80   | Traefik        | Public (when `traefik` mode is enabled)  |
| 8080 | M3TAL API      | Host-local                               |
| 8081 | Traefik Dash   | Host-local only                          |
| 8082 | M3TAL Dashboard| Direct port (when `local` mode is enabled) or via Traefik |

## Firewall Note

If you are exposing Traefik to the public internet (e.g., for domain-based access), ensure that port 80 is allowed through your firewall. For example, if you are using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its status and view logs using the following commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View logs in real-time:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```