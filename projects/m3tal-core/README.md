# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary and API daemon are distributed via an APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core Components
The M3TAL system is comprised of several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare Tunnel container for establishing zero-configuration internet access to services.

## Filesystem Contract
The following table outlines the critical file system paths and their purposes within the M3TAL system:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file, managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`   | SQLite state database, automatically created by the API daemon.      |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik dynamic configurations. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store, managed by `m3tal dashpass`.             |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside. For user convenience and interaction, `/docker` is a symlink alias to `/opt/m3tal/stack/`, making `/docker/` the primary user-facing directory for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Execute `m3tal up` to start all deployed stacks, including your newly added one.

## Quick Demo

To quickly get started and run specific M3TAL components:

*   **Start the Dashboard:**
    To start only the M3TAL dashboard container, execute:
    ```bash
    m3tal dash up
    ```
    This command specifically downloads the necessary dashboard Docker Compose files and starts the dashboard with the appropriate configuration based on your `DASHBOARD_EXPOSE_MODE` setting.

*   **Deploy all M3TAL Stacks:**
    To orchestrate and deploy all M3TAL-managed Docker Compose stacks, including any user-defined compose files placed in the `/docker/` directory, use:
    ```bash
    m3tal up
    ```

## Dashboard Access
The M3TAL Dashboard provides a web-based interface for system management. Its access method is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for a new installation).
*   **Mechanism:** This mode utilizes `m3tal-compose.local.yml` to add a direct port binding, exposing the dashboard container's internal port `8082` directly on the host machine.
*   **Access:** A new user performing a default installation can access the dashboard directly via their host's IP address and port `8082`, for example: `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik gateway is required for this access method. It is ideal for LAN-only setups or initial testing.

### Traefik Mode
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** This mode uses `m3tal-compose.traefik.yml` to apply Traefik labels to the dashboard service. Traefik then routes traffic from a specified domain (`dash.DOMAIN`) to the dashboard container on its internal port `8082`.
*   **Access:** Access is via `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   **Requirements:** The Traefik gateway must be running (`m3tal up` will start it if configured) and properly configured with a `DOMAIN` in `/etc/m3tal/.env`.

## Traefik Gateway
Traefik functions as the central reverse proxy for services deployed within the M3TAL ecosystem. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik dynamically loads configuration from files within `/docker/dynamic/` (which is symlinked from `/opt/m3tal/stack/dynamic/`). This allows for hot-reloading of routing rules.

*   **Routing to Host-Local Services:** Traefik uses dynamic configuration files to route requests to services listening on host-local ports. For instance, `dynamic/api.yml` routes `api.DOMAIN` to the M3TAL Go API daemon, which listens on the host-local port `8080`, via `http://host.docker.internal:8080`.
*   **Routing to Containerized Services:**
    *   `dash.DOMAIN` routes to the `m3tal-dashboard` container when `DASHBOARD_EXPOSE_MODE=traefik` is configured, leveraging Traefik labels within `m3tal-compose.traefik.yml`.

### Exposing a Custom User Service via Traefik
To expose a custom service running in a user-defined Docker Compose stack (e.g., `my-app-compose.yml`) through Traefik, add the appropriate labels to its service definition:

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-custom-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN}`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network for Traefik discovery

networks:
  proxy:
    external: true
```
After placing this file in `/docker/` and running `m3tal up`, your application will be accessible at `http://app.DOMAIN`.

## Service Management
The M3TAL API daemon is managed as a systemd service.

*   **Check Status:** `systemctl status m3tal-api.service`
*   **Restart Service:** `systemctl restart m3tal-api.service`
*   **View Logs:** `journalctl -u m3tal-api.service -f`

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :----------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                                    |
| 8080 | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                                               |
| 8081 | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine.                      |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration
M3TAL uses environment variables for configuration, primarily managed through the `/etc/m3tal/.env` file. The `m3tal config wizard` command assists in setting up these variables.

### Environment Variables
The following environment variables are commonly used within the M3TAL system, with their default values:

| Key                      | Default Value             | Description                                          |
| :----------------------- | :------------------------ | :--------------------------------------------------- |
| `API_PORT`               | `5050`                    | Internal API port.                                   |
| `DASHBOARD_PORT`         | `8082`                    | Internal Dashboard container port.                   |
| `DASHBOARD_EXPOSE_MODE`  | `local`                   | Defines how the dashboard is exposed (`local` or `traefik`). |
| `HTTP_PORT`              | `8080`                    | The M3TAL API daemon's listening port.               |
| `STATE_DIR`              | `./state`                 | Directory for the SQLite state database.             |
| `LOG_LEVEL`              | `info`                    | Logging verbosity level.                             |
| `DASHBOARD_SECRET`       | `change_me_immediately`   | Secret key for dashboard session management.         |
| `API_TOKEN`              | `change_me_api_token`     | Authentication token for API access.                 |
| `ADMIN_PASSWORD`         | `admin_pass`              | Default password for the admin user.                 |
| `NETWORK_NAME`           | `m3tal`                   | Default Docker network for M3TAL components.         |
| `LOCAL_IP`               | `127.0.0.1`               | Local IP address for host-internal communication.    |
| `DOMAIN`                 | `localhost`               | Base domain for Traefik routing.                     |
| `VPN_USER`               | `user`                    | Username for VPN services.                           |
| `VPN_PASSWORD`           | `password`                | Password for VPN services.                           |
| `BASE_STORAGE_PATH`      | `./data`                  | Base directory for persistent storage.               |
| `MEDIA_PATH`             | `./data/media`            | Path for media files.                                |
| `CONFIG_PATH`            | `./data/config`           | Path for configuration files.                        |
| `DOWNLOADS_PATH`         | `./data/downloads`        | Path for downloaded content.                         |
| `PUID`                   | `1000`                    | User ID for container processes.                     |
| `PGID`                   | `1000`                    | Group ID for container processes.                    |
| `TZ`                     | `America/Denver`          | Timezone setting.                                    |
| `TRAEFIK_WEB_PORT`       | `80`                      | Traefik's public HTTP entry point.                   |
| `TRAEFIK_WEBHTTPS_PORT`  | `443`                     | Traefik's public HTTPS entry point.                  |
| `TRAEFIK_DASHBOARD_PORT` | `8080`                    | Internal Traefik dashboard port (host-local `8081`). |
| `DEBUG_MODE`             | `false`                   | Enables debug logging.                               |
| `METRICS_ENABLED`        | `true`                    | Enables system metrics collection.                   |