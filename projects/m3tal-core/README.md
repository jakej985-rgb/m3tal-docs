# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

## Installation

To install the M3TAL CLI binary and API daemon, use the following APT commands:

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

The M3TAL system is composed of several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container designed to expose Docker services by domain name, typically on host port `80`. It utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration public internet access for services.

## Filesystem Contract

The following table details the key file system paths and their purposes within the M3TAL system:

| Path                     | Purpose                                                          |
| :----------------------- | :--------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon.           |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.         |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside. The `/docker` directory is a user-facing symlink alias pointing to `/opt/m3tal/stack/` for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all stacks, including your newly added one.

## Service Management

The M3TAL API daemon is managed by systemd as `m3tal-api.service`. Standard systemctl commands can be used for service control and status:

*   **Check service status:** `systemctl status m3tal-api`
*   **Restart the service:** `systemctl restart m3tal-api`
*   **View service logs:** `journalctl -u m3tal-api -f`

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for new installations).
*   **Mechanism:** M3TAL applies an override (`m3tal-compose.local.yml`) that creates a direct port binding on the host.
*   **Access:** Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** Traefik is not required for this mode.
*   **Usage:** This mode is ideal for first-time users, local testing, or LAN-only deployments, as a new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** M3TAL applies an override (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard container, allowing Traefik to route traffic.
*   **Access:** Via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
*   **Requirements:** The Traefik gateway (`routing-compose.yml`) must be running via `m3tal up` for this mode to function.
*   **Usage:** Suited for domain-based setups and environments where multiple services are managed behind a central reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml`, which binds host port `80` (and `443` if HTTPS is configured) as its public HTTP entry point. It dynamically loads additional routing configurations from `/docker/dynamic/` (using a file provider with hot-reload).

*   **API Daemon Routing:** Traefik uses dynamic configuration files (e.g., `/docker/dynamic/api.yml`) to route requests from `api.DOMAIN` to the M3TAL Go API daemon, which listens on the host-local port `8080` (routed via `http://host.docker.internal:8080` from within the Docker network).
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the dashboard container via labels specified in `m3tal-compose.traefik.yml`, targeting the dashboard's internal port `8082`.

### Exposing a Custom Service via Traefik

To expose a custom user service via Traefik, add appropriate labels to its service definition in your Docker Compose file:

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

Ensure your custom stack uses the `proxy` network, which is the default external network Traefik listens on for service discovery.

## Quick Demo

The M3TAL CLI provides specific commands for managing components:

*   To start only the M3TAL dashboard container specifically (useful for initial setup or troubleshooting):
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard compose files, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate override.

*   To orchestrate and deploy all M3TAL stacks, including Traefik, Cloudflared, and any user-defined Docker Compose files placed in `/docker/`:
    ```bash
    m3tal up
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                     | Access                                      | Description                                                                                             |
| :--- | :-------------------------- | :------------------------------------------ | :------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public                                      | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)       | Host-local                                  | The internal port the M3TAL API daemon listens on.                                                      |
| 8081 | Traefik dashboard           | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.