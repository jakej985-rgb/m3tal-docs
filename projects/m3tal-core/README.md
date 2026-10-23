# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

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

## Filesystem Contract

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Components

*   **CLI binary** (`/usr/bin/m3tal`): Unified Go binary acting as the single entrypoint for all operations.
*   **API daemon** (`m3tal-api.service`): Go binary running as a systemd service on host-local port `8080`. It manages Docker interactions, the state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container exposing services by domain name on host port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for zero-configuration internet access to services.

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a symlink to `/opt/m3tal/stack/`. While `/opt/m3tal/stack/` is the canonical source of truth for all stack files, `/docker/` is the user-facing alias for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your new one.

## Dashboard Access
The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`) - Default
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` is the default setting for new installations.
*   **Mechanism:** This mode utilizes a direct port binding, exposing the dashboard container's internal port `8082` directly on the host machine. It leverages the `m3tal-compose.local.yml` override file to achieve this.
*   **Access:** The dashboard is accessible directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik gateway is required for this mode.
*   **Use Case:** This is the default behavior for a new user performing a default installation, providing immediate access without complex networking configuration. Ideal for LAN-only setups or initial testing.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)
*   **Configuration:** Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
*   **Mechanism:** This mode configures the dashboard container with Traefik labels via the `m3tal-compose.traefik.yml` override. Traefik then discovers these labels and routes traffic based on the configured domain.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   **Requirements:** The Traefik gateway (part of `routing-compose.yml`) must be running via `m3tal up` for this mode to function.
*   **Use Case:** Suited for domain-based environments where multiple services are managed behind a central reverse proxy.

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via the `routing-compose.yml` stack. It binds host port `80` (and `443` if configured) as its primary HTTP entry point and dynamically loads configuration from the `/docker/dynamic/` directory (which is a symlink to `/opt/m3tal/stack/dynamic/`).

*   **API Daemon Routing:** Traefik routes requests to `api.DOMAIN` to the M3TAL API daemon. This is achieved through a dynamic configuration file, such as `/docker/dynamic/api.yml`, which routes to the API daemon listening on the host-local port `8080` via `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.DOMAIN` to the `m3tal-dashboard` container, which listens internally on port `8082`.

### Exposing a Custom User Service via Traefik
To expose a custom Docker Compose service through Traefik, add the necessary Traefik labels to its service definition.
For example, in a hypothetical `my-app-compose.yml` file placed in `/docker/`:

```yaml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
      - "traefik.docker.network=proxy" # Ensure it's on the 'proxy' network
    networks:
      - proxy

networks:
  proxy:
    external: true # Essential for Traefik to find services
```
After defining these labels and ensuring the service is on the `proxy` network, run `m3tal up` to deploy the stack and allow Traefik to discover and route traffic to `app.DOMAIN`.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`.
*   To check the status: `systemctl status m3tal-api.service`
*   To restart the API daemon: `systemctl restart m3tal-api.service`
*   To view logs: `journalctl -u m3tal-api.service -f`

## Quick Demo
*   To start only the M3TAL Dashboard container (and automatically pull its latest images and apply configuration overrides based on `DASHBOARD_EXPOSE_MODE`):
    ```bash
    m3tal dash up
    ```
*   To orchestrate and deploy all Docker Compose stacks defined by `*-compose.yml` files within the `/docker/` directory (including the routing stack, dashboard, and any user-defined compose files):
    ```bash
    m3tal up
    ```

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.