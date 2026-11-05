# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
To install M3TAL, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract
The following table outlines the critical file system paths used by M3TAL:

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Core Components
The M3TAL system is comprised of several key components:

*   **CLI binary** (`/usr/bin/m3tal`): A Go binary serving as the unified command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It manages Docker interactions, the internal state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container responsible for routing external requests to internal services based on domain names, typically on host port `80`. It leverages a file provider for dynamic configuration.
*   **Cloudflared** (Optional, part of `routing-compose.yml`): A Cloudflare tunnel container, providing an optional solution for secure internet access without direct port exposure.

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a symlink to `/opt/m3tal/stack/`. This means that `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside, and `/docker` is the user-facing alias for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all active Docker Compose stacks, including your newly added one.

## Dashboard Access
The M3TAL Dashboard supports two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
When `DASHBOARD_EXPOSE_MODE=local` (which is the default setting for a new installation), the dashboard is directly exposed via a port binding.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** Utilizes the `m3tal-compose.local.yml` override, adding a direct port binding (`${DASHBOARD_PORT:-8082}:8082`).
*   **Access:** The dashboard is accessible directly at `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Note:** A new user performing a default M3TAL installation will access the dashboard directly via port `8082`. No Traefik configuration is required for this mode.

### Traefik Mode
When `DASHBOARD_EXPOSE_MODE=traefik`, the dashboard is routed via the Traefik Gateway.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** Utilizes the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard container. Traefik then routes `dash.DOMAIN` to the dashboard container on its internal port `8082`.
*   **Access:** The dashboard is accessible at `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`). This mode requires Traefik to be running via `m3tal up`.

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is configured to load dynamic configuration files from `/docker/dynamic/`. This allows for routing requests to services that are not managed directly by Traefik's Docker provider (e.g., host-local services). For instance, `api.DOMAIN` is routed to the M3TAL Go API daemon, which listens on host-local port `8080`, via a dynamic configuration file (e.g., `dynamic/api.yml`) that defines a router and service pointing to `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` routes to the dashboard container when in Traefik mode.

### Exposing a Custom Service via Traefik
To expose a custom user service via Traefik, add appropriate labels to its service definition in your Docker Compose file (e.g., `my-app-compose.yml`):

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
```
In this example, `app.DOMAIN` would route to your `my-app` service. The `networks: - proxy` entry ensures your service can communicate with the Traefik container.

## Service Management
The M3TAL API daemon is managed by systemd under the service name `m3tal-api.service`.
You can manage its lifecycle using standard `systemctl` commands:

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo
This section provides quick commands to get started.

*   **Start the M3TAL Dashboard:**
    ```bash
    m3tal dash up
    ```
    This command specifically starts the M3TAL Dashboard container, downloading its required compose files and applying the correct override based on your `DASHBOARD_EXPOSE_MODE` setting.
*   **Deploy all M3TAL Stacks:**
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all Docker Compose stacks defined by `*-compose.yml` files found in the `/docker/` directory. This includes core M3TAL services like Traefik, and any user-defined compose files.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Environment Variables
The following environment variables are primarily used by the M3TAL system and can be configured in `/etc/m3tal/.env`:

*   `API_PORT`: The port the M3TAL API daemon listens on. Default: `5050`
*   `API_TOKEN`: An authentication token for API access. Default: `""`
*   `M3TAL_STATE_DB`: Path to the SQLite state database. Default: `""` (auto-managed)
*   `M3TAL_PLUGINS_DIR`: Directory for M3TAL plugins. Default: `""`