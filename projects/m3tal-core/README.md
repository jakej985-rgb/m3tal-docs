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

## Core Components

The M3TAL system is comprised of several integrated components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the internal SQLite state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, internally listening on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik Gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing and automatically discovers services via Docker labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, used for establishing secure, zero-configuration internet access to services.

## Filesystem Contract

The following paths define critical locations within the M3TAL filesystem:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary system-wide configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik dynamic configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by the `m3tal dashpass` command. |

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

It is critical to note that `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside. The `/docker` directory is a user-facing symlink alias provided for convenience in all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added service.

## Dashboard Access

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)
This is the default access mode and requires no Traefik configuration.
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` is set in `/etc/m3tal/.env`.
*   **Mechanism**: The `m3tal-dashboard` container's internal port `8082` is directly bound to the host machine's port `8082` (or the port specified by `DASHBOARD_PORT`). This is achieved via the `m3tal-compose.local.yml` override file.
*   **Access**: A new user performing a default installation will access the dashboard directly via port 8082 at `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Use Case**: Ideal for LAN-only setups, initial configurations, or local development environments.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)
This mode routes dashboard access through the Traefik Gateway, requiring Traefik to be running.
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik` is set in `/etc/m3tal/.env`.
*   **Mechanism**: The `m3tal-dashboard` container is configured with Traefik labels via the `m3tal-compose.traefik.yml` override file. These labels instruct Traefik to route requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: Access is achieved via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
*   **Use Case**: Suited for domain-based setups where multiple services are managed behind a single reverse proxy.

## Traefik Gateway

Traefik is deployed as a Docker container via `routing-compose.yml`. It acts as the primary ingress point for services exposed via domain names. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik utilizes a file provider to load dynamic configuration from `/docker/dynamic/` (symlink to `/opt/m3tal/stack/dynamic/`), enabling hot-reloading of routing rules.

*   **API Daemon Routing**: Traefik routes requests for `api.DOMAIN` to the M3TAL API daemon, which listens on the host-local port `8080`. This is achieved through a dynamic configuration file, such as `dynamic/api.yml`, which defines a router and service pointing to `http://host.docker.internal:8080`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the `m3tal-dashboard` container based on labels applied to the dashboard service, targeting its internal port `8082`.

### Example: Exposing a Custom Service via Traefik
To expose a custom user service (e.g., `my-app`) via Traefik, add the necessary Traefik labels to its Docker Compose service definition:

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true" # Enable Traefik for this service
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)" # Route requests for app.DOMAIN
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Target container's internal port 80
      - "traefik.http.routers.myapp.entrypoints=web" # Use the 'web' entrypoint (port 80)
      - "traefik.docker.network=proxy" # Ensure Traefik can see this service on the 'proxy' network
    networks:
      - proxy

networks:
  proxy:
    external: true # Use the existing 'proxy' network managed by M3TAL
```

After placing `my-app-compose.yml` in `/docker/` and running `m3tal up`, `app.DOMAIN` will route to your `my-app` service.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`.

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

The M3TAL CLI provides convenience commands for quick deployment and management.

*   **Start the Dashboard only**: To specifically start or update the M3TAL Dashboard container, ensuring it runs with the correct `DASHBOARD_EXPOSE_MODE` configuration:
    ```bash
    m3tal dash up
    ```
    This command will download the necessary dashboard compose files, apply overrides based on `/etc/m3tal/.env`, and launch the dashboard container.

*   **Deploy all M3TAL stacks**: To orchestrate and deploy all Docker Compose stacks found within the `/docker/` directory (including the Dashboard, Traefik, and any user-defined compose files):
    ```bash
    m3tal up
    ```
    This command acts as a wrapper for `docker compose up -d` across all `*-compose.yml` files, bringing all defined services online.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.