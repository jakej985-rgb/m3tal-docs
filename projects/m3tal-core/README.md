# M3TAL System Documentation

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

## Filesystem Contract

The following paths define the M3TAL system's filesystem structure:

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker` | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | M3TAL Dashboard credential store. Managed by `m3tal dashpass`. |

## Quick Demo

This section outlines immediate operational commands:

-   To specifically start only the M3TAL Dashboard container:
    ```bash
    m3tal dash up
    ```
    This command downloads the necessary Docker Compose files for the dashboard, reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard container with the appropriate configuration.
-   To orchestrate and deploy all Docker Compose stacks defined in the `/docker/` directory (including any user-defined compose files):
    ```bash
    m3tal up
    ```

## Core Components

The M3TAL system is comprised of the following key components:

-   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary entrypoint for all M3TAL operations.
-   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the SQLite state database, and exposes the M3TAL API routes.
-   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container running internally on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
-   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services by domain name on host port `80`. It utilizes Docker labels for service discovery and a file provider for dynamic routing configurations.
-   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for providing zero-configuration external access to services, deployed as part of the `routing` stack.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Use standard systemctl commands for operation:

-   Check service status: `systemctl status m3tal-api`
-   Restart the API daemon: `systemctl restart m3tal-api`
-   View service logs: `journalctl -u m3tal-api -f`

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside, and `/docker` serves as a user-facing symlink alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env`. These can be managed using `m3tal config wizard` or `m3tal config set KEY value`.
3.  Run `m3tal up` to start all active stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

-   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
-   **Behavior:** This mode adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container via the `m3tal-compose.local.yml` override.
-   **Access:** The dashboard is directly accessible at `http://HOST_IP:8082` or `http://localhost:8082`.
-   **Requirements:** No Traefik gateway is required for this mode. A new user performing a default M3TAL installation will access the dashboard directly via port 8082.
-   **Use Case:** Ideal for LAN-only setups, initial configurations, or local testing environments.

### Traefik Mode

-   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
-   **Behavior:** This mode enables Traefik labels on the dashboard service via the `m3tal-compose.traefik.yml` override, allowing Traefik to route traffic to the dashboard.
-   **Access:** The dashboard is accessible via a domain name, specifically `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
-   **Requirements:** The Traefik gateway must be running (`m3tal up` will deploy it if `routing-compose.yml` is present).
-   **Use Case:** Suited for domain-based environments or when integrating multiple services behind a single reverse proxy.

## Traefik Gateway

Traefik functions as the system's reverse proxy. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik's configuration involves both static settings (from `traefik.yml` within its container) and dynamic configurations (from files like `/docker/dynamic/api.yml`). These dynamic configuration files are used to route requests to services that might not be running as Docker containers discoverable by Traefik's Docker provider, such as the M3TAL API daemon. For instance, `dynamic/api.yml` routes `api.DOMAIN` to the M3TAL API daemon, which listens on the host-local port `8080`, specifically `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` routes to the dashboard container on its internal port `8082` via its Docker labels.

### Exposing a Custom Service via Traefik

To expose a custom Docker Compose service through Traefik, add the necessary labels to its service definition:

```yaml
# Example: my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure the service is on the same network as Traefik
```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.