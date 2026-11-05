# Getting Started with M3TAL

This guide will walk you through the initial setup of the M3TAL ecosystem.

## Step 1: Prerequisites

Before you begin, ensure you have the following installed:

*   **Docker Engine**
*   **Docker Compose V2**

You can verify their installation by running:

```bash
docker --version && docker compose version
```

If these are not installed, please refer to the official Docker documentation for installation instructions.

## Step 2: Install M3TAL

Install M3TAL using the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Step 3: Run the Configuration Wizard

The M3TAL configuration wizard will guide you through essential setup options. Run the following command:

```bash
sudo m3tal config wizard
```

You will be prompted with the following questions:

*   **API Port**: This is the port on which the M3TAL API daemon will run. The default is `5050`. You can change this if the default port is already in use.
*   **API Token**: This is a security token for API access. It is recommended to set this for enhanced security. Leave it empty if you do not wish to set a token at this time.
*   **M3TAL State DB**: This specifies the path to the M3TAL state database. The default is `/var/lib/m3tal/state.db`. This file will be automatically created if it doesn't exist.
*   **M3TAL Plugins Dir**: This is the directory where custom M3TAL plugins can be placed. The default is usually an empty string, meaning no specific plugin directory is set by default.

## Step 4: Start the Routing Stack (Traefik)

The routing stack, managed by Traefik, acts as your system's reverse proxy and entry point. Start it using:

```bash
m3tal up
```

This command will orchestrate and start all Docker Compose files found in the `/docker/` directory. This includes Traefik itself, enabling it to route traffic to your deployed services.

## Step 5: Start the Dashboard

The M3TAL dashboard provides a web-based interface for managing your M3TAL system. To start the dashboard:

```bash
m3tal dash up
```

This command will pull the latest M3TAL dashboard Docker image and start the container. The `m3tal dash up` command respects the `DASHBOARD_EXPOSE_MODE` setting in your `/etc/m3tal/.env` file, determining how the dashboard is exposed (directly via port or through Traefik).

## Step 6: Access the Dashboard

Open your web browser and navigate to the dashboard's address:

*   If `DASHBOARD_EXPOSE_MODE` is set to `local` (default), access it at: `http://YOUR_IP:8082` or `http://localhost:8082`.
*   If `DASHBOARD_EXPOSE_MODE` is set to `traefik` and Traefik is configured with a domain, access it at: `http://dash.YOUR_DOMAIN`.

Replace `YOUR_IP` with your server's IP address and `YOUR_DOMAIN` with your configured domain.

## Step 7: Log In

Upon first access, you will be presented with a login screen.

*   **Default Credentials**:
    *   Username: `admin`
    *   Password: `admin`

**Important**: It is highly recommended to change these default credentials immediately for security. You can do this using the following command:

```bash
sudo m3tal dashpass
```

Follow the prompts to set a new secure password for the dashboard.

---

## Filesystem Contract

M3TAL utilizes specific directories and files for its operation and configuration. Understanding this contract is crucial for managing and troubleshooting your M3TAL installation.

| Path                 | Purpose                                                                                                                                                                 |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`    | The primary configuration file for M3TAL. This file stores environment variables and is managed by the `m3tal config wizard` and `m3tal config set` commands.             |
| `/var/lib/m3tal/`    | This directory contains persistent data for M3TAL.                                                                                                                      |
| `state.db` (within `/var/lib/m3tal/`) | The SQLite state database for M3TAL. This file is auto-created by the API daemon and stores essential system state information.                                    |
| `/opt/m3tal/stack/`  | The canonical directory containing M3TAL's core Docker Compose files, Traefik configuration, and other essential stack components.                                      |
| `/docker`            | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose-related operations, including adding custom stacks.                    |
| `/docker/users.json` | Stores the dashboard credentials. This file is managed exclusively by the `m3tal dashpass` command.                                                                      |

## Port Map

The following ports are utilized by M3TAL and its components:

| Port   | Service          | Access                                              |
| :----- | :--------------- | :-------------------------------------------------- |
| 80     | Traefik          | Public (if exposed and configured)                  |
| 8080   | M3TAL API Daemon | Host-local                                          |
| 8081   | Traefik Dashboard| Host-local only                                     |
| 8082   | M3TAL Dashboard  | Direct port (when `DASHBOARD_EXPOSE_MODE=local`) or via Traefik (when `DASHBOARD_EXPOSE_MODE=traefik`) |

## Firewall Note

If Traefik is exposed to the public internet (e.g., on a server outside your local network), you must ensure that port 80 is open in your firewall. If you are using `ufw`, you can allow traffic with:

```bash
sudo ufw allow 80
```

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

*   **Check the status of the M3TAL API service:**
    ```bash
    systemctl status m3tal-api
    ```

*   **View live logs from the M3TAL API service:**
    ```bash
    journalctl -u m3tal-api -f
    ```