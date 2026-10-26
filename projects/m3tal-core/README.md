# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. M3TAL is designed for orchestrating and managing containerized services.

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

After installation, it is recommended to run `m3tal config wizard` to set up initial system configurations.

## Core Components

The M3TAL system comprises the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary that serves as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration internet access for exposed services.

## Filesystem Contract

The following table details the critical directories and files within the M3TAL filesystem:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | The canonical directory containing all Docker Compose stack files and Traefik configuration files. |
| `/docker`                   | A symbolic link to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside. The `/docker` directory serves as a user-facing symlink alias for all stack operations, simplifying access and management.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard offers two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting after a new installation.)
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Accessible directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik instance is required.
*   **Usage**: Ideal for initial setup, LAN-only environments, and local testing. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container. Traefik then routes `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access**: Accessible via `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN` is `localhost`).
*   **Requirements**: Traefik must be running via `m3tal up`.
*   **Usage**: Suitable for domain-based setups where multiple services are exposed through Traefik.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and listens on host port `80` as its HTTP entry point. It loads dynamic configuration files from `/docker/dynamic/` (which is `/etc/traefik/dynamic` inside the container), allowing for hot-reload of routing rules.

### Key Traefik Routing Mechanisms:

*   **API Daemon Routing**: Traefik routes requests for `api.DOMAIN` to the M3TAL Go API daemon, which listens on the host-local port `8080`. This is configured via a dynamic configuration file, such as `/docker/dynamic/api.yml`, which translates `api.DOMAIN` to `http://host.docker.internal:8080`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the `m3tal-dashboard` container by interpreting the labels present in `m3tal-compose.traefik.yml`, targeting the dashboard's internal port `8082`.

### Exposing a Custom Service via Traefik

To expose a custom user service, ensure it is part of the `proxy` network (which Traefik uses for discovery) and define appropriate Traefik labels in its Docker Compose service definition.

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
      - proxy
networks:
  proxy:
    external: true
```

After placing this file in `/docker/` and running `m3tal up`, Traefik will expose your `my-app` service at `http://app.DOMAIN`.

## Service Management

The M3TAL API daemon is managed as a systemd service. Use standard `systemctl` commands for interaction:

*   Check service status: `systemctl status m3tal-api`
*   Restart the service: `systemctl restart m3tal-api`
*   View live logs: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get the M3TAL Dashboard running and explore the system:

1.  **Start the Dashboard**:
    To start *only* the dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command specifically downloads the latest dashboard compose files and starts the dashboard with the appropriate override based on `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`.
    If you're using the default `DASHBOARD_EXPOSE_MODE=local`, you can then access the dashboard at `http://HOST_IP:8082`.

2.  **Deploy all Stacks**:
    To deploy all other configured M3TAL stacks, including any custom user-defined compose files placed in `/docker/`, run:
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all `*-compose.yml` files found in the `/docker/` directory. This includes core M3TAL components like Traefik (`routing-compose.yml`) and potentially the dashboard if `m3tal dash up` hasn't been run or if its state needs to be synchronized with a full system deployment.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.