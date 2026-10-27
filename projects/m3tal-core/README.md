# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

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

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## System Components

The M3TAL system comprises several key components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing a single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It is responsible for managing Docker interactions, the state database, and exposing API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name, primarily on host port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, used for establishing secure, zero-configuration internet access to services.

## Filesystem Contract

The following table details the critical M3TAL filesystem paths and their purposes:

| Path | Purpose |
| :----------------------- | :-------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains all Docker Compose files and Traefik dynamic configuration. |
| `/docker` | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files reside. For user convenience, `/docker` is a symbolic link to `/opt/m3tal/stack/`, serving as the user-facing alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`** (default setting).
*   The dashboard container is exposed via a direct port binding.
*   Access via: `http://HOST_IP:8082` or `http://localhost:8082`.
*   A new user performing a default installation will access the dashboard directly via port `8082`.
*   This mode does not require Traefik to be running.
*   Suitable for local network access or initial setup.

### Traefik Mode

*   **`DASHBOARD_EXPOSE_MODE=traefik`**.
*   The dashboard container is exposed via Traefik, utilizing Traefik labels for routing.
*   Access via: `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   Requires the Traefik gateway (`routing-compose.yml` stack) to be running via `m3tal up`.
*   Ideal for domain-based access and integration with other services behind Traefik.

## Traefik Gateway

Traefik acts as the reverse proxy for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik also handles dynamic configurations for routing requests to services not directly managed by Docker Compose labels, such as the M3TAL API daemon. For instance, Traefik loads dynamic configuration files from `/docker/dynamic/` (which is `/opt/m3tal/stack/dynamic/`). The file `dynamic/api.yml` routes requests for `api.DOMAIN` to the Go API daemon listening on the host-local port `8080` via `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` routes to the M3TAL dashboard container when `DASHBOARD_EXPOSE_MODE=traefik`.

### Exposing a Custom User Service via Traefik

To expose your own Docker Compose service through Traefik, add the necessary Traefik labels to its service definition.

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

After adding this file to `/docker/` and running `m3tal up`, Traefik will route `http://app.DOMAIN` to your `my-app` service.

## Service Management

The M3TAL API daemon is managed as a systemd service. Use standard `systemctl` commands for interaction:

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo

To quickly get started with M3TAL:

*   To start only the M3TAL Dashboard container (useful for initial setup or troubleshooting):
    ```bash
    m3tal dash up
    ```
    This command specifically manages the dashboard container, downloading the necessary Docker Compose files and applying the appropriate override based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.
*   To orchestrate and deploy all M3TAL core stacks (dashboard, Traefik, Cloudflared) and any other user-defined Docker Compose files placed in `/docker/`:
    ```bash
    m3tal up
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.