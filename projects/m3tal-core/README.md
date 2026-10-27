# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. M3TAL is a Docker-centric orchestration and management platform, comprising a Go CLI, a Go API daemon, and a Python/Flask dashboard, designed to deploy and manage containerized services.

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

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Components

The M3TAL system consists of the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the main entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port 8080. It manages Docker interactions, the internal state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container that runs internally on port 8082. It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port 80. It utilizes a file provider for dynamic routing configurations and Docker labels for service discovery.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for establishing secure, zero-configuration internet access to services.

## Filesystem Contract

The following paths are critical to the M3TAL system's operation and configuration:

| Path                     | Purpose                                                              |
| :----------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`      | The canonical directory for all Docker Compose stack files and Traefik dynamic configuration. |
| `/docker`                | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`     | The credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all M3TAL's internal Docker Compose stack files and associated configurations reside. The `/docker` directory is a user-facing symlink alias to `/opt/m3tal/stack/`, intended as the primary location for users to interact with and manage all stack files.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Execute `m3tal up` to deploy all Docker Compose stacks, including your newly added one.

## Traefik Gateway

Traefik functions as the primary ingress controller for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik also utilizes dynamic configuration files, such as `dynamic/api.yml` located within `/docker/dynamic/`. These files are essential for routing requests to services listening on host-local ports. For example, `api.DOMAIN` is routed to the M3TAL Go API daemon on host-local port `8080` via `http://host.docker.internal:8080` using a dynamic configuration. Similarly, `dash.DOMAIN` routes to the M3TAL Dashboard container when `DASHBOARD_EXPOSE_MODE` is set to `traefik`.

### Exposing a Custom Service via Traefik

To expose a custom user service, define appropriate Traefik labels within your Docker Compose service definition:

```yaml
# Example: /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is connected to the 'proxy' network
    # ... other service configurations ...

networks:
  proxy:
    external: true
```

After adding or modifying your Docker Compose file, run `m3tal up` to apply the changes.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting after a new installation).
*   **Mechanism**: The dashboard container exposes its internal port `8082` directly on the host machine. This is achieved via a Docker Compose override (`m3tal-compose.local.yml`) that adds a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement**: No Traefik gateway is required for this mode.
*   **Behavior for new users**: A new user performing a default M3TAL installation will access the dashboard directly via host port `8082`.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism**: The dashboard container is configured with Traefik labels (`m3tal-compose.traefik.yml` override), allowing Traefik to discover and route traffic to it. Traefik routes requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (requires the Traefik gateway to be running).
*   **Requirement**: Traefik must be deployed and operational through `m3tal up` for this mode to function.

## Quick Demo

To quickly start the M3TAL Dashboard container:

*   Execute `m3tal dash up`. This command specifically downloads the latest dashboard Docker Compose files and starts the dashboard container with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

To deploy all configured Docker Compose stacks, including any custom user-defined compose files located in `/docker/`:

*   Execute `m3tal up`. This command orchestrates and deploys all services defined across all `*-compose.yml` files in the `/docker/` directory.

## Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service. You can manage its lifecycle using standard `systemctl` commands:

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`