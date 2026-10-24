# M3TAL - Get Started

This guide will walk you through the initial setup of the M3TAL ecosystem.

---

## Step 1: Prerequisites

Before you begin, ensure you have the following software installed on your system:

*   **Docker Engine**: The foundational containerization platform.
*   **Docker Compose V2**: For managing multi-container Docker applications.

You can verify your installations by running the following commands in your terminal:

```bash
docker --version
docker compose version
```

If these commands return version information, you are ready to proceed. If not, please refer to the official Docker documentation for installation instructions.

---

## Step 2: Install M3TAL via APT

M3TAL provides an official APT repository for easy installation and updates. Execute the following commands in your terminal:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

This will download and install the M3TAL CLI binary and set up the necessary systemd service for the API daemon.

---

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through setting up essential environment variables. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted for the following settings:

*   **`PUID` / `PGID`**: These are your User ID (UID) and Group ID (GID). They are used to set file permissions for M3TAL's data directories. You can usually find these by running `id -u` and `id -g` in your terminal.
*   **`TZ`**: Your local timezone (e.g., `America/New_York`, `Europe/London`). This is important for accurate logging and scheduling.
*   **`DOMAIN`**: The domain name you wish to use for M3TAL services (e.g., `mydomain.com`, `localhost`).
*   **`DASHBOARD_EXPOSE_MODE`**: Determines how the M3TAL dashboard is accessed.
    *   `local`: The dashboard will be directly accessible via its port (default 8082). This is recommended for initial setup and LAN-only access.
    *   `traefik`: The dashboard will be exposed via Traefik, typically at `dash.YOUR_DOMAIN`. This requires Traefik to be running.
*   **`API_PORT` / `DASHBOARD_PORT`**: The ports for the M3TAL API and Dashboard respectively.
*   **`DASHBOARD_SECRET`**: A secret key for the dashboard. **Change this from the default immediately for security.**
*   **`API_TOKEN`**: A token for API authentication. **Change this from the default immediately for security.**
*   **`ADMIN_PASSWORD`**: The password for accessing the M3TAL dashboard. **Change this from the default immediately for security.**
*   **`BASE_STORAGE_PATH`**: The base directory where M3TAL will store its data and configurations.

The wizard will save your choices to `/etc/m3tal/.env`.

---

## Step 4: Start the Routing Stack (Traefik)

This step starts the Traefik reverse proxy, which will manage incoming network traffic for your M3TAL services.

```bash
m3tal up
```

This command reads all Docker Compose files located in `/docker/` and starts the corresponding services. This includes Traefik, which is essential for routing traffic to other M3TAL components if you're using the `traefik` dashboard expose mode.

---

## Step 5: Start the Dashboard

Now, start the M3TAL dashboard.

```bash
m3tal dash up
```

This command will:
1. Pull the latest `m3tal-dashboard` Docker image if it's not already present.
2. Read your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`.
3. Start the dashboard container, configured according to your chosen expose mode.

---

## Step 6: Access the M3TAL Dashboard

Open your web browser and navigate to the dashboard URL. The address depends on your `DASHBOARD_EXPOSE_MODE` setting:

*   **If `DASHBOARD_EXPOSE_MODE=local`**: Access the dashboard at `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or use `http://localhost:8082` if accessing from the server itself).
*   **If `DASHBOARD_EXPOSE_MODE=traefik`**: Access the dashboard at `http://dash.YOUR_DOMAIN` (replace `YOUR_DOMAIN` with the domain you configured). Ensure Traefik is running (`m3tal up` has completed successfully).

---

## Step 7: Log In to the Dashboard

Upon accessing the dashboard, you will be prompted to log in.

*   **Default Credentials:**
    *   **Username:** `admin`
    *   **Password:** The `ADMIN_PASSWORD` you set during the configuration wizard (or `admin_pass` if you used the default and haven't changed it).

**It is highly recommended to change the default password immediately after your first login.**

To change the dashboard password from the command line, use:

```bash
sudo m3tal dashpass admin NEW_PASSWORD
```

Replace `admin` with your username and `NEW_PASSWORD` with your desired secure password.

---

## Filesystem Contract

M3TAL utilizes specific directories and files for its configuration and state. Understanding this contract is crucial for managing your installation:

| Path                      | Purpose                                                                                                 | Managed By                                             |
| :------------------------ | :------------------------------------------------------------------------------------------------------ | :----------------------------------------------------- |
| `/etc/m3tal/.env`         | Primary environment configuration file. Stores all key-value settings.                                  | `m3tal config wizard`, `m3tal config set`, manual edit |
| `/var/lib/m3tal/state.db` | SQLite database storing M3TAL's internal state, including registered services and configurations.     | M3TAL API daemon                                       |
| `/opt/m3tal/stack/`       | Canonical directory containing M3TAL's core Docker Compose files and Traefik configuration.             | M3TAL installation                                     |
| `/docker`                 | A symbolic link that points to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. | M3TAL installation                                     |
| `/docker/users.json`      | Stores dashboard user credentials.                                                                      | `m3tal dashpass`                                       |

---

## Port Map

The following ports are used by M3TAL and its components:

| Port | Service           | Access                                 |
| :--- | :---------------- | :------------------------------------- |
| 80   | Traefik (HTTP)    | Public (if Traefik is exposed)         |
| 8080 | M3TAL API Daemon  | Host-local                             |
| 8081 | Traefik Dashboard | Host-local only (unless explicitly configured otherwise) |
| 8082 | M3TAL Dashboard   | Direct port (local mode) or via Traefik (traefik mode) |

---

## Firewall Note

If you are exposing M3TAL services to the internet via Traefik, ensure that port 80 (HTTP) is allowed through your firewall. For example, if you are using `ufw`:

```bash
sudo ufw allow 80
```

---

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its status and view logs using the following commands:

*   **Check service status:**
    ```bash
    systemctl status m3tal-api
    ```
*   **View real-time logs:**
    ```bash
    journalctl -u m3tal-api -f
    ```
*   **Restart the service:**
    ```bash
    sudo systemctl restart m3tal-api
    ```