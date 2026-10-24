# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
To install the M3TAL CLI and API daemon, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Components
The M3TAL system is composed of the following core components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations, installed via APT.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and exposes API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, internally listening on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services by domain name on host port `80`. It uses a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container providing Cloudflare Tunnel functionality for zero-configuration internet access.

## Filesystem Contract
The following table outlines the key directories and files used by the M3TAL system:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.   |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by the `m3tal dashpass` command. |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all M3TAL-managed Docker Compose stack files (e.g., `m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration files reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, serving as the primary user-facing interface for all stack management operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access
The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode uses an override file (`m3tal-compose.local.yml`) to add a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access Method**: The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik gateway is required. This mode works out of the box for first-time users and local installations. A new user performing a default M3TAL installation will access the dashboard directly via port 8082.
*   **Use Cases**: Ideal for LAN-only setups, initial configurations, and local development or testing.

### Traefik Mode
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode uses an override file (`m3tal-compose.traefik.yml`) to add specific Traefik labels to the dashboard container. Traefik then routes incoming requests for `dash.DOMAIN` to the dashboard container on its internal port 8082.
*   **Access Method**: The dashboard is accessible via `http://dash.DOMAIN`.
*   **Requirements**: The Traefik gateway must be running (typically started via `m3tal up`) and configured to handle the specified domain.
*   **Use Cases**: Suited for domain-based setups where multiple services are exposed through a single reverse proxy.

## Traefik Gateway
Traefik is deployed as a Docker container via `routing-compose.yml` and serves as the primary reverse proxy for M3TAL services. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds host port `80` (and optionally `443` for HTTPS) as its public HTTP entry point. It dynamically loads configuration from files within the `/docker/dynamic/` directory (which is symlinked from `/opt/m3tal/stack/dynamic`), allowing for hot-reloading of routing rules.

*   **Host-Local Service Routing**: Traefik can route requests to services listening on host-local ports. For example, `api.DOMAIN` is routed to the M3TAL API daemon listening on `http://host.docker.internal:8080` via a dynamic configuration file such as `/docker/dynamic/api.yml`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the M3TAL Dashboard container on its internal port 8082 using Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom User Service via Traefik
To expose a custom Docker Compose service (e.g., `my-app`) via Traefik, add the necessary labels to its service definition:

```yaml
# In /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN:-localhost}`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
      - "traefik.docker.network=proxy" # Ensure this service uses the same network as Traefik
    networks:
      - proxy

networks:
  proxy:
    external: true # Link to the existing 'proxy' network
```

After placing this file in `/docker/` and running `m3tal up`, your `my-app` service will be accessible via `http://app.DOMAIN`.

## Service Management
The M3TAL API daemon runs as a systemd service, `m3tal-api.service`. You can manage its lifecycle and inspect its status using standard `systemctl` commands:

*   **Check Status**: `systemctl status m3tal-api.service`
*   **Restart Service**: `systemctl restart m3tal-api.service`
*   **View Logs**: `journalctl -u m3tal-api.service -f`

## Quick Demo
This section provides basic commands to get started with M3TAL.

*   **Start the M3TAL Dashboard**:
    The `m3tal dash up` command is specifically designed to manage the M3TAL Dashboard container. It downloads the latest compose files for the dashboard, reads your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.
    ```bash
    m3tal dash up
    ```
    Once started, if using the default `DASHBOARD_EXPOSE_MODE=local`, access the dashboard at `http://HOST_IP:8082`.

*   **Deploy All M3TAL Stacks and User-Defined Services**:
    The `m3tal up` command orchestrates and deploys all Docker Compose stacks defined by `*-compose.yml` files within the `/docker/` directory. This includes core M3TAL services like Traefik and any additional user-defined compose files you have placed there.
    ```bash
    m3tal up
    ```

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------- | :--------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                   | The public-facing HTTP port for services exposed via Traefik.                      |
| 8080 | M3TAL API daemon (Go)       | Host-local                               | The internal port the M3TAL API daemon listens on.                                 |
| 8081 | Traefik dashboard           | Host-local only                          | The internal Traefik dashboard port, accessible only from the host machine.        |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration
M3TAL uses environment variables for configuration, primarily managed through the `/etc/m3tal/.env` file. The `m3tal config wizard` command provides an interactive way to set these variables.

### Environment Variables
Key environment variables configurable for the M3TAL system include:

*   **API_PORT**: Internal port for the M3TAL API daemon (default: `5050` for container, `8080` for host).
*   **DASHBOARD_PORT**: Internal port for the M3TAL Dashboard container (default: `8082`).
*   **DASHBOARD_EXPOSE_MODE**: Controls dashboard access mode (`local` or `traefik`, default: `local`).
*   **HTTP_PORT**: Not explicitly used system-wide, may be used by user-defined services (default: `8080`).
*   **STATE_DIR**: Directory for internal state files (default: `./state`).
*   **LOG_LEVEL**: Logging verbosity (default: `info`).
*   **DASHBOARD_SECRET**: Secret key for dashboard sessions (default: `change_me_immediately`).
*   **API_TOKEN**: Token for API authentication (default: `change_me_api_token`).
*   **ADMIN_PASSWORD**: Initial administrator password for the dashboard (default: `admin_pass`).
*   **NETWORK_NAME**: Name of the Docker network M3TAL services operate on (default: `m3tal`).
*   **LOCAL_IP**: Local IP address (default: `127.0.0.1`).
*   **DOMAIN**: Base domain for Traefik routing (default: `localhost`).
*   **VPN_USER**: VPN username (default: `user`).
*   **VPN_PASSWORD**: VPN password (default: `password`).
*   **BASE_STORAGE_PATH**: Base path for data storage (default: `./data`).
*   **MEDIA_PATH**: Path for media files (default: `./data/media`).
*   **CONFIG_PATH**: Path for configuration files (default: `./data/config`).
*   **DOWNLOADS_PATH**: Path for downloads (default: `./data/downloads`).
*   **PUID**: User ID for container processes (default: `1000`).
*   **PGID**: Group ID for container processes (default: `1000`).
*   **TZ**: Timezone setting (default: `America/Denver`).
*   **TRAEFIK_WEB_PORT**: Public HTTP port for Traefik (default: `80`).
*   **TRAEFIK_WEBHTTPS_PORT**: Public HTTPS port for Traefik (default: `443`).
*   **TRAEFIK_DASHBOARD_PORT**: Internal Traefik dashboard port (default: `8080`).
*   **DEBUG_MODE**: Enable debug logging (default: `false`).
*   **METRICS_ENABLED**: Enable system metrics (default: `true`).