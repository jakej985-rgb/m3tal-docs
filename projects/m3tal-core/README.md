# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. It describes the core components, deployment mechanisms, configuration, and service management practices.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install the M3TAL CLI and API daemon via APT, execute the following commands:

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

## Filesystem Contract

The following table outlines the critical directories and files within the M3TAL system:

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard` and `m3tal config set`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker` | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | M3TAL Dashboard credential store. Managed by `m3tal dashpass`. |

## Components

The M3TAL system comprises the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations, including configuration, service management, and Docker orchestration.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database (`/var/lib/m3tal/state.db`), and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`. Its exposure method (direct port or via Traefik) depends on the `DASHBOARD_EXPOSE_MODE` setting.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` (and `443` for HTTPS, if configured). Traefik automatically discovers Docker services via their labels and loads dynamic routing configurations from files.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, managed alongside Traefik, used for establishing secure, zero-config internet access to M3TAL services.

## Core Configuration

The primary configuration file for M3TAL is `/etc/m3tal/.env`. This file contains environment variables that control various aspects of the system, including network settings, dashboard access mode, and file paths.

*   To interactively configure the system, use `m3tal config wizard`.
*   To set a specific key-value pair, use `m3tal config set KEY value`.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard systemctl commands apply:

*   Check service status: `systemctl status m3tal-api`
*   Restart the service: `systemctl restart m3tal-api`
*   View live logs: `journalctl -u m3tal-api -f`

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The canonical source of truth for all stack files and Traefik configuration is `/opt/m3tal/stack/`. The `/docker` directory is a symlink to `/opt/m3tal/stack/`, serving as the user-facing alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Traefik Gateway

Traefik functions as the primary reverse proxy for M3TAL, deployed as a Docker container via `routing-compose.yml`. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds to host port `80` (and `443` if HTTPS is configured) as its public HTTP entry point. It loads dynamic routing configurations from files placed in `/docker/dynamic/` (which corresponds to `/etc/traefik/dynamic` inside the Traefik container), allowing for hot-reloading of routes without restarting Traefik.

For example, `api.DOMAIN` is routed to the M3TAL Go API daemon (listening on host-local port `8080`) through a dynamic configuration file (e.g., `dynamic/api.yml`). This file specifies a router to capture requests for `api.DOMAIN` and forward them to `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` is routed to the M3TAL Dashboard container when `DASHBOARD_EXPOSE_MODE=traefik` is set, by utilizing Traefik labels attached to the dashboard service.

### Exposing a Custom Service via Traefik

To expose a hypothetical `my-app` service from a user-defined Docker Compose stack via Traefik, ensure its Docker Compose service definition includes the following labels and is connected to the `proxy` network:

```yaml
# In your my-app-compose.yml file located in /docker/
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
    external: true # Ensure this network is declared external to connect to Traefik's network
```

After adding or modifying such a service, run `m3tal up` to deploy the changes. Traefik will automatically detect the new labels and begin routing traffic for `app.YOUR_DOMAIN` to your `my-app` container.

## Dashboard Access

The M3TAL Dashboard provides a web-based interface for managing the system. It offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`. The internal port for the dashboard container is `8082`, which can be customized via the `DASHBOARD_PORT` environment variable.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for a new installation).
*   **Mechanism**: The dashboard container exposes port `8082` directly on the host machine using a port binding (e.g., `${DASHBOARD_PORT:-8082}:8082`). This mode does not require Traefik.
*   **Access**: A new user performing a default installation will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Use Case**: Ideal for LAN-only setups, initial configurations, or local testing, where direct IP-based access is sufficient.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism**: The dashboard container is configured with Traefik labels that enable Traefik to discover and route traffic to it. Traefik handles domain-based routing to the dashboard's internal port `8082`. This mode requires Traefik to be running.
*   **Access**: Access is via a domain, typically `http://dash.YOUR_DOMAIN`.
*   **Use Case**: Suited for deployments requiring domain-based access, often alongside other services exposed via Traefik, or when HTTPS is desired (configured through Traefik).

## Quick Demo

To quickly get the M3TAL Dashboard up and running:

1.  **Start the Dashboard**: Run `m3tal dash up`. This command specifically manages the M3TAL Dashboard container, ensuring the latest compose files are used and the correct `DASHBOARD_EXPOSE_MODE` is applied.
2.  **Access the Dashboard**:
    *   **Default (Local Mode)**: If `DASHBOARD_EXPOSE_MODE` is set to `local` (which is the default after installation), access the dashboard directly via your host machine's IP address on port 8082: `http://HOST_IP:8082`.
    *   **Traefik Mode**: If `DASHBOARD_EXPOSE_MODE` is set to `traefik`, ensure Traefik is running (e.g., via `m3tal up`) and access the dashboard at `http://dash.YOUR_DOMAIN`.
3.  **Deploy All Stacks**: To deploy all other M3TAL core stacks (like Traefik, Cloudflared, and any user-defined compose files in `/docker/`), run `m3tal up`. This command orchestrates and deploys all `*-compose.yml` files found within the `/docker/` directory.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.