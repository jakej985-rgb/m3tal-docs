# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system. It describes the core components, their interdependencies, installation methods, and deployment strategies for managing Docker-based services.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

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

## Core Components
The M3TAL system comprises the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations and interactions.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker operations, interacts with the internal state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, internally listening on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name, primarily on host port `80`. It uses a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration public internet access for exposed services.

## Filesystem Contract
The following table outlines the critical filesystem paths and their respective purposes within the M3TAL system:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the API daemon.             |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                   | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.           |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside, and `/docker` is a user-facing symlink alias for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. These can be managed using `m3tal config wizard` or `m3tal config set KEY value`.
3.  Execute `m3tal up` to deploy all Docker Compose stacks, including your newly added one.

## Service Management
The M3TAL API daemon is managed as a systemd service. Its status and operations can be controlled using standard systemctl commands:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Dashboard Access
The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`) - Default
This is the default access method.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **Access Method:** The dashboard container binds directly to host port `8082`. It is accessed via `http://HOST_IP:8082` (replace `HOST_IP` with the IP address of your M3TAL host) or `http://localhost:8082`.
*   **Requirements:** No Traefik gateway is required for this mode.
*   **Behavior:** A new user performing a default installation will access the dashboard directly via port 8082, as this is the behavior of the `DASHBOARD_EXPOSE_MODE=local` setting.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE` is set to `traefik`.
*   **Access Method:** Traefik labels are applied to the dashboard container, allowing Traefik to route traffic to it. The dashboard is then accessed via `http://dash.DOMAIN` (replace `DOMAIN` with your configured domain).
*   **Requirements:** The Traefik gateway must be running (`m3tal up` will deploy it).

## Traefik Gateway
Traefik is deployed as a container via `routing-compose.yml` and functions as the M3TAL system's reverse proxy. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds host port `80` as its primary HTTP entry point and loads dynamic configuration files from `/docker/dynamic/` (which supports hot-reloading).

*   **API Daemon Routing:** Traefik routes `api.DOMAIN` (e.g., `api.example.com`) to the M3TAL API daemon listening on host-local port `8080`. This is achieved through a dynamic configuration file (e.g., `dynamic/api.yml`) that specifies a router for `Host(`api.${DOMAIN}`)` and directs it to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` (e.g., `dash.example.com`) to the M3TAL Dashboard container on its internal port `8082`.

### Exposing a Custom Service via Traefik
To expose a custom user service through Traefik, you need to add appropriate Traefik labels to its Docker Compose service definition.

Example `my-app-compose.yml` snippet:

```yaml
services:
  my-app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network
```

## Quick Demo

*   **Start the M3TAL Dashboard:**
    To specifically start or restart only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard compose files, reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration (local or Traefik mode).

*   **Deploy all M3TAL Stacks:**
    To orchestrate and deploy all Docker Compose stacks defined in the `/docker/` directory, including the M3TAL control plane components (dashboard, API, Traefik) and any user-defined compose files, use:
    ```bash
    m3tal up
    ```
    This command ensures all services are brought up according to their respective `*-compose.yml` definitions.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                               |
| 8080 | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                                          |
| 8081 | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine.                 |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration
The primary configuration for M3TAL is stored in `/etc/m3tal/.env`. This file contains environment variables that control the behavior of the M3TAL API daemon and influence Docker Compose stack deployments. The `m3tal config wizard` command provides an interactive way to manage these settings.

## Environment Variables
Key environment variables and their default values:

*   `API_PORT`: `5050`
*   `DASHBOARD_PORT`: `8082`
*   `DASHBOARD_EXPOSE_MODE`: `local`
*   `HTTP_PORT`: `8080`
*   `STATE_DIR`: `./state`
*   `LOG_LEVEL`: `info`
*   `DASHBOARD_SECRET`: `change_me_immediately`
*   `API_TOKEN`: `change_me_api_token`
*   `ADMIN_PASSWORD`: `admin_pass`
*   `NETWORK_NAME`: `m3tal`
*   `LOCAL_IP`: `127.0.0.1`
*   `DOMAIN`: `localhost`
*   `VPN_USER`: `user`
*   `VPN_PASSWORD`: `password`
*   `BASE_STORAGE_PATH`: `./data`
*   `MEDIA_PATH`: `./data/media`
*   `CONFIG_PATH`: `./data/config`
*   `DOWNLOADS_PATH`: `./data/downloads`
*   `PUID`: `1000`
*   `PGID`: `1000`
*   `TZ`: `America/Denver`
*   `TRAEFIK_WEB_PORT`: `80`
*   `TRAEFIK_WEBHTTPS_PORT`: `443`
*   `TRAEFIK_DASHBOARD_PORT`: `8080`
*   `DEBUG_MODE`: `false`
*   `METRICS_ENABLED`: `true`

## Docker Service Images
The M3TAL system utilizes the following Docker images for its core services:

*   `m3tal-dashboard`: `ghcr.io/jakej985-rgb/m3tal-godash:debug`
*   `traefik`: `traefik:latest`
*   `cloudflared`: `cloudflare/cloudflared:latest`