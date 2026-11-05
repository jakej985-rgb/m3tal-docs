# GETTING STARTED WITH M3TAL

This guide walks you through the initial setup of M3TAL for first-time users.

---

## Step 1: Prerequisites

Before proceeding, ensure you have the following installed on your system.

Docker Engine and Docker Compose V2 must be installed.

Verify your installation:

```bash
docker --version
docker compose version
```

If these commands do not return version information, please install them before continuing.

---

## Step 2: Install M3TAL via APT

M3TAL is distributed via an APT repository for easy installation and updates.

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

The M3TAL configuration wizard will guide you through setting up essential parameters.

Run the wizard using the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions. Here's what each prompt means:

*   **`Enter API_PORT (default: 5050):`**: This is the port on which the M3TAL API daemon will listen. The default is 5050. You can press Enter to accept the default or provide a different port number.
*   **`Enter API_TOKEN (leave empty for none):`**: This is an optional API token for authentication. If you intend to secure your API, enter a strong token here. Otherwise, leave it blank by pressing Enter.
*   **`Enter M3TAL_STATE_DB (default: /var/lib/m3tal/state.db):`**: This specifies the path to the SQLite database used by M3TAL to store its state. The default location is `/var/lib/m3tal/state.db`. You can change this if desired.
*   **`Enter M3TAL_PLUGINS_DIR (default: /opt/m3tal/plugins):`**: This is the directory where M3TAL will look for any installed plugins. The default is `/opt/m3tal/plugins`.
*   **`Enter DASHBOARD_EXPOSE_MODE (local or traefik, default: local):`**: This setting determines how the M3TAL dashboard will be exposed.
    *   `local` (default): Exposes the dashboard directly via a port (8082). This is suitable for LAN-only access or initial setup.
    *   `traefik`: Configures the dashboard to be accessible through Traefik, typically via a subdomain. This requires Traefik to be running.

---

## Step 4: Start the Routing Stack (Traefik)

This command initiates the core M3TAL routing infrastructure, primarily Traefik.

```bash
m3tal up
```

This command reads all Docker Compose files located within the `/docker/` directory (which is a symlink to `/opt/m3tal/stack/`) and starts the services defined in them. This typically includes Traefik itself, which acts as the reverse proxy for other M3TAL services.

---

## Step 5: Start the Dashboard

After the routing stack is up, you can start the M3TAL dashboard.

```bash
m3tal dash up
```

This command will:
1.  Download the latest necessary Docker Compose configuration files for the dashboard from GitHub.
2.  Read the `DASHBOARD_EXPOSE_MODE` setting from your `/etc/m3tal/.env` file.
3.  Start the M3TAL dashboard container using the appropriate Docker Compose override file based on your `DASHBOARD_EXPOSE_MODE`.

---

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard address. The exact address depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE` is `local` (default):**
    Open your browser to `http://YOUR_IP:8082` or `http://localhost:8082`. Replace `YOUR_IP` with the actual IP address of your server.

*   **If `DASHBOARD_EXPOSE_MODE` is `traefik`:**
    Open your browser to `http://dash.DOMAIN`. Replace `dash.DOMAIN` with the domain name you have configured for the dashboard in Traefik. This requires Traefik to be running and properly configured.

---

## Step 7: Log In to the Dashboard

Upon accessing the dashboard, you will be prompted to log in.

*   **Default Credentials:**
    *   Username: `admin`
    *   Password: `m3tal`

*   **Changing Your Password:**
    To change the default password, use the following command:

    ```bash
    sudo m3tal dashpass
    ```

    Follow the prompts to enter your new password.

---

## Filesystem Contract

M3TAL relies on specific file locations for its operation. Understanding these paths is crucial for advanced configuration and troubleshooting.

| Path                      | Purpose                                                                    |
| :------------------------ | :------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | Primary M3TAL configuration file. Stores environment variables. Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's operational state. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`       | Canonical directory containing M3TAL's core Docker Compose files and Traefik configuration. |
| `/docker`                 | Symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing M3TAL stacks. New service compose files are typically placed here. |
| `/docker/users.json`      | Stores dashboard credentials. Managed by `m3tal dashpass`. |

---

## Port Map

M3TAL utilizes several ports for its services and external access.

| Port | Service             | Access                                       |
| :--- | :------------------ | :------------------------------------------- |
| 80   | Traefik             | Public (when `DASHBOARD_EXPOSE_MODE=traefik` and Traefik is running) |
| 8080 | M3TAL API Daemon    | Host-local access only                      |
| 8081 | Traefik Dashboard   | Host-local access only                      |
| 8082 | M3TAL Dashboard     | Direct port access (`local` mode) or via Traefik (`traefik` mode) |

---

## Firewall Note

If your M3TAL server is accessible from the internet and you are using Traefik to expose services (e.g., with `DASHBOARD_EXPOSE_MODE=traefik`), ensure that port 80 is open in your firewall.

If you are using `ufw`, you can allow port 80 with:

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

*   **View real-time logs for the M3TAL API service:**

    ```bash
    journalctl -u m3tal-api -f
    ```