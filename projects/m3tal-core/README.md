# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install the M3TAL CLI and API daemon, execute the following commands:

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

The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A Go binary providing a unified entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It manages Docker interactions, the state database, and exposes core API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy Docker container that exposes services by domain name on host port `80`. It uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, utilized for establishing zero-configuration internet access.

## Filesystem Contract

The following table details critical directories and files within the M3TAL filesystem:

| Path                        | Purpose                                                      |
| :-------------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary system-wide configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | The SQLite state database for the API daemon. Auto-created.  |
| `/opt/m3tal/stack/`         | The canonical directory for all M3TAL Docker Compose stack files and Traefik dynamic configuration. |
| `/docker`                   | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | The credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a user-facing symlink alias pointing to `/opt/m3tal/stack/`, which is the canonical source of truth directory where all M3TAL stack files reside.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all currently defined stacks.

## Dashboard Access

The M3TAL Dashboard provides a graphical interface and can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting upon a new installation).
*   **Mechanism**: This mode uses an override (`m3tal-compose.local.yml`) to add a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the host machine. Traefik is not required for access in this mode.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Behavior**: A new user performing a default M3TAL installation will access the dashboard directly via port 8082.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode uses an override (`m3tal-compose.traefik.yml`) to apply specific Traefik labels to the dashboard container. Traefik (if running via `m3tal up`) then routes `dash.DOMAIN` to the dashboard container on its internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN`. This requires the Traefik gateway to be deployed and operational.

## Traefik Gateway

Traefik is deployed as a Docker container via `routing-compose.yml`. It acts as the primary reverse proxy for M3TAL services.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

### Dynamic Configuration and Routing

Traefik loads additional dynamic configuration from the `/docker/dynamic/` directory (a file provider with hot-reload). This mechanism is used to route requests to services not directly managed by Docker labels (e.g., host-local daemons):

*   **M3TAL API Daemon**: The `api.DOMAIN` route is defined in a dynamic configuration file (e.g., `dynamic/api.yml`) to route requests to the M3TAL Go API daemon, which listens on the host-local port `8080` (e.g., `http://host.docker.internal:8080`).
*   **M3TAL Dashboard**: When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` routes to the dashboard container via Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom User Service via Traefik

To expose a custom Docker Compose service via Traefik, add appropriate labels to its service definition. Ensure your service is part of the `proxy` network.

Example `my-app-compose.yml`:

```yaml
services:
  my-app:
    image: nginx:alpine
    container_name: my-custom-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.routers.myapp.entrypoints=web"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

## Service Management

The M3TAL API daemon runs as a systemd service (`m3tal-api.service`). Use standard `systemctl` commands for management:

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly start and access the M3TAL Dashboard:

1.  To start only the M3TAL Dashboard container (and its dependencies like the `proxy` network):
    ```bash
    m3tal dash up
    ```
2.  If `DASHBOARD_EXPOSE_MODE` is set to `local` (the default), access the dashboard via `http://HOST_IP:8082` in your web browser.
3.  To deploy *all* M3TAL-managed Docker Compose stacks, including the dashboard (if not already running) and any user-defined compose files in `/docker/`:
    ```bash
    m3tal up
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                         | The public-facing HTTP port for services exposed via Traefik.                                                    |
| 8080 | M3TAL API daemon (Go)       | Host-local                                     | The internal port the M3TAL API daemon listens on.                                                               |
| 8081 | Traefik dashboard           | Host-local only                                | The internal Traefik dashboard port, accessible only from the host machine.                                      |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.