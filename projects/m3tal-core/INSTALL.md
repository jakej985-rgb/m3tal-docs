# M3TAL: Getting Started

This guide will walk you through the initial setup of the M3TAL ecosystem.

---

## Step 1: Prerequisites

Before you can install M3TAL, ensure you have the following software installed on your system:

*   **Docker Engine**: The core containerization platform.
*   **Docker Compose V2**: The tool for defining and running multi-container Docker applications.

You can verify your installations by running the following commands:

```bash
docker --version
docker compose version
```

If these commands do not output version numbers, please refer to the official Docker documentation for installation instructions.

---

## Step 2: Install M3TAL via APT

M3TAL is distributed via an APT repository for easy installation and management. Run the following three commands in your terminal:

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

The `m3tal config wizard` command will guide you through setting up your M3TAL environment. It will prompt you to configure essential settings.

Run the wizard:

```bash
sudo m3tal config wizard
```

You will be presented with several prompts. Here's a brief explanation of each:

*   **`Enter a value for API_PORT [5050]:`**: The port the M3TAL API daemon will listen on. The default is 5050.
*   **`Enter a value for DASHBOARD_PORT [8082]:`**: The port the M3TAL dashboard will be accessible on. The default is 8082.
*   **`Enter a value for DASHBOARD_EXPOSE_MODE [local]:`**: This determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via a port binding. Ideal for initial setup and LAN access.
    *   `traefik`: Exposes the dashboard through Traefik, allowing access via a domain name. Requires Traefik to be running.
*   **`Enter a value for HTTP_PORT [8080]:`**: This setting appears to be for an internal API port and the default 8080 is generally suitable.
*   **`Enter a value for STATE_DIR [./state]:`**: The directory where M3TAL will store its persistent state, including the `state.db` file. The default is a relative path. For a system-wide installation, this might be adjusted by the installer.
*   **`Enter a value for LOG_LEVEL [info]:`**: Sets the verbosity of M3TAL's logs. `info` is standard. Other options include `debug`, `warn`, `error`.
*   **`Enter a value for DASHBOARD_SECRET [change_me_immediately]:`**: A secret key used for securing the dashboard. **It is highly recommended to change this from the default.**
*   **`Enter a value for API_TOKEN [change_me_api_token]:`**: A token for authenticating with the M3TAL API. **It is highly recommended to change this from the default.**
*   **`Enter a value for ADMIN_PASSWORD [admin_pass]:`**: The password for the dashboard's default administrator account. **It is highly recommended to change this from the default.**
*   **`Enter a value for NETWORK_NAME [m3tal]:`**: The name of the Docker network M3TAL will use.
*   **`Enter a value for LOCAL_IP [127.0.0.1]:`**: The local IP address to bind services to.
*   **`Enter a value for DOMAIN [localhost]:`**: The domain name to use for accessing services if Traefik is configured.
*   **`Enter a value for BASE_STORAGE_PATH [./data]:`**: The base path for M3TAL's data storage.
*   **`Enter a value for PUID [1000]:`**: The User ID to run Docker containers as.
*   **`Enter a value for PGID [1000]:`**: The Group ID to run Docker containers as.
*   **`Enter a value for TZ [America/Denver]:`**: Your system's timezone.

---

## Step 4: Start the Routing Stack (Traefik)

The M3TAL ecosystem uses Traefik as its reverse proxy and router. Starting the routing stack will launch Traefik, making your services accessible.

Run the following command:

```bash
m3tal up
```

This command orchestrates Docker Compose files located in `/docker/`. It will download necessary images and start the Traefik reverse proxy and any other services defined in the compose files within that directory.

---

## Step 5: Start the Dashboard

The M3TAL dashboard provides a web interface to manage your M3TAL services.

Run the following command to start the dashboard container:

```bash
m3tal dash up
```

This command will:
1.  Pull the latest `m3tal-dashboard` Docker image.
2.  Read the `DASHBOARD_EXPOSE_MODE` setting from your `/etc/m3tal/.env` file.
3.  Start the dashboard container, applying the correct configuration for exposing it (either via direct port binding or Traefik).

---

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address.

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (the default), access it at:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address or `localhost` if accessing from the same machine).

*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is running, you can access it via:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured, e.g., `dash.localhost`).

---

## Step 7: Log In to the Dashboard

Upon accessing the dashboard for the first time, you will be prompted to log in.

*   **Default Credentials:**
    *   **Username:** `admin`
    *   **Password:** `admin_pass` (or the password you set during the wizard if you changed it)

**It is strongly recommended to change the default password immediately after your first login.**

To change the dashboard password using the CLI:

```bash
sudo m3tal dashpass
```

This command will prompt you for the new password and update the credentials file used by the dashboard.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure for its configuration and data. Understanding these paths is crucial for management and troubleshooting.

| Path                       | Purpose                                                                 |
| :------------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`          | The primary M3TAL configuration file. Modified by the `m3tal config wizard` and `m3tal config set` commands. |
| `/var/lib/m3tal/state.db`  | The SQLite state database where M3TAL stores its operational data. Automatically created and managed by the M3TAL API daemon. |
| `/opt/m3tal/stack/`        | The canonical directory containing M3TAL's core Docker Compose files and Traefik configuration. |
| `/docker`                  | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations (e.g., adding custom compose files). |
| `/docker/users.json`       | Stores the M3TAL dashboard user credentials. Managed by the `m3tal dashpass` command. |

---

## Port Map

M3TAL and its components utilize the following network ports:

| Port | Service                       | Access                                            |
| :--- | :---------------------------- | :------------------------------------------------ |
| 80   | Traefik (HTTP entry point)    | Public (when `DASHBOARD_EXPOSE_MODE=traefik`)     |
| 8080 | M3TAL API daemon (Go)         | Host-local access only                            |
| 8081 | Traefik dashboard             | Host-local access only                            |
| 8082 | M3TAL Dashboard               | Direct port access (`local` mode) or via Traefik (`traefik` mode) |

---

## Firewall Configuration

If you are exposing Traefik to the public internet (e.g., for `DASHBOARD_EXPOSE_MODE=traefik` or other services), you will need to open port 80 (and potentially 443 for HTTPS) in your firewall.

If you are using `ufw` (Uncomplicated Firewall), run:

```bash
sudo ufw allow 80
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs for the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```

*   **Restart the M3TAL API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```