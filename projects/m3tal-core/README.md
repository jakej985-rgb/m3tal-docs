# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install the M3TAL CLI and API daemon, use the following APT commands:

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

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration internet access for services.

## Filesystem Contract

The following table details the critical directories and files within the M3TAL system:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.      |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files.            |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.           |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all M3TAL-managed Docker Compose files reside. The `/docker` path is a symbolic link alias to `/opt/m3tal/stack/`, providing a convenient user-facing location for all stack-related operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy your new stack along with all other managed stacks.

## Dashboard Access

The M3TAL Dashboard offers two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for a new installation).
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file, which adds a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik gateway is needed for this mode. A new user performing a default installation will access the dashboard directly via port 8082.
*   **Use Case**: Ideal for LAN-only setups, first-time users, and local testing environments.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file, which adds Traefik labels to the dashboard service. These labels instruct Traefik to route `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access**: Via a configured domain, e.g., `http://dash.YOUR_DOMAIN`.
*   **Requirements**: The Traefik gateway must be running and configured correctly via `m3tal up`.
*   **Use Case**: Suited for domain-based setups and environments where multiple services are exposed behind a single reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds to host port `80` (and optionally `443` for HTTPS) as its HTTP entry point. It also processes dynamic configuration files located in `/docker/dynamic/` (which is symlinked from `/opt/m3tal/stack/dynamic/`), allowing for hot-reloading of routing rules.

An example of dynamic configuration is the routing of `api.DOMAIN` to the M3TAL Go API daemon. The `dynamic/api.yml` file configures Traefik to route requests for `api.YOUR_DOMAIN` to `http://host.docker.internal:8080`, which is the host-local port where the M3TAL API daemon listens. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the dashboard container via its defined Traefik labels.

### Exposing a Custom Service via Traefik

To expose a custom user service (e.g., `my-app`) via Traefik, add appropriate labels to its service definition in your Docker Compose file (`my-app-compose.yml`):

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN}`)" # Route requests for app.YOUR_DOMAIN
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Target internal container port 80
      - "traefik.http.routers.myapp.entrypoints=web" # Use the 'web' entrypoint (port 80)
      - "traefik.docker.network=proxy" # Ensure Traefik can discover the service on the 'proxy' network
    networks:
      - proxy

networks:
  proxy:
    external: true # Use the existing 'proxy' network created by M3TAL
```
After placing this file in `/docker/`, run `m3tal up` to deploy the service and configure Traefik.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. You can control and monitor its status using standard `systemctl` commands:

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL Dashboard:

1.  Start the M3TAL Dashboard container specifically:
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard Docker Compose files, reads your `DASHBOARD_EXPOSE_MODE` setting from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.
2.  If `DASHBOARD_EXPOSE_MODE` is set to `local` (the default), access the dashboard directly via `http://HOST_IP:8082` (replace `HOST_IP` with your server's IP address or `localhost`).
3.  To deploy all other M3TAL-managed stacks and any user-defined Docker Compose files you've placed in `/docker/`, execute:
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all compose files found in `/docker/`, including the core M3TAL services and any custom user stacks.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                     | Access                                      | Description                                                                                             |
| :--- | :-------------------------- | :------------------------------------------ | :------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public                                      | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)       | Host-local                                  | The internal port the M3TAL API daemon listens on.                                                      |
| 8081 | Traefik dashboard           | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.