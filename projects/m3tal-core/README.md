# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

## Installation

### APT Installation
To install the M3TAL CLI and API daemon via APT:

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
*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container designed to expose services via domain names on host port `80`. It utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, facilitating zero-configuration internet access for exposed services.

## Filesystem Contract
The following table outlines the critical filesystem paths and their purposes within the M3TAL system:

| Path | Purpose |
| :------------------------- | :----------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env` | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database, automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/` | The canonical directory for all Docker Compose stack files and Traefik dynamic configuration. |
| `/docker` | A symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | The credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Core Configuration
The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file contains environment variables that control the behavior of various M3TAL components. It can be managed interactively using the `m3tal config wizard` command or individual variables can be set using `m3tal config set KEY value`.

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth where all Docker Compose stack files and associated configurations reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a convenient user-facing path for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env` (e.g., using `m3tal config wizard` or `m3tal config set KEY value`).
3.  Execute `m3tal up` to deploy your new stack along with all other managed services.

## Dashboard Access
The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode uses an override (`m3tal-compose.local.yml`) to add a direct port binding, exposing the dashboard container's internal port `8082` directly on the host.
*   **Access Method**: The dashboard is accessible directly at `http://HOST_IP:8082` (or `http://localhost:8082`).
*   **Requirement**: No Traefik gateway is required for this mode.
*   **Behavior for New Users**: A new user performing a default M3TAL installation will access the dashboard directly via port 8082, as `DASHBOARD_EXPOSE_MODE=local` is the default setting.
*   **Use Cases**: Ideal for LAN-only setups, first-time users, and local development or testing environments.

### Traefik Mode
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode utilizes an override (`m3tal-compose.traefik.yml`) to add Traefik labels to the dashboard service. These labels instruct Traefik to route requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access Method**: The dashboard is accessible via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`). This requires the Traefik gateway to be running (`m3tal up` will deploy it).
*   **Use Cases**: Suited for domain-based environments where multiple services are managed behind a reverse proxy.

## Traefik Gateway
Traefik serves as the ingress controller for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via the `routing-compose.yml` stack, binding host port `80` (and `443` if configured) as its HTTP entry point. It dynamically loads configuration from files located in `/docker/dynamic/`, allowing for hot-reloading of routing rules.

### Dynamic Routing for Host-Local Services
Traefik uses dynamic configuration files, such as `/docker/dynamic/api.yml`, to route requests to services that are not Docker containers but listen on host-local ports. For instance, `api.DOMAIN` is routed to the M3TAL Go API daemon, which listens on host-local port `8080`, via `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` routes to the dashboard container's internal port `8082` when `DASHBOARD_EXPOSE_MODE=traefik` is set, enabled by Traefik labels in `m3tal-compose.traefik.yml`.

### Example: Exposing a Custom User Service via Traefik
To expose a custom Docker Compose service, add the necessary Traefik labels to its service definition. This example demonstrates how to expose a hypothetical `my-app` service:

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true" # Enable Traefik for this service
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)" # Route requests for app.DOMAIN
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Target port within the container
      - "traefik.http.routers.myapp.entrypoints=web" # Use the 'web' entrypoint (port 80)
    networks:
      - proxy # Ensure the service is on the 'proxy' network

networks:
  proxy:
    external: true # Use the existing 'proxy' network managed by M3TAL
```
After placing `my-app-compose.yml` in `/docker/` and running `m3tal up`, `app.DOMAIN` will route to your `my-app` service.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`.
Standard `systemctl` commands can be used for its management:
*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo
This section provides quick commands to get started with key M3TAL functionalities.

*   **Start the Dashboard**:
    To specifically start or restart the M3TAL Dashboard container (and ensure it's configured based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`), use:
    ```bash
    m3tal dash up
    ```
    If `DASHBOARD_EXPOSE_MODE=local` (the default), the dashboard will be accessible directly via `http://HOST_IP:8082`. If `DASHBOARD_EXPOSE_MODE=traefik`, it will be accessible via `http://dash.DOMAIN` (assuming Traefik is running).

*   **Deploy All Stacks**:
    To orchestrate and deploy all Docker Compose stacks defined by `*-compose.yml` files within the `/docker/` directory (including any user-defined compose files and core M3TAL services like Traefik), use:
    ```bash
    m3tal up
    ```
    This command ensures all configured services are running according to their respective Docker Compose definitions.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.