```markdown
# M3TAL Getting Started Guide

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before you begin, ensure you have the following installed on your system:

- **Docker Engine**
- **Docker Compose V2**

Verify your installation by running the following commands in your terminal:

```bash
docker --version
docker compose version
```

If these commands do not output version information, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL via APT

Install the M3TAL CLI binary and its core components using the following commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

Initialize and configure M3TAL by running the configuration wizard:

```bash
sudo m3tal config wizard
```

You will be prompted with several questions to configure your M3TAL environment. Here's what each prompt means:

- **`Enter desired port for Traefik HTTP entrypoint (default: 80):`**: This is the port Traefik will listen on for incoming HTTP traffic. The default of `80` is standard for HTTP.
- **`Enter desired port for M3TAL API (default: 8080):`**: This is the port the M3TAL API daemon will listen on. It's used for internal communication and management.
- **`Enter desired port for M3TAL Dashboard (default: 8082):`**: This is the port the M3TAL Dashboard will be accessible on.
- **`Enter desired mode for Dashboard exposure (local/traefik) (default: local):`**:
    - `local`: The dashboard will be directly accessible via its port (`8082` by default). This is simpler for initial setup and LAN-only access.
    - `traefik`: The dashboard will be exposed through Traefik, typically via a subdomain. This is more advanced and requires Traefik to be running.
- **`Enter your desired domain for Traefik (default: localhost):`**: If you are using Traefik to expose services (e.g., `dash.yourdomain.com`), enter your primary domain here. For local testing, `localhost` is sufficient.
- **`Enter your desired PUID (default: 1000):`**: The user ID for running containers. Usually your primary user's ID.
- **`Enter your desired PGID (default: 1000):`**: The group ID for running containers. Usually your primary user's group ID.
- **`Enter desired Timezone (default: America/Denver):`**: Your local timezone for consistent logging and scheduling.
- **`Enter a secure password for the M3TAL Dashboard (default: change_me_immediately):`**: This is the password you will use to log into the M3TAL Dashboard. **It is highly recommended to change this from the default immediately.**
- **`Enter a secure API Token for M3TAL (default: change_me_api_token):`**: A token for authenticating API requests. **Change this from the default.**
- **`Enter desired LOG_LEVEL (info/debug/warn/error) (default: info):`**: Sets the verbosity of M3TAL's logs. `debug` is useful for troubleshooting.

These settings will be saved to `/etc/m3tal/.env`.

## Step 4: Start the Routing Stack (Traefik)

The `m3tal up` command starts all Docker Compose services defined in the `/docker/` directory. This includes the Traefik reverse proxy, which is essential for routing traffic to your services.

```bash
m3tal up
```

This command will:
- Read all `*.yml` files in `/docker/`.
- Build and start the necessary Docker containers (Traefik, and any other services you've added).
- Set up networking for these containers.

## Step 5: Start the Dashboard

The M3TAL Dashboard provides a web interface to manage your M3TAL services.

```bash
m3tal dash up
```

This command will:
- Pull the latest `m3tal-dashboard` Docker image if it's not already present.
- Start the M3TAL Dashboard container.
- If `DASHBOARD_EXPOSE_MODE` is set to `local` in `/etc/m3tal/.env`, it will bind the dashboard's port directly to your host.
- If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, it will configure Traefik to route traffic to the dashboard.

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address.

- **If `DASHBOARD_EXPOSE_MODE=local`**: Access the dashboard at `http://YOUR_IP:8082` (replace `YOUR_IP` with your server's IP address, or `http://localhost:8082` if accessing from the same machine).
- **If `DASHBOARD_EXPOSE_MODE=traefik`**: Access the dashboard at `http://dash.DOMAIN` (replace `DOMAIN` with the domain you configured in Step 3, e.g., `http://dash.localhost`).

## Step 7: Log In

You will be presented with a login screen.

- **Username**: `admin`
- **Password**: The password you set during the configuration wizard (or `admin_pass` if you skipped setting one and are using default environment variables for testing).

**It is crucial to change the default password immediately for security.** You can do this via the command line:

```bash
sudo m3tal dashpass
```

This command will prompt you to enter a new password for the dashboard.

---

## Filesystem Contract

M3TAL relies on a specific filesystem structure for configuration and state management.

| Path                     | Purpose                                                                                              | Notes                                                                                      |
| :----------------------- | :--------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary M3TAL environment configuration file. Stores all user-defined settings.                      | Managed by `m3tal config wizard` and `m3tal config set`.                                 |
| `/var/lib/m3tal/state.db`| SQLite database storing M3TAL's operational state, including service configurations and metadata.  | Automatically created and managed by the M3TAL API daemon.                                 |
| `/opt/m3tal/stack/`      | The canonical directory for M3TAL's core Docker Compose files and Traefik configuration.             | Contains essential Compose files and Traefik configuration.                                |
| `/docker`                | **Symlink** pointing to `/opt/m3tal/stack/`. This is the user-facing path for all M3TAL stack operations. | Any `*-compose.yml` files placed here will be managed by `m3tal up`.                       |
| `/docker/users.json`     | Stores M3TAL Dashboard credentials.                                                                  | Managed by `m3tal dashpass` and used when `DASHBOARD_EXPOSE_MODE=local`.                   |

## Port Map

The following ports are utilized by M3TAL and its components.

| Port | Service             | Access                                       |
| :--- | :------------------ | :------------------------------------------- |
| 80   | Traefik HTTP        | Public (if `DASHBOARD_EXPOSE_MODE=traefik`)  |
| 8080 | M3TAL API daemon    | Host-local (for internal communication)      |
| 8081 | Traefik Dashboard   | Host-local only                              |
| 8082 | M3TAL Dashboard     | Direct port (if `local` mode) or via Traefik |

## Firewall Note

If you are exposing M3TAL services to the internet via Traefik (e.g., `DASHBOARD_EXPOSE_MODE=traefik`), ensure that port 80 (and port 443 if using HTTPS) is open in your firewall.

For example, if using `ufw`:

```bash
sudo ufw allow 80
# If you configure HTTPS later:
# sudo ufw allow 443
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

- **Check status:**
  ```bash
  systemctl status m3tal-api
  ```

- **View logs:**
  ```bash
  journalctl -u m3tal-api -f
  ```
  (Press `Ctrl+C` to exit the log stream)

- **Restart the service:**
  ```bash
  sudo systemctl restart m3tal-api
  ```

- **Enable service on boot:**
  ```bash
  sudo systemctl enable m3tal-api
  ```

- **Disable service on boot:**
  ```bash
  sudo systemctl disable m3tal-api
  ```
```