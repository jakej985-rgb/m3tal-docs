# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

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

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Components
*   **CLI binary** (`/usr/bin/m3tal`): Unified Go binary installed via APT. Single entrypoint for all operations.
*   **API daemon** (`m3tal-api.service`): Go binary running as a systemd service on port 8080. Manages Docker, state DB, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): Python/Flask container running internally on port 8082. Communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): Reverse proxy container exposing services by domain name on port 80. Uses file provider for dynamic routing.
*   **Cloudflared** (`routing-compose.yml`): Optional Cloudflare tunnel container for zero-config internet access.

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, serving as the user-facing path for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure required variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all stacks.

## Dashboard Access
The M3TAL Dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

*   **Local Mode (Default):**
    *   Set `DASHBOARD_EXPOSE_MODE=local` in `/etc/m3tal/.env`. This is the default setting for a new installation.
    *   The dashboard container binds directly to host port `8082`.
    *   Access via: `http://HOST_IP:8082` or `http://localhost:8082`.
    *   No Traefik instance is required for this mode. A new user performing a default installation will access the dashboard directly via port 8082.

*   **Traefik Mode:**
    *   Set `DASHBOARD_EXPOSE_MODE=traefik` in `/etc/m3tal/.env`.
    *   The dashboard container is configured with Traefik labels, allowing Traefik to route `dash.DOMAIN` to the dashboard container on its internal port `8082`.
    *   Access via: `http://dash.DOMAIN`.
    *   Requires the Traefik gateway to be running (usually started via `m3tal up`).

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and listens on host port `80` (and optionally `443` for HTTPS). It uses a file provider to load dynamic configuration from `/docker/dynamic/` (which is `/opt/m3tal/stack/dynamic/`).

For example, Traefik routes requests for `api.DOMAIN` to the Go API daemon, which listens on the host-local port `8080`, by using the dynamic configuration file `dynamic/api.yml`. This file contains instructions to route `http://api.DOMAIN` to `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels in `m3tal-compose.traefik.yml` enable routing `dash.DOMAIN` to the dashboard container's internal port `8082`.

### Exposing a Custom User Service via Traefik
To expose a custom user service, define the necessary Traefik labels within its Docker Compose service definition:

```yaml
# Example: my-app-compose.yml
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
After placing this `my-app-compose.yml` in `/docker/` and running `m3tal up`, `my-app` would be accessible via `http://app.DOMAIN` through Traefik.

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
The M3TAL API daemon is managed as a systemd service.
*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo
*   To start only the M3TAL Dashboard container, use: `m3tal dash up`. This command specifically downloads the necessary compose files and starts the dashboard with the correct expose mode based on your `/etc/m3tal/.env` configuration.
*   To orchestrate and deploy all other stacks, including any user-defined compose files placed in the `/docker/` directory, use the command: `m3tal up`. This command will deploy all `*-compose.yml` files as independent stacks.