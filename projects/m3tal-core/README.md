# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

---

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary and API daemon are installed via a dedicated APT repository.

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

The M3TAL system is comprised of several interacting components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing a single entry point for all M3TAL operations and user interactions.
*   **API daemon** (`m3tal-api.service`): A Go binary that runs as a systemd service, listening on host-local port `8080`. It is responsible for managing Docker interactions, maintaining the SQLite state database, and serving M3TAL's API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container. It listens internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container that facilitates zero-configuration internet access through Cloudflare tunnels.

## Filesystem Contract

The following table details critical directories and files within the M3TAL filesystem:

| Path                        | Purpose                                                                    |
| :-------------------------- | :------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file for M3TAL. Managed by `m3tal config wizard`.    |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | The canonical directory containing all Docker Compose stack files and Traefik dynamic configuration. |
| `/docker`                   | A symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                   |

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :---------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`. This means that `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside, and `/docker` is the user-facing symlink alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Behavior**: This mode enables a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`, exposing the dashboard directly on the host machine. This is the default setting for a new installation.
*   **Access**: A new user performing a default installation will access the dashboard directly via port 8082 at `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik configuration is required.
*   **Use Case**: Ideal for LAN-only setups, first-time users, and local testing.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Behavior**: This mode configures the dashboard container with Traefik labels, allowing Traefik to route traffic based on a domain name. Requires the Traefik gateway to be running.
*   **Access**: The dashboard is accessible via `http://dash.DOMAIN`, where `DOMAIN` is configured in `/etc/m3tal/.env`.
*   **Use Case**: Suited for domain-based setups and environments where multiple services are managed behind a Traefik reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and binds host port `80` (and optionally `443` for HTTPS) as its HTTP entry point. It dynamically loads configuration files from `/docker/dynamic/` (symlinked from `/opt/m3tal/stack/dynamic/`), enabling hot-reloading of routing rules.

For example, Traefik uses a dynamic configuration file such as `dynamic/api.yml` to route `api.DOMAIN` to the Go API daemon listening on the host-local port `8080` via `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels are added to the dashboard container to route `dash.DOMAIN` to the dashboard container's internal port `8082`.

### Exposing a Custom Service via Traefik

To expose a custom user service via Traefik, add appropriate labels to its service definition in your Docker Compose file:

```yaml
# /docker/my-app-compose.yml
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

After placing this file in `/docker/` and ensuring `DOMAIN` is set in your `.env`, run `m3tal up` to deploy the service and make it accessible via `http://app.DOMAIN`.

## Service Management

The M3TAL API daemon runs as a systemd service, `m3tal-api.service`. You can manage its lifecycle using standard `systemctl` commands:

*   **Check Status**: `systemctl status m3tal-api.service`
*   **Restart Service**: `systemctl restart m3tal-api.service`
*   **View Logs**: `journalctl -u m3tal-api.service -f`

## Quick Demo

To quickly get started with M3TAL:

1.  **Start the M3TAL Dashboard**: Run `m3tal dash up`. This command specifically downloads and starts the `m3tal-dashboard` container with the appropriate configuration based on your `DASHBOARD_EXPOSE_MODE` setting.
2.  **Access the Dashboard**:
    *   If `DASHBOARD_EXPOSE_MODE=local` (the default), access the dashboard at `http://HOST_IP:8082`.
    *   If `DASHBOARD_EXPOSE_MODE=traefik`, ensure Traefik is running (via `m3tal up`) and access at `http://dash.DOMAIN`.
3.  **Deploy All Stacks**: To orchestrate and deploy all other stacks defined by `*-compose.yml` files in the `/docker/` directory, including any user-defined compose files, execute `m3tal up`. This will start components like Traefik (if `routing-compose.yml` is present) and any other services you've configured.