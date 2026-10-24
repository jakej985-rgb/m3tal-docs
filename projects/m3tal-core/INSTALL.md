```markdown
# M3TAL: Getting Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Table of Contents

1.  [Prerequisites](#step-1-prerequisites)
2.  [Install M3TAL via APT](#step-2-install-m3tal-via-apt)
3.  [Run the Configuration Wizard](#step-3-run-the-configuration-wizard)
4.  [Start the Routing Stack (Traefik)](#step-4-start-the-routing-stack-traefik)
5.  [Start the Dashboard](#step-5-start-the-dashboard)
6.  [Access the Dashboard](#step-6-access-the-dashboard)
7.  [Log In](#step-7-log-in)

---

## Step 1: Prerequisites

Before you begin, ensure you have the following installed on your system:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify your installation by running the following command:

```bash
docker --version && docker compose version
```

If these are not installed, please refer to the official Docker documentation for installation instructions.

---

## Step 2: Install M3TAL via APT

M3TAL is installed using a dedicated APT repository. Execute the following three commands:

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

The `m3tal config wizard` command will guide you through setting up essential M3TAL configurations.

```bash
sudo m3tal config wizard
```

You will be prompted for various settings. Here's a breakdown of each prompt and its meaning:

*   **`Enter your desired domain [localhost]`**: This is the base domain name that M3TAL will use for internal service routing (e.g., `api.your_domain.com`, `dash.your_domain.com`). If you are running M3TAL on your local network without a public domain, `localhost` is a suitable default.
*   **`Enter your desired dashboard port [8082]`**: This is the port on which the M3TAL dashboard will be accessible. The default is `8082`.
*   **`Enter your desired API port [5050]`**: This is the port on which the M3TAL API daemon will be accessible. The default is `5050`.
*   **`Enter your desired Dashboard expose mode [local]`**: This determines how the dashboard is exposed.
    *   `local`: Exposes the dashboard directly via the specified `DASHBOARD_PORT`. This is recommended for initial setup and LAN-only access.
    *   `traefik`: Exposes the dashboard through Traefik, allowing access via `dash.YOUR_DOMAIN`. This requires Traefik to be running.
*   **`Enter your desired Log Level [info]`**: Sets the verbosity of M3TAL logs. Options typically include `debug`, `info`, `warn`, `error`.
*   **`Enter your desired Dashboard Secret [change_me_immediately]`**: This is a security token for your dashboard. **It is crucial to change this from the default value to something secure.**
*   **`Enter your desired API Token [change_me_api_token]`**: This is a token for authenticating with the M3TAL API. **Change this from the default value.**
*   **`Enter your desired Admin Password [admin_pass]`**: This is the password for the default administrator account in the dashboard. **Change this from the default value.**
*   **`Enter your desired User ID (PUID) [1000]`**: The User ID to run Docker containers under. Typically, this matches your primary user's UID.
*   **`Enter your desired Group ID (PGID) [1000]`**: The Group ID to run Docker containers under. Typically, this matches your primary user's GID.
*   **`Enter your desired Timezone [America/Denver]`**: Set this to your local timezone for accurate logging and scheduling.

The wizard will save these settings to `/etc/m3tal/.env`.

---

## Step 4: Start the Routing Stack (Traefik)

This command initiates the core routing infrastructure, primarily Traefik, which acts as a reverse proxy for your services.

```bash
m3tal up
```

This command will:
*   Locate all `*-compose.yml` files within the `/docker/` directory.
*   Use Docker Compose V2 to start the services defined in these files.
*   This includes starting the Traefik reverse proxy, which will manage incoming traffic based on domain names.

---

## Step 5: Start the Dashboard

This command is specific to starting and managing the M3TAL dashboard container.

```bash
m3tal dash up
```

This command will:
1.  Download the necessary M3TAL dashboard Docker image (`ghcr.io/jakej985-rgb/m3tal-godash:debug`).
2.  Read your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
3.  Start the dashboard container, applying the appropriate configuration for the chosen exposure mode.

---

## Step 6: Open Browser at `http://YOUR_IP:8082` (or `http://dash.DOMAIN` if Traefik is configured)

Access the M3TAL dashboard through your web browser. The exact address depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local` (default):** Open your browser and navigate to `http://YOUR_IP:8082`, where `YOUR_IP` is the IP address of the server where M3TAL is installed. You can also likely use `http://localhost:8082` if accessing from the same machine.
*   **If `DASHBOARD_EXPOSE_MODE=traefik`:** Open your browser and navigate to `http://dash.DOMAIN`, where `DOMAIN` is the domain you configured in Step 3. This mode requires Traefik to be running (`m3tal up`).

---

## Step 7: Log In

Upon reaching the dashboard, you will be presented with a login screen.

*   **Default Credentials:**
    *   **Username:** `admin`
    *   **Password:** The password you set during the configuration wizard (or `admin_pass` if you skipped changing it).

To change the dashboard password after initial login, use the following command:

```bash
sudo m3tal dashpass
```

This will prompt you for a new password and update the credential store.

---

## Filesystem Contract

M3TAL utilizes a specific filesystem structure for configuration and data. Understanding these paths is important for management and troubleshooting.

| Path                       | Purpose                                                                                             | Notes                                                                 |
| :------------------------- | :-------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------- |
| `/etc/m3tal/.env`          | Primary configuration file for M3TAL. Contains all user-defined settings.                           | Managed by `m3tal config wizard` and `m3tal config set`.            |
| `/var/lib/m3tal/state.db`  | SQLite database storing the operational state of M3TAL and its managed services.                    | Auto-created by the API daemon on first run.                          |
| `/docker`                  | **Symlink** pointing to the canonical stack directory. This is the user-facing path for stack ops.  | Symlinks to `/opt/m3tal/stack/`.                                      |
| `/opt/m3tal/stack/`        | The actual location of M3TAL's core Docker Compose files and Traefik configuration.                 | Internal M3TAL directory. All user commands operate relative to `/docker`. |
| `/docker/users.json`       | Stores dashboard user credentials.                                                                  | Managed by `m3tal dashpass`.                                          |

---

## Port Table

The following table outlines the key ports used by M3TAL and its components:

| Port   | Service / Component | Access                                          | Notes                                                      |
| :----- | :------------------ | :---------------------------------------------- | :--------------------------------------------------------- |
| **80** | Traefik             | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)     | Entry point for HTTP traffic when Traefik is exposed.      |
| **8080** | M3TAL API (Go)      | Host-local                                      | The core M3TAL API daemon, accessible by other services.   |
| **8081** | Traefik Dashboard   | Host-local only (`127.0.0.1:8081`)              | Traefik's administrative interface.                        |
| **8082** | M3TAL Dashboard     | Direct port (`http://YOUR_IP:8082`) or via Traefik | Accessible directly or routed by Traefik based on config. |

---

## Firewall Note

If Traefik is exposed to the public internet and you are using a firewall like `ufw`, ensure that port 80 (and potentially 443 for HTTPS if configured) is allowed:

```bash
sudo ufw allow 80
# If you configure HTTPS:
# sudo ufw allow 443
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs from the M3TAL API service:**
    ```bash
    sudo journalctl -u m3tal-api -f
    ```

*   **Restart the M3TAL API service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```

---

This concludes the initial setup guide. You should now have a functional M3TAL instance ready for further configuration and use.
```