# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL system is distributed via an APT repository. Follow these steps to install the `m3tal` CLI binary:

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

The M3TAL system comprises several interdependent components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. This daemon manages Docker interactions, the internal state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, listening internally on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing and automatically discovers Docker services via labels.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration internet access for services.

## Filesystem Contract

The following paths represent critical locations within the M3TAL filesystem:

| Path                        | Purpose                                                              |
| :-------------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik dynamic configuration. |
| `/docker`                   | A symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by the `m3tal dashpass` command. |
| `/docker/dynamic/`          | Directory for Traefik dynamic configuration files. Hot-reloaded by Traefik. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files and Traefik configuration files reside. For user convenience and operational consistency, `/docker` serves as a symlink alias to `/opt/m3tal/stack/` for all stack-related operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Execute `m3tal up` to deploy all Docker Compose stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard's access method is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** This mode utilizes the `m3tal-compose.local.yml` override file, which adds a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement:** No Traefik gateway is required for this mode.
*   **Behavior:** A new user performing a default installation will access the dashboard directly via port 8082. This behavior is linked to the default `DASHBOARD_EXPOSE_MODE=local` setting.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** This mode utilizes the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the dashboard container. Traefik then routes `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN`.
*   **Requirement:** The Traefik gateway must be running (`m3tal up` will deploy it).

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and listens on host port `80` (and `443` if configured) as its primary HTTP entry point. It loads dynamic configuration files from `/docker/dynamic/`, which are hot-reloaded upon changes.

*   **API Daemon Routing:** The M3TAL API daemon, running on the host machine at `http://localhost:8080`, is exposed via Traefik. A dynamic configuration file (e.g., `dynamic/api.yml`) routes requests for `api.DOMAIN` to `http://host.docker.internal:8080`, which resolves to the host's M3TAL API daemon from within the Docker network.
*   **Dashboard Routing:** In `traefik` mode, `dash.DOMAIN` is routed to the `m3tal-dashboard` container by Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik

To expose a custom Docker Compose service via Traefik, add the necessary Traefik labels to its service definition:

```yaml
# /docker/my-app-compose.yml
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
  # Ensure your service is connected to the 'proxy' network
  # for Traefik to discover it.
  proxy:
    external: true
```

After adding or modifying a stack, run `m3tal up` to apply changes.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard systemctl commands apply:

*   **Check Status:** `systemctl status m3tal-api`
*   **Restart Service:** `systemctl restart m3tal-api`
*   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo

*   To start only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command specifically downloads the latest dashboard Compose files, applies the correct override based on `DASHBOARD_EXPOSE_MODE`, and starts the dashboard.
*   To deploy or update all configured Docker Compose stacks, including any user-defined compose files located in `/docker/`:
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all `*-compose.yml` files found in `/docker/`.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :---------------------------------------- | :----------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.