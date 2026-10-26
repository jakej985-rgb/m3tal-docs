# M3TAL CLI Command Reference

As DocSmith, the M3TAL Ecosystem Documentation Architect, my mission is to provide you with the definitive guide to interacting with your M3TAL system. This document serves as a complete cheat-sheet for the `m3tal` command-line interface (CLI), offering practical examples for every command and delving into the underlying architecture.

The `m3tal` CLI is your single entry point for managing the M3TAL Linux Desktop Application, headless services, configuration, and Docker-based stacks. It orchestrates complex operations with simple commands, ensuring a streamlined experience.

---

## M3TAL System Overview

M3TAL is a modular, Docker-centric platform managed by a Go CLI binary and an accompanying API daemon. Understanding its core components and filesystem layout is key to effective management.

### Core Components

*   **CLI Binary (`/usr/bin/m3tal`)**: A unified Go binary, installed via APT, serving as the primary user interface for all M3TAL operations.
*   **API Daemon (`m3tal-api.service`)**: A Go binary running as a systemd service on `http://localhost:8080`. It manages Docker interactions, maintains the SQLite state database, and exposes RESTful API routes.
*   **Dashboard Container (`m3tal-dashboard`)**: A Python/Flask web application running inside Docker, typically on port 8082 internally. It communicates with the `m3tal-api` daemon at `http://host.docker.internal:8080`.
*   **Traefik Gateway (`routing-compose.yml`)**: A powerful reverse proxy container that exposes M3TAL services (and your custom stacks) on standard HTTP/HTTPS ports (e.g., 80, 443). It uses Docker labels and a file provider for dynamic routing.
*   **Cloudflared (`routing-compose.yml`)**: An optional Cloudflare tunnel container, integrated with Traefik, providing secure, zero-config internet access to your services without opening firewall ports.

### Filesystem Contract

M3TAL establishes a clear filesystem contract to organize its configuration, data, and Docker stacks.

| Path                     | Purpose                                                                   |
| :----------------------- | :------------------------------------------------------------------------ |
| `/etc/m3tal/.env`        | Primary configuration file, containing all environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database, storing internal M3TAL state. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical directory for M3TAL's internal Docker Compose files and Traefik dynamic configuration. |
| `/docker`                | **User-facing symlink** to `/opt/m3tal/stack/`. This is where you place your custom `*-compose.yml` files. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.                   |
| `/docker/dynamic/`       | Directory within `/docker` where Traefik's dynamic file provider configuration is stored. |

### Traefik Routing Architecture

Traefik acts as the central ingress point for domain-based access.

*   It is deployed as a container via `/docker/routing-compose.yml`.
*   Binds host port 80 (HTTP) and optionally 443 (HTTPS with LetsEncrypt) as entry points.
*   Automatically discovers services by inspecting Docker container labels (e.g., `m3tal-dashboard` when `DASHBOARD_EXPOSE_MODE=traefik`).
*   Loads additional dynamic configuration from `/docker/dynamic/` (using a file provider with hot-reload).
*   **Example**: Routes `api.YOUR_DOMAIN` to the `m3tal-api` daemon (`http://host.docker.internal:8080`) via a definition in `/docker/dynamic/api.yml`.
*   **Example**: Routes `dash.YOUR_DOMAIN` to the `m3tal-dashboard` container via Traefik labels defined in `m3tal-compose.traefik.yml` (when `DASHBOARD_EXPOSE_MODE=traefik`).

### Port Map

| Port | Service                     | Access                                      |
| :--- | :-------------------------- | :------------------------------------------ |
| 80   | Traefik HTTP Entry Point    | Public (if `routing` stack is up)           |
| 443  | Traefik HTTPS Entry Point   | Public (if `routing` stack is up)           |
| 8080 | M3TAL API Daemon (Go)       | Host-local only                             |
| 8081 | Traefik Dashboard (internal)| Host-local only (accessed via `127.0.0.1:8081`) |
| 8082 | M3TAL Dashboard (container) | Direct port (local mode) or via Traefik (traefik mode) |

---

## M3TAL CLI Commands

The `m3tal` CLI provides a powerful and consistent interface for managing your M3TAL system.

### Installation

Before using the CLI, ensure M3TAL is installed on your system.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

---

### General Commands

#### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application. This command is typically used on a desktop environment where a graphical interface is desired.

**Usage Example:**
```bash
m3tal
```
Or:
```bash
m3tal ui
```

#### `m3tal headless [port]`

Starts M3TAL in headless mode, meaning the CLI itself runs as a background process, usually as part of a systemd service. This is ideal for home servers or environments without a graphical interface. If `port` is omitted, it defaults to the API daemon's configured port (typically 8080).

**Usage Example (Default Port):**
```bash
m3tal headless
```

**Usage Example (Custom Port):**
```bash
m3tal headless 8080
```
*(Note: In most production setups, the `m3tal-api.service` systemd unit manages the headless instance.)*

#### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics directly to your terminal. This provides a quick snapshot of your system's health and performance.

**Usage Example:**
```bash
m3tal stats
```

#### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. This command verifies crucial components such as Docker connectivity, the validity of your `/etc/m3tal/.env` file, and the availability of required ports. Essential for troubleshooting initial setup issues.

**Usage Example:**
```bash
m3tal doctor
```

---

### Configuration Management (`m3tal config`)

The `m3tal config` commands interact with `/etc/m3tal/.env`, M3TAL's primary configuration file.

#### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring or updating your `/etc/m3tal/.env` file. This is the recommended way to set up M3TAL initially or modify multiple variables.

**Usage Example:**
```bash
m3tal config wizard
```

#### `m3tal config set KEY VALUE`

Sets a single environment variable in `/etc/m3tal/.env`. This is useful for quick adjustments or automating specific configurations.

**Usage Example (Setting a domain):**
```bash
m3tal config set DOMAIN myhomelab.net
```

**Usage Example (Changing dashboard exposure mode):**
```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

#### `m3tal config get KEY`

Reads and prints the value of a single environment variable from `/etc/m3tal/.env`.

**Usage Example:**
```bash
m3tal config get PUID
```

#### `m3tal config scan`

Lists all environment variables known to M3TAL, including defaults and those defined in your `/etc/m3tal/.env` file. It aggregates variables across all internal stacks and provides a comprehensive overview.

**Usage Example:**
```bash
m3tal config scan
```

#### `m3tal config list`

Displays the current contents of your `/etc/m3tal/.env` file directly to the terminal, showing only the variables explicitly set in that file.

**Usage Example:**
```bash
m3tal config list
```

---

### Dashboard Management (`m3tal dash`)

The `m3tal dash` commands specifically manage the `m3tal-dashboard` container.

#### Dashboard Access Modes (Critical)

The M3TAL Dashboard has two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

1.  **`local` (Default Mode)**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
    *   **Mechanism**: Uses the `m3tal-compose.local.yml` override file, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
    *   **Access Method**: Access the dashboard directly via your host's IP address and port:
        *   `http://HOST_IP:8082`
        *   `http://localhost:8082` (if accessing from the host machine)
    *   **Requirements**: No Traefik required. Works out of the box on a home server or for first-time users.
    *   **Best For**: LAN-only setups, initial testing, users who don't need domain-based access.

2.  **`traefik` Mode**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
    *   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the dashboard container. Traefik (if running) routes incoming requests from a subdomain (e.g., `dash.YOUR_DOMAIN`) to the dashboard's internal port (8082).
    *   **Access Method**: Access the dashboard via its configured subdomain:
        *   `http://dash.YOUR_DOMAIN` (Requires Traefik to be running via `m3tal up` and your `DOMAIN` variable set in `/etc/m3tal/.env`).
    *   **Requirements**: Traefik must be deployed and running. The `DOMAIN` environment variable must be correctly configured in `/etc/m3tal/.env`.
    *   **Best For**: Domain-based setups, users managing multiple services behind a reverse proxy, or those integrating with Cloudflared for remote access.

#### `m3tal dashpass [username] [password]`

Updates or sets a user's password for the M3TAL dashboard. If no arguments are provided, it launches an interactive prompt to set a new password for the `admin` user. This command manages the `/docker/users.json` file.

**Usage Example (Interactive):**
```bash
m3tal dashpass
# Follow prompts to set a new password for the default 'admin' user.
```

**Usage Example (Directly setting password for 'admin'):**
```bash
m3tal dashpass admin SuperSecurePassword123
```

**Usage Example (Creating/updating another user):**
```bash
m3tal dashpass jake P@ssword4Jake!
```

#### `m3tal dash up`

Pulls the latest dashboard Compose configuration from GitHub (`m3tal-compose.yml`, `m3tal-compose.local.yml`, `m3tal-compose.traefik.yml`) to `/docker/`. It then starts the `m3tal-dashboard` container using the appropriate override file based on your `DASHBOARD_EXPOSE_MODE` setting.

**Usage Example:**
```bash
m3tal dash up
```

#### `m3tal dash down`

Stops and removes the `m3tal-dashboard` container.

**Usage Example:**
```bash
m3tal dash down
```

#### `m3tal dash restart`

Restarts the `m3tal-dashboard` container. Useful after configuration changes or for troubleshooting.

**Usage Example:**
```bash
m3tal dash restart
```

#### `m3tal dash logs`

Streams the logs from the `m3tal-dashboard` container to your terminal. This is invaluable for debugging dashboard issues.

**Usage Example:**
```bash
m3tal dash logs
```

#### `m3tal dash status`

Shows the current status of the `m3tal-dashboard` container (e.g., running, stopped, exited).

**Usage Example:**
```bash
m3tal dash status
```

---

