# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
The M3TAL CLI binary and API daemon are installed via an APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## M3TAL System Components
The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary interface for system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, a state database, and exposes API routes for system control.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy Docker container that exposes services by domain name on host port `80`. It uses a file provider for dynamic configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing zero-configuration internet access for exposed services.

## Filesystem Contract
The following table outlines the critical filesystem paths and their purposes within the M3TAL system:

| Path | Purpose |
| :------------------------- | :------------------------------------------------------ |
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker` | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth where all Docker Compose stack files (e.g., `m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration reside. The `/docker` directory is a user-facing symlink alias to `/opt/m3tal/stack/` for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`.
3.  Execute `m3tal up` to deploy or update all Docker Compose stacks, including your new one.

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and listens on host port `80` (HTTP entry point). It loads dynamic configuration from `/docker/dynamic/` (which is `/opt/m3tal/stack/dynamic/` through the symlink).

*   Requests to `api.DOMAIN` are routed to the M3TAL API daemon (Go) listening on host-local port `8080`. This is configured via a dynamic configuration file, for instance, `dynamic/api.yml`, which translates the domain request to `http://host.docker.internal:8080`.
*   Requests to `dash.DOMAIN` are routed to the `m3tal-dashboard` container, specifically to its internal port `8082`, when `DASHBOARD_EXPOSE_MODE=traefik` is configured and Traefik labels are applied to the dashboard service.

### Exposing a Custom Service via Traefik
To expose a custom user service (e.g., an Nginx instance) through Traefik, add the necessary Traefik labels to its service definition within your Docker Compose file (e.g., `my-app-compose.yml`):

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
      - proxy # Crucial: Services exposed via Traefik must be on the 'proxy' network
networks:
  proxy:
    external: true # Assumes the 'proxy' network is external and created by M3TAL
```

## Dashboard Access
The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### 1. Local Mode (Default)
When `DASHBOARD_EXPOSE_MODE=local`, the dashboard container binds its internal port `8082` directly to the host machine's `8082` port.
*   **Access:** `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Behavior:** A new user performing a default installation will access the dashboard directly via port `8082` because `DASHBOARD_EXPOSE_MODE=local` is the default setting, enabling direct port binding.
*   **Requirements:** No Traefik gateway is required for this mode.
*   **Use Case:** Ideal for LAN-only setups, initial configurations, and local development or testing.

### 2. Traefik Mode
When `DASHBOARD_EXPOSE_MODE=traefik`, the dashboard container is configured with Traefik labels, allowing the Traefik gateway to route traffic to it.
*   **Access:** `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN` is set to `localhost`).
*   **Requirements:** Traefik must be running (`m3tal up` will deploy it).
*   **Behavior:** Traefik uses the labels to route the `dash.DOMAIN` request to the dashboard container's internal port `8082`.
*   **Use Case:** Suited for domain-based environments where multiple services are exposed through a single reverse proxy.

## Service Management
The M3TAL API daemon (`m3tal-api.service`) is managed as a systemd service.
*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo
To quickly start and access the M3TAL Dashboard:
1.  Run `m3tal dash up`. This command specifically manages the dashboard container, downloading its latest Docker Compose files and starting it with the appropriate configuration based on `DASHBOARD_EXPOSE_MODE`.
2.  By default (`DASHBOARD_EXPOSE_MODE=local`), you can access the dashboard directly via `http://HOST_IP:8082`.
3.  The `m3tal up` command orchestrates and deploys all other Docker Compose stacks found in the `/docker/` directory, including the `routing-compose.yml` (Traefik) and any user-defined compose files. This command should be used for full system deployment.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration
M3TAL uses environment variables for configuration, primarily managed through `/etc/m3tal/.env`. The `m3tal config wizard` and `m3tal config set KEY value` commands are used to interact with this file.

Key environment variables include:

*   **API_PORT**: The host-local port for the M3TAL API daemon (default: `5050`).
*   **DASHBOARD_PORT**: The internal port the dashboard container listens on (default: `8082`).
*   **DASHBOARD_EXPOSE_MODE**: Controls how the dashboard is exposed (`local` or `traefik`, default: `local`).
*   **DOMAIN**: The base domain for services exposed via Traefik (default: `localhost`).
*   **STATE_DIR**: Directory for the SQLite state database (default: `./state`).
*   **BASE_STORAGE_PATH**: Base path for all M3TAL storage volumes (default: `./data`).
*   **CONFIG_PATH**: Path for configuration files (default: `./data/config`).
*   **USERS_FILE**: Path to the dashboard user credentials file (default: `/docker/state/config/users.json`).
*   **DASHBOARD_SECRET**: Secret key for dashboard session management.
*   **API_TOKEN**: Token for API authentication.
*   **ADMIN_PASSWORD**: Default administrator password for the dashboard.
*   **PUID/PGID**: User and group IDs for container processes to ensure proper file permissions (default: `1000`).
*   **TZ**: Timezone setting (default: `America/Denver`).
*   **TRAEFIK_WEB_PORT**: Public HTTP port for Traefik (default: `80`).
*   **TRAEFIK_WEBHTTPS_PORT**: Public HTTPS port for Traefik (default: `443`).