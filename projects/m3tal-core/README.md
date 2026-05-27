# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. It describes the system's architecture, component interactions, deployment mechanisms, and essential management tasks.

## Prerequisites

**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

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

## Filesystem Contract

The M3TAL system relies on a defined filesystem structure for its operation and configuration:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created by the API daemon.                 |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files and Traefik config. |
| `/docker`                   | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |

## Components

The M3TAL system comprises the following core components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing a single entrypoint for all system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes the system's API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and uses a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling secure, zero-configuration internet access for services.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files and Traefik dynamic configuration files reside. For user convenience, `/docker` is a symlink alias to `/opt/m3tal/stack/`, serving as the primary user-facing directory for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added service.

## Dashboard Access

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting upon initial installation).
*   **Mechanism**: M3TAL uses the `m3tal-compose.local.yml` override file, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`. No Traefik configuration is involved.
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Behavior for New Users**: A new user performing a default installation will access the dashboard directly via port 8082, as `DASHBOARD_EXPOSE_MODE` defaults to `local`.
*   **Use Case**: Ideal for LAN-only setups, initial testing, or users who prefer direct port access without domain routing.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: M3TAL uses the `m3tal-compose.traefik.yml` override file, which adds specific Traefik labels to the dashboard service. These labels instruct Traefik to route `dash.${DOMAIN}` to the dashboard container's internal port `8082`. This requires the Traefik gateway to be running.
*   **Access**: `http://dash.DOMAIN` (e.g., `http://dash.localhost` or `http://dash.yourdomain.com`).
*   **Use Case**: Suitable for domain-based deployments and integrating the dashboard into a larger ecosystem managed by Traefik.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and listens on host port `80` (HTTP). It loads dynamic configuration from `/docker/dynamic/` (which symlinks to `/opt/m3tal/stack/dynamic/`), allowing for hot-reloading of routing rules.

### Dynamic Routing to Host-Local Services

Traefik can route requests to services running directly on the host machine, outside of Docker. For instance, `api.DOMAIN` is routed to the M3TAL Go API daemon, which listens on host-local port `8080`. This is achieved through a dynamic configuration file (e.g., `/docker/dynamic/api.yml`) that defines a Traefik service pointing to `http://host.docker.internal:8080`.

**Example: `api.DOMAIN` routing (`/docker/dynamic/api.yml`)**
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
Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` routes to the dashboard container via Traefik labels specified in `m3tal-compose.traefik.yml`.

### Exposing a Custom User Service via Traefik

To expose a new Docker Compose service through Traefik, add the necessary labels to its service definition:

**Example: `my-app-compose.yml`**
```yaml
version: '3.8'

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
      - proxy

networks:
  proxy:
    external: true
```
Ensure your service is connected to the `proxy` network, which is the network Traefik monitors. After placing this file in `/docker/` and running `m3tal up`, your application will be accessible at `http://app.DOMAIN`.

## Service Management

The M3TAL API daemon runs as a `systemd` service. You can manage its lifecycle using standard `systemctl` commands:

*   **Check status**: `systemctl status m3tal-api.service`
*   **Restart service**: `systemctl restart m3tal-api.service`
*   **View logs**: `journalctl -u m3tal-api.service -f`

## Quick Demo

To quickly bring up essential M3TAL components and demonstrate basic functionality:

1.  **Start the Dashboard**:
    To start only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command specifically fetches the latest dashboard compose files, reads your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`, and starts the dashboard with the appropriate Docker Compose override. By default (`DASHBOARD_EXPOSE_MODE=local`), you can access it at `http://HOST_IP:8082`.

2.  **Deploy all M3TAL stacks**:
    To orchestrate and deploy all M3TAL-managed Docker Compose stacks, including the Traefik gateway, Cloudflared, and any user-defined compose files placed in the `/docker/` directory:
    ```bash
    m3tal up
    ```
    This command provides a comprehensive deployment of the entire M3TAL Docker infrastructure.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                    | Access                                        | Description                                                          |
| :--- | :------------------------- | :-------------------------------------------- | :------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public                                        | The public-facing HTTP port for services exposed via Traefik.        |
| 8080 | M3TAL API daemon (Go)      | Host-local                                    | The internal port the M3TAL API daemon listens on.                   |
| 8081 | Traefik dashboard          | Host-local only                               | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.