### Stack Management (`m3tal up`/`down`/`logs`)

These commands manage all your Docker Compose stacks, including M3TAL's core components and any custom services you add.

#### Docker / Compose Runtime

M3TAL relies on **Docker Engine** and **Docker Compose V2** (typically integrated as `docker compose`) as fundamental dependencies. The CLI acts as an intelligent wrapper around these tools.

*   `m3tal up` translates into running `docker compose up -d` across all `*-compose.yml` files it finds in `/docker/`.
*   `m3tal down` translates into running `docker compose down` across all `*-compose.yml` files in `/docker/`.

#### Deployment Lifecycle — Day 2 Operations

To deploy a new Docker Compose-based service (a "stack") with M3TAL:

1.  **Place your Compose file**: Create or copy your `docker-compose.yml` (or `my-stack-compose.yml`) file into the `/docker/` directory.
    ```bash
    sudo cp ~/my-app/docker-compose.yml /docker/my-app-compose.yml
    ```
2.  **Configure environment variables**: If your stack requires specific environment variables (e.g., ports, paths, API keys), ensure they are set in `/etc/m3tal/.env`. Use `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for specific variables.
    ```bash
    m3tal config set MY_APP_PORT 9000
    m3tal config set MEDIA_PATH /mnt/mydata/media
    ```
3.  **Start all stacks**: Run `m3tal up` to bring up all Docker Compose stacks defined in `/docker/`, including your newly added service.
    ```bash
    m3tal up
    ```

#### `m3tal up`

Runs `docker compose up -d` across *all* `*-compose.yml` files found in the `/docker/` directory. This command starts or recreates all services defined in your M3TAL ecosystem.

**Usage Example:**
```bash
m3tal up
```

#### `m3tal down`

Runs `docker compose down` across *all* `*-compose.yml` files found in the `/docker/` directory. This command stops and removes all containers, networks, and volumes defined in your M3TAL stacks.

**Usage Example:**
```bash
m3tal down
```

#### `m3tal logs`

Streams aggregated logs from all currently running M3TAL containers to your terminal. This provides a unified view for monitoring and debugging your entire ecosystem.

**Usage Example:**
```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service, ensuring it starts automatically on boot and can be managed like other system services.

#### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api` systemd service. This shows if it's active, running, and provides recent log entries.

**Usage Example:**
```bash
sudo systemctl status m3tal-api
```

#### `systemctl restart m3tal-api`

Restarts the `m3tal-api` systemd service. This is often necessary after making changes to `/etc/m3tal/.env` that the API daemon needs to re-read.

**Usage Example:**
```bash
sudo systemctl restart m3tal-api
```

#### `journalctl -u m3tal-api -f`

Streams the full log output from the `m3tal-api` service in real-time. This is crucial for in-depth debugging and monitoring the API daemon's operations.

**Usage Example:**
```bash
sudo journalctl -u m3tal-api -f
```

---

## Docker Direct Commands (Fallback / Advanced)

While `m3tal` CLI abstracts away Docker Compose details, understanding the underlying commands can be invaluable for advanced debugging, specific container management, or when troubleshooting `m3tal` itself.

M3TAL uses **Docker Compose V2**, which is typically invoked as `docker compose` (note the space instead of a hyphen). All M3TAL's internal and user-defined compose files reside in `/docker/`.

To run direct `docker compose` commands, you typically need to specify the compose files using the `-f` flag. For M3TAL, this means including all relevant `*-compose.yml` files.

#### Direct `docker compose up`

To bring up *all* M3TAL and custom stacks manually:

```bash
cd /docker/
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f m3tal-compose.local.yml -f m3tal-compose.traefik.yml -f my-custom-stack-compose.yml up -d
```
*(You would dynamically add `-f your-other-stack-compose.yml` for every compose file present.)*

#### Direct `docker compose down`

To stop and remove *all* M3TAL and custom stacks manually:

```bash
cd /docker/
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f m3tal-compose.local.yml -f m3tal-compose.traefik.yml -f my-custom-stack-compose.yml down
```

#### Direct `docker compose logs`

To stream logs from *all* running containers managed by these compose files:

```bash
cd /docker/
sudo docker compose -f routing-compose.yml -f m3tal-compose.yml -f m3tal-compose.local.yml -f m3tal-compose.traefik.yml -f my-custom-stack-compose.yml logs -f
```

#### Managing a Single Container (e.g., the dashboard)

You can manage individual containers or services directly using `docker compose` by targeting specific services within a compose file.

To start *only* the dashboard in `local` mode:
```bash
cd /docker/
sudo docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d m3tal-dashboard
```

To view logs for *only* the dashboard:
```bash
sudo docker logs -f m3tal-dashboard
```

To stop *only* the dashboard:
```bash
sudo docker stop m3tal-dashboard
```

These direct Docker commands offer granular control and are excellent for debugging specific components or when the `m3tal` CLI might not be functioning as expected.