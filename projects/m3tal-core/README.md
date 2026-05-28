# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## APT Installation

To install the M3TAL CLI and API daemon on Debian-based systems:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract

The following table details the primary directories and files utilized by the M3TAL system:

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Components

The M3TAL system consists of the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A Go binary providing a unified entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It manages Docker interactions, the state database, and exposes API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, listening internally on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services by domain name on host port `80`. It uses a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container for establishing a Cloudflare tunnel, enabling zero-configuration public internet access for exposed services.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files (e.g., `m3tal-compose.yml`, `routing-compose.yml`, and any user-defined stacks) reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a user-facing path for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode uses the `m3tal-compose.local.yml` override file, which adds a direct port binding to the dashboard container (`${DASHBOARD_PORT:-8082}:8082`). No Traefik configuration is involved or required for access.
*   **Access**: The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **New User Experience**: A new user performing a default M3TAL installation will access the dashboard directly via host port `8082` due to the `DASHBOARD_EXPOSE_MODE=local` setting being the default. This is suitable for LAN-only setups or initial testing.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode uses the `m3tal-compose.traefik.yml` override file. This file applies Traefik labels to the dashboard service, enabling Traefik to route incoming requests for `dash.DOMAIN` to the dashboard container's internal port `8082`. This requires the Traefik gateway to be running.
*   **Access**: The dashboard is accessible via `http://dash.DOMAIN`, where `DOMAIN` is configured in `/etc/m3tal/.env`. This mode is intended for domain-based setups where multiple services are managed by Traefik.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and binds host port `80` as its HTTP entry point. It loads additional dynamic configuration from the `/docker/dynamic/` directory, which supports hot-reloading.

For example, Traefik routes requests for `api.DOMAIN` to the M3TAL API daemon, which listens on the host-local port `8080`. This is achieved through a dynamic configuration file like `dynamic/api.yml`:

```yaml
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)"
      service: api
      entryPoints:
        - web

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080"
```

Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels within `m3tal-compose.traefik.yml` instruct Traefik to route `dash.DOMAIN` to the M3TAL Dashboard container, which listens internally on port `8082`.

### Exposing a Custom Service via Traefik

To expose a custom user service through Traefik, add appropriate labels to its Docker Compose service definition:

```yaml
# Example: my-app-compose.yml
version: '3.8'

services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    ports: # Note: Direct port binding not strictly necessary if only exposing via Traefik.
      - "8083:80" # Example direct port for testing, remove for Traefik-only
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Internal port of the container
      - "traefik.http.routers.myapp.entrypoints=web"
      - "traefik.docker.network=proxy" # Explicitly specify the proxy network if not default
    networks:
      - proxy

networks:
  proxy:
    external: true
```
Ensure that your service is connected to the `proxy` network, which is the external network Traefik listens on for Docker services.

## Service Management

The M3TAL API daemon operates as a systemd service, `m3tal-api.service`. Standard `systemctl` commands are used for its management:

*   **Check Status**: `systemctl status m3tal-api`
*   **Restart Service**: `systemctl restart m3tal-api`
*   **View Logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL Dashboard:

1.  **Start the Dashboard**: Execute `m3tal dash up`. This command specifically downloads the latest dashboard Docker Compose configuration and starts the `m3tal-dashboard` container with the appropriate exposure mode (defaulting to local mode if not otherwise configured).
2.  **Access the Dashboard**: If using the default `DASHBOARD_EXPOSE_MODE=local`, navigate your browser to `http://HOST_IP:8082` (replace `HOST_IP` with your server's IP address).

The `m3tal up` command, on the other hand, orchestrates and deploys all other Docker Compose stacks present in the `/docker/` directory, including the Traefik gateway, Cloudflared (if configured), and any user-defined compose files you have added.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|-----------------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.