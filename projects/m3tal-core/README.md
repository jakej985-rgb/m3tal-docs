# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
To install the M3TAL system, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

After installation, it is recommended to run `m3tal config wizard` to configure initial settings.

## Filesystem Contract
The following table details the critical paths within the M3TAL filesystem:

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Core Components

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations, including configuration, service management, and Docker orchestration.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. This daemon manages Docker interactions, maintains the SQLite state database (`/var/lib/m3tal/state.db`), and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080` for backend operations.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configurations and discovers Docker services via labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare Tunnel container that provides secure, zero-configuration internet access to services without opening public firewall ports.

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` serves as the canonical source of truth where all stack files (e.g., `m3tal-compose.yml`, `routing-compose.yml`, and any user-defined compose files) reside. The `/docker` directory is a symlink to `/opt/m3tal/stack/`, acting as the user-facing alias for all stack operations, allowing users to interact with compose files directly within this convenient path.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are configured in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` to manage these.
3.  Execute `m3tal up` to deploy or update all defined Docker Compose stacks, including your new stack.

## Dashboard Access
The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
When `DASHBOARD_EXPOSE_MODE=local` (the default setting for a new installation), the dashboard is directly accessible via a port binding.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** An override file (`m3tal-compose.local.yml`) adds a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`, to the `m3tal-dashboard` container.
*   **Access:** A new user performing a default installation will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik configuration is required for this mode.
*   **Use Case:** Ideal for LAN-only setups, initial testing, or first-time users who require immediate access without domain configuration.

```yaml
# m3tal-compose.local.yml snippet for direct port binding
services:
  m3tal-dashboard:
    ports:
      - "${DASHBOARD_PORT:-8082}:8082"
```

### Traefik Mode
When `DASHBOARD_EXPOSE_MODE=traefik`, access to the dashboard is routed through the Traefik Gateway using domain-based routing.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** An override file (`m3tal-compose.traefik.yml`) applies Traefik labels to the `m3tal-dashboard` service. These labels instruct Traefik to route `dash.${DOMAIN}` to the dashboard container, which listens internally on port `8082`.
*   **Access:** `http://dash.DOMAIN` (requires Traefik to be running via `m3tal up`).
*   **Use Case:** Suited for environments with domain-based access, where multiple services are exposed through a single reverse proxy.

```yaml
# m3tal-compose.traefik.yml snippet for Traefik routing
services:
  m3tal-dashboard:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`dash.${DOMAIN:-localhost}`)"
      - "traefik.http.routers.dashboard.entrypoints=web"
      - "traefik.http.services.dashboard.loadbalancer.server.port=8082"
      - "traefik.docker.network=proxy"
```

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml`. It binds host port `80` (and `443` if HTTPS is configured) as its public HTTP entry point. Dynamic configurations are loaded from files within `/docker/dynamic/`, allowing for hot-reload of routing rules.

*   **API Daemon Routing:** Traefik uses a dynamic configuration file (e.g., `dynamic/api.yml`) to route requests for `api.DOMAIN` to the M3TAL API daemon. Since the API daemon runs directly on the host machine (not within a Docker container accessible by Traefik's internal network), the routing rule directs traffic to `http://host.docker.internal:8080`, where `host.docker.internal` resolves to the Docker host's IP address.
*   **Dashboard Routing:** In `traefik` mode (`DASHBOARD_EXPOSE_MODE=traefik`), Traefik routes `dash.DOMAIN` to the `m3tal-dashboard` container by interpreting the labels set in `m3tal-compose.traefik.yml`, targeting the container's internal port `8082`.

### Example: Exposing a Custom User Service via Traefik
To expose a custom Docker Compose service (e.g., `my-app`) via Traefik, add the necessary Traefik labels to its service definition:

```yaml
# my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN}`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy

networks:
  proxy:
    external: true # Assumes 'proxy' network is defined and external
```

In this example, Traefik will:
1.  **`traefik.enable=true`**: Enable Traefik for this service.
2.  **`traefik.http.routers.myapp.rule=Host(`app.${DOMAIN}`)`**: Create a router named `myapp` that matches requests for `app.YOUR_DOMAIN`.
3.  **`traefik.http.services.myapp.loadbalancer.server.port=80`**: Define the backend service `myapp` to forward traffic to port `80` of the `my-app` container.
4.  **`traefik.http.routers.myapp.entrypoints=web`**: Attach this router to the `web` entrypoint (which typically listens on host port 80).
5.  **`networks: - proxy`**: Ensure the service is connected to the same `proxy` network that Traefik uses for discovery.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. You can interact with it using standard systemctl commands:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo
This section demonstrates basic M3TAL Docker orchestration commands:

*   **Start the Dashboard only:** To specifically bring up or restart the `m3tal-dashboard` container with its current `DASHBOARD_EXPOSE_MODE` setting, use:
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard compose files, reads `/etc/m3tal/.env` for the `DASHBOARD_EXPOSE_MODE`, and starts the dashboard with the appropriate override.

*   **Deploy all stacks:** To orchestrate and deploy all Docker Compose stacks defined by `*-compose.yml` files in the `/docker/` directory (including `m3tal-dashboard`, `routing`, and any user-defined compose files), use:
    ```bash
    m3tal up
    ```
    This command will ensure all services are running according to their definitions.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.