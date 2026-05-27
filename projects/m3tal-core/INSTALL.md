# M3TAL - Getting Started Guide

This guide will walk you through the initial setup of M3TAL.

## Step 1: Prerequisites

Before proceeding, ensure you have the following software installed on your system:

- **Docker Engine**
- **Docker Compose V2**

You can verify your installation by running the following command:

```bash
docker --version && docker compose version
```

If these commands do not output version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL

M3TAL is installed using APT. Execute the following three commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

After installation, you will run the M3TAL configuration wizard to set up essential parameters.

```bash
sudo m3tal config wizard
```

The wizard will present you with several prompts:

-   **`DASHBOARD_PORT`**: The port on which the M3TAL dashboard will be accessible. The default is `8082`.
-   **`DASHBOARD_EXPOSE_MODE`**: Determines how the dashboard is exposed.
    -   `local`: Exposes the dashboard directly via a port binding (e.g., `http://YOUR_IP:8082`). This is the default and recommended for initial setup.
    -   `traefik`: Exposes the dashboard through Traefik, accessible via a domain name (e.g., `http://dash.DOMAIN`). Requires Traefik to be running.
-   **`HTTP_PORT`**: The port for the M3TAL API daemon. The default is `8080`.
-   **`STATE_DIR`**: The directory where M3TAL stores its state information. The default is `./state`.
-   **`LOG_LEVEL`**: Controls the verbosity of M3TAL logs. Common options include `info`, `debug`, `warn`, `error`.
-   **`DASHBOARD_SECRET`**: A secret key used for securing the dashboard. It is highly recommended to change this from the default.
-   **`API_TOKEN`**: A token used for authenticating with the M3TAL API. It is highly recommended to change this from the default.
-   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard. It is highly recommended to change this from the default.
-   **`DOMAIN`**: The domain name used for services exposed via Traefik. Defaults to `localhost`.
-   **`PUID` / `PGID`**: The User ID and Group ID to run containers with. Defaults to `1000`.

## Step 4: Start the Routing Stack (Traefik)

This step starts Traefik, which acts as a reverse proxy for your M3TAL services.

```bash
m3tal up
```

This command will execute `docker compose` commands for all `.yml` files found within the `/docker/` directory. This includes the Traefik configuration.

## Step 5: Start the Dashboard

Now, you will start the M3TAL dashboard container.

```bash
m3tal dash up
```

This command will:
1. Download the necessary Docker Compose files for the dashboard.
2. Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3. Start the `m3tal-dashboard` container, applying the appropriate configuration based on your chosen expose mode.

## Step 6: Access the Dashboard

Open your web browser and navigate to one of the following addresses:

-   If `DASHBOARD_EXPOSE_MODE` is set to `local`:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. You can also use `http://localhost:8082` if accessing from the same machine.)

-   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is running:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured, e.g., `http://dash.localhost` or `http://dash.yourdomain.com`.)

## Step 7: Log In

Upon accessing the dashboard, you will be prompted to log in.

-   **Username:** `admin`
-   **Password:** The password you set during the configuration wizard (or the default `admin_pass` if you did not change it).

To change your dashboard password after initial login, use the following command:

```bash
sudo m3tal dashpass
```

## Filesystem Contract

M3TAL utilizes specific locations on your filesystem for configuration and state:

| Path                     | Purpose                                                                 |
| :----------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | The primary configuration file, managed by `m3tal config wizard`.       |
| `/var/lib/m3tal/state.db`| The SQLite state database for M3TAL. Auto-created by the API daemon.  |
| `/docker`                | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations and Docker Compose files. |
| `/docker/users.json`     | Stores dashboard credentials, managed by `m3tal dashpass`.              |

## Port Map

| Port   | Service               | Access Method                               |
| :----- | :-------------------- | :------------------------------------------ |
| 80     | Traefik HTTP          | Public (if Traefik is exposed externally)   |
| 8080   | M3TAL API daemon      | Host-local                                  |
| 8081   | Traefik Dashboard     | Host-local only                             |
| 8082   | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) |

## Firewall Note

If you are exposing Traefik to the internet, ensure that port 80 (and potentially 443 if using HTTPS) is allowed through your firewall. For example, if using `ufw`:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

-   **Check status:**
    ```bash
    systemctl status m3tal-api
    ```

-   **View logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```