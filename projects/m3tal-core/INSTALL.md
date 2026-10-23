# M3TAL Get Started Guide

This guide provides step-by-step instructions for setting up M3TAL for the first time.

---

## Step 1: Prerequisites

Before proceeding, ensure you have the following installed on your system:

*   **Docker Engine:** The containerization platform.
*   **Docker Compose V2:** The tool for defining and running multi-container Docker applications.

Verify your installation by running the following commands in your terminal:

```bash
docker --version && docker compose version
```

---

## Step 2: Install M3TAL via APT

Install M3TAL using the official APT repository with the following commands:

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

After installation, run the M3TAL configuration wizard to set up your initial environment.

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's an explanation of each:

*   **`DASHBOARD_EXPOSE_MODE`**: Choose how you want to access the M3TAL dashboard.
    *   `local`: The dashboard will be accessible directly via its port on your host IP. This is recommended for initial setup and LAN-only access.
    *   `traefik`: The dashboard will be exposed through Traefik, allowing access via a domain name if Traefik is configured.
*   **`DOMAIN`**: If you choose `traefik` mode, enter the domain name you want to use for accessing services (e.g., `mydomain.com`). For `local` mode, `localhost` is generally sufficient.
*   **`DASHBOARD_PORT`**: The port on which the dashboard will be accessible (default is `8082`).
*   **`API_PORT`**: The port on which the M3TAL API daemon will listen (default is `5050`).
*   **`HTTP_PORT`**: The port for general HTTP access, often used by Traefik (default is `8080`).
*   **`STATE_DIR`**: The directory where M3TAL will store its state database and configuration (default is `./state` relative to the execution context, but typically resolved to a persistent location).
*   **`LOG_LEVEL`**: The verbosity of M3TAL logs (`debug`, `info`, `warn`, `error`).
*   **`DASHBOARD_SECRET`**: A secret key for securing the dashboard (a strong, unique value is recommended).
*   **`API_TOKEN`**: A token for authenticating API requests (a strong, unique value is recommended).
*   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard (a strong, unique value is recommended).
*   **`PUID` / `PGID`**: The User and Group IDs to run Docker containers as. These typically match your user's IDs. You can find yours by running `id -u` and `id -g`.
*   **`TZ`**: Your timezone (e.g., `America/Denver`).

---

## Step 4: Start the Routing Stack (Traefik)

M3TAL uses Docker Compose to manage its services. The `m3tal up` command orchestrates all compose files located in `/docker/`.

To start the core routing stack, including Traefik, run:

```bash
m3tal up
```

This command will parse and execute all `*-compose.yml` files found in the `/docker/` directory, starting services like Traefik.

---

## Step 5: Start the Dashboard

Now, start the M3TAL dashboard. This command specifically manages the dashboard container based on your configuration.

```bash
m3tal dash up
```

This command will:
1.  Pull the latest `m3tal-dashboard` Docker image.
2.  Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3.  Start the `m3tal-dashboard` container, applying either the `m3tal-compose.local.yml` (for `local` mode) or `m3tal-compose.traefik.yml` (for `traefik` mode) override.

---

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard URL:

*   If you set `DASHBOARD_EXPOSE_MODE` to `local`:
    `http://YOUR_IP:8082`
    (Replace `YOUR_IP` with your server's IP address. If running locally, you can often use `http://localhost:8082`).
*   If you set `DASHBOARD_EXPOSE_MODE` to `traefik`:
    `http://dash.DOMAIN`
    (Replace `DOMAIN` with the domain you configured, and ensure Traefik is running via `m3tal up`).

---

## Step 7: Log In to the Dashboard

You will be presented with the M3TAL login screen.

*   **Username**: `admin`
*   **Password**: The `ADMIN_PASSWORD` you set during the `m3tal config wizard`.

**Changing Your Password:**

To change the dashboard administrator password after logging in, use the following command:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new password.

---

## Filesystem Contract

M3TAL relies on a specific filesystem structure for its operation:

| Path                    | Purpose                                                                                                               |
| :---------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`       | The primary configuration file for M3TAL. This file is managed by the `m3tal config wizard` and `m3tal config set` commands. |
| `/var/lib/m3tal/state.db` | The SQLite database where M3TAL stores its operational state. This file is automatically created by the API daemon.   |
| `/docker`               | A symbolic link that points to `/opt/m3tal/stack/`. This is the user-facing directory for managing Docker Compose files for your stacks. |

---

## Port Map

The following ports are utilized by M3TAL and its components:

| Port | Service         | Access Method / Purpose                                  |
| :--- | :-------------- | :------------------------------------------------------- |
| 80   | Traefik HTTP    | Public access point for services when Traefik is enabled. |
| 8080 | M3TAL API       | Host-local access for the API daemon (port 5050 is default from env). |
| 8082 | M3TAL Dashboard | Direct port access (if `DASHBOARD_EXPOSE_MODE=local`) or via Traefik. |

---

## Firewall Note

If Traefik is exposed to the internet (e.g., on a public server), ensure that port 80 (and port 443 if HTTPS is configured) is allowed through your firewall. For example, if using `ufw`:

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
*   **View logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```