# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. It describes the core components, their interactions, deployment mechanisms, and management interfaces.

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

## System Architecture

The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary interface for system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port 8080. It is responsible for Docker management, interacting with the state database, and exposing core API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port 8082. It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port 80. It utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing zero-configuration internet access for services.

## Filesystem Contract

The following table outlines the critical file paths and their purposes within the M3TAL system:

| Path | Purpose |
| :----------------------- | :---------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside, and `/docker` is a user-facing symlink alias for all stack operations, providing a consistent entry point for managing services.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your new service.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** This mode uses an override (`m3tal-compose.local.yml`) to add a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access:** Direct access via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Key Point:** A new user performing a default installation will access the dashboard directly via port 8082, as this is the default behavior when `DASHBOARD_EXPOSE_MODE=local`. Traefik is not required for dashboard access in this mode.
*   **Use Case:** Ideal for LAN-only setups, initial configurations, or local testing environments.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** This mode uses an override (`m3tal-compose.traefik.yml`) to add Traefik labels to the dashboard container. Traefik then routes `dash.DOMAIN` to the dashboard on its internal port 8082.
*   **Access:** Via a domain, typically `http://dash.DOMAIN` (requires Traefik to be running via `m3tal up`).
*   **Use Case:** Suited for domain-based environments where multiple services are managed behind a single reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is configured to:
*   Bind to host port `80` as its primary HTTP entry point.
*   Discover Docker services via the `proxy` network.
*   Load dynamic configurations from `/docker/dynamic/`, supporting hot-reload for immediate updates.
*   Route `api.DOMAIN` to the M3TAL API daemon (running on host-local port `8080`) by leveraging dynamic configuration files such as `dynamic/api.yml`. This routes requests to `http://host.docker.internal:8080`.
*   Route `dash.DOMAIN` to the M3TAL Dashboard container (internal port `8082`) when `DASHBOARD_EXPOSE_MODE=traefik` is configured, using labels within `m3tal-compose.traefik.yml`.

### Example: Exposing a Custom Service via Traefik

To expose a custom user service, `my-app`, through Traefik, add the following labels to its service definition in your `my-app-compose.yml` file:

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

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`.
Common systemctl commands for service management include:

*   Check service status: `systemctl status m3tal-api`
*   Restart the service: `systemctl restart m3tal-api`
*   View live logs: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL Dashboard:

*   To start only the M3TAL Dashboard container, ensuring the necessary compose files are pulled and activated according to `DASHBOARD_EXPOSE_MODE`:
    ```bash
    m3tal dash up
    ```
    If `DASHBOARD_EXPOSE_MODE=local` (the default), you can access the dashboard at `http://HOST_IP:8082`.
*   To orchestrate and deploy all Docker Compose stacks defined in the `/docker/` directory, including the M3TAL Dashboard (if `m3tal dash up` has been run or the relevant compose files exist) and any custom user-defined compose files:
    ```bash
    m3tal up
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Environment Variables

The M3TAL system relies on environment variables defined in `/etc/m3tal/.env` for configuration. Below are some key variables:

*   **API_PORT**: Port for the M3TAL API daemon. (Default: `5050`)
*   **DASHBOARD_PORT**: Internal port for the M3TAL Dashboard. (Default: `8082`)
*   **DASHBOARD_EXPOSE_MODE**: Controls how the dashboard is exposed (`local` or `traefik`). (Default: `local`)
*   **HTTP_PORT**: Not directly used by M3TAL core, typically Traefik's `80` or other application ports. (Default: `8080`)
*   **STATE_DIR**: Directory for state files. (Default: `./state`)
*   **LOG_LEVEL**: Logging verbosity. (Default: `info`)
*   **DASHBOARD_SECRET**: Secret key for dashboard sessions. (Default: `change_me_immediately`)
*   **API_TOKEN**: API authentication token. (Default: `change_me_api_token`)
*   **ADMIN_PASSWORD**: Default password for dashboard admin. (Default: `admin_pass`)
*   **NETWORK_NAME**: Docker network name for services. (Default: `m3tal`)
*   **LOCAL_IP**: Local IP address. (Default: `127.0.0.1`)
*   **DOMAIN**: Base domain for Traefik routing. (Default: `localhost`)
*   **PUID**: User ID for container processes. (Default: `1000`)
*   **PGID**: Group ID for container processes. (Default: `1000`)
*   **TZ**: Timezone setting. (Default: `America/Denver`)
*   **BASE_STORAGE_PATH**: Base path for storage. (Default: `./data`)
*   **TRAEFIK_WEB_PORT**: Public HTTP port for Traefik. (Default: `80`)
*   **TRAEFIK_WEBHTTPS_PORT**: Public HTTPS port for Traefik. (Default: `443`)