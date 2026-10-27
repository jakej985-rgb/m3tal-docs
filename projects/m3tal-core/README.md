# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

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
The M3TAL system comprises the following core components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary that serves as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service on host-local port `8080`. It is responsible for managing Docker interactions, maintaining the SQLite state database, and handling API routes.
*   **M3TAL Dashboard container** (`m3tal-dashboard`): A Python/Flask container that runs internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, used for establishing secure, zero-configuration internet access to services.

## Filesystem Contract
The following table details the critical filesystem paths used by the M3TAL system:

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Service Management
The M3TAL API daemon is managed as a systemd service, `m3tal-api.service`. Standard `systemctl` commands can be used for its operation:

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth for all M3TAL-managed Docker Compose stack files. The `/docker` directory is a symlink to `/opt/m3tal/stack/`, providing a user-facing alias for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all stacks, including your newly added one.

## Dashboard Access
The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### 1. Local Mode (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` (This is the default setting upon installation).
*   **Mechanism:** Uses the `m3tal-compose.local.yml` override file, which adds a direct port binding to the dashboard container (`${DASHBOARD_PORT:-8082}:8082`). No Traefik configuration is required for this mode.
*   **Access:** Access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Behavior:** A new user performing a default installation will access the dashboard directly via port 8082, as this is the behavior linked to the default setting `DASHBOARD_EXPOSE_MODE=local`.
*   **Use Case:** Ideal for LAN-only setups, initial configurations, and local testing environments.

### 2. Traefik Mode
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism:** Uses the `m3tal-compose.traefik.yml` override file, which applies Traefik labels to the dashboard service. These labels instruct Traefik to route requests for `dash.${DOMAIN}` to the dashboard container on its internal port `8082`. This mode requires Traefik to be actively running as part of the M3TAL deployment (`m3tal up`).
*   **Access:** Access the dashboard via `http://dash.DOMAIN`.
*   **Use Case:** Suitable for domain-based deployments where services are exposed through a central reverse proxy.

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a Docker container via `routing-compose.yml`. It binds host port `80` (and `443` if configured) as the HTTP entry point. It utilizes its Docker provider to automatically detect services with Traefik labels and also loads dynamic configuration from files located in `/docker/dynamic/`, allowing for hot-reloading of routing rules.

For example:
*   Dynamic configuration files, such as `/docker/dynamic/api.yml`, are used to route requests to services not managed directly by Traefik's Docker provider. For instance, `api.DOMAIN` is routed to the M3TAL Go API daemon, which listens on the host-local port `8080`, via `http://host.docker.internal:8080`.
*   When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the M3TAL Dashboard container based on the Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik
To expose a custom user service through Traefik, add the necessary Traefik labels to its service definition in your Docker Compose file. Ensure your service is on the `proxy` network, which Traefik is configured to monitor.

Example snippet for a hypothetical `my-app-compose.yml`:
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
After placing this file in `/docker/` and running `m3tal up`, Traefik will automatically configure routing for `app.DOMAIN` to your `my-app` service.

## Quick Demo
To quickly get the M3TAL Dashboard running:
*   To start *only* the M3TAL Dashboard container, use `m3tal dash up`. This command will handle downloading the necessary Docker Compose files and starting the dashboard based on your `DASHBOARD_EXPOSE_MODE` setting.
*   To deploy all M3TAL-managed stacks, including the Traefik gateway and any user-defined Docker Compose files placed in the `/docker/` directory, use the `m3tal up` command. This orchestrates and starts all declared services.