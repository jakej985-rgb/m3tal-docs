# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. M3TAL is designed to manage and orchestrate containerized applications using Docker Engine and Docker Compose V2. It consists of a Go-based CLI, an API daemon, a Python/Flask dashboard, and a Traefik reverse proxy for service exposure.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI and API daemon are distributed via an APT repository. Follow these steps to install M3TAL:

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

The M3TAL system is comprised of the following key components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary entry point for all system operations and management tasks.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, typically on host-local port `8080`. It is responsible for managing Docker operations, interacting with the state database, and handling API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application containerized and running internally on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy Docker container that exposes M3TAL services and user-defined services by domain name, typically on host port `80`. It uses a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, managed alongside Traefik, to provide secure, zero-configuration internet access to services without requiring open inbound firewall ports.

## Filesystem Contract

The following paths define critical locations within the M3TAL filesystem:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created by the API daemon.      |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configurations. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.             |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` is the canonical source of truth where all Docker Compose stack files reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a user-friendly and consistent path for all stack management operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard provides two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode uses an override (`m3tal-compose.local.yml`) to add a direct port binding, exposing the dashboard container's internal port `8082` to the host's `8082` port.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement**: No Traefik instance is required for this mode.
*   **Default Behavior**: A new user performing a default installation will access the dashboard directly via port `8082`, as this is the behavior linked to the default `DASHBOARD_EXPOSE_MODE=local` setting.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode employs an override (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard container's service definition. Traefik then routes incoming requests for `dash.DOMAIN` to the dashboard container on its internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (assuming Traefik is running and configured, typically through `m3tal up`).
*   **Requirement**: Requires the Traefik gateway to be running and properly configured to handle domain-based routing.

## Traefik Gateway

Traefik serves as the primary reverse proxy for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik's configuration is split into static configuration (via `traefik.yml`) and dynamic configuration (via files in `/docker/dynamic/`).

*   **Dynamic Configuration**: M3TAL utilizes dynamic configuration files (e.g., `dynamic/api.yml` located in `/docker/dynamic/`) to define routes for services that may not be directly exposed as Docker containers but listen on host-local ports. For instance, `dynamic/api.yml` routes `api.DOMAIN` to the M3TAL Go API daemon, which listens on host-local port `8080`, by routing traffic to `http://host.docker.internal:8080`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the M3TAL dashboard container, which listens internally on port `8082`.

### Example: Exposing a Custom Service via Traefik

To expose a custom user service (e.g., `my-app`) via Traefik, add the necessary Traefik labels to its service definition in your Docker Compose file (`my-app-compose.yml`):

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
      - proxy

networks:
  proxy:
    external: true
```
After placing this in `/docker/my-app-compose.yml` and running `m3tal up`, Traefik will automatically configure a route for `app.DOMAIN` to your `my-app` container.

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage its state using standard `systemctl` commands:

*   **Check Status**: `systemctl status m3tal-api.service`
*   **Restart Service**: `systemctl restart m3tal-api.service`
*   **View Logs**: `journalctl -u m3tal-api.service -f`

## Quick Demo

To quickly get the M3TAL Dashboard running:

1.  Start the M3TAL Dashboard container specifically:
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard Docker Compose files, reads your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.
2.  Access the dashboard:
    *   If `DASHBOARD_EXPOSE_MODE=local` (default), access at `http://HOST_IP:8082`.
    *   If `DASHBOARD_EXPOSE_MODE=traefik`, access at `http://dash.DOMAIN` (ensure Traefik is running via `m3tal up`).

The `m3tal up` command orchestrates and deploys all Docker Compose stacks found within the `/docker/` directory, including any user-defined compose files in addition to core M3TAL components like Traefik and Cloudflared (if configured).

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.