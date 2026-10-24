# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install the M3TAL CLI and API daemon via APT, execute the following commands:

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

The M3TAL system comprises the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It manages Docker interactions, the state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services by domain name on port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for establishing zero-configuration internet access.

## Filesystem Contract

The following table details the critical directories and files within the M3TAL system:

| Path                        | Purpose                                                                       |
| :-------------------------- | :---------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.                 |
| `/var/lib/m3tal/state.db`   | SQLite state database. Automatically created and managed by the API daemon.   |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik config.    |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.                      |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all M3TAL-managed Docker Compose files and Traefik configuration reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a user-facing and convenient path for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` to manage these variables.
3.  Execute `m3tal up` to initiate or update all Docker Compose stacks, including your newly added one.

## M3TAL API Daemon (Go)

The M3TAL API daemon runs as a systemd service, `m3tal-api.service`. It is responsible for core system logic and Docker interaction.

To manage the API daemon:

*   Check its status: `systemctl status m3tal-api`
*   Restart the service: `systemctl restart m3tal-api`
*   View live logs: `journalctl -u m3tal-api -f`

## Dashboard Access

The M3TAL Dashboard provides a graphical interface for system management. Its access method is controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: The dashboard container exposes port `8082` directly on the host machine via a port binding (`${DASHBOARD_PORT:-8082}:8082`). This mode uses the `m3tal-compose.local.yml` override file.
*   **Access**: `http://HOST_IP:8082` or `http://localhost:8082`
*   **Behavior for New Users**: A new user performing a default installation will access the dashboard directly via port `8082`, as `DASHBOARD_EXPOSE_MODE=local` is the default setting.
*   **Requirements**: No Traefik gateway is required for this mode.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: The `m3tal-compose.traefik.yml` override file applies Traefik labels to the dashboard container. Traefik then routes incoming requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN=localhost`)
*   **Requirements**: The Traefik gateway must be running (`m3tal up` will deploy it).

## Traefik Gateway

The Traefik gateway is deployed as a Docker container via `routing-compose.yml`. It acts as the primary entry point for domain-based service access. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is configured to:

*   Bind host port `80` as its HTTP entry point (`web`).
*   Discover services via the Docker provider on the `proxy` network, but only those explicitly exposing Traefik labels (`exposedByDefault: false`).
*   Load dynamic configuration from `/docker/dynamic/` (which is a symlink to `/opt/m3tal/stack/dynamic/`), enabling hot-reloading of routing rules.

### Dynamic Routing Examples

*   **M3TAL API Daemon**: Traefik routes `api.DOMAIN` to the M3TAL Go API daemon, which listens on host-local port `8080`. This is achieved using a file provider configuration like `dynamic/api.yml`, which translates the domain request into a request to `http://host.docker.internal:8080`.
*   **M3TAL Dashboard**: When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the dashboard container, targeting its internal port `8082` as defined by its Traefik labels.

### Exposing a Custom User Service via Traefik

To expose a new custom service (e.g., `my-app`) via Traefik, you would add appropriate labels to its service definition within your Docker Compose file (`my-app-compose.yml` in `/docker/`):

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-custom-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network for Traefik discovery

networks:
  proxy:
    external: true
```

After placing this file in `/docker/` and running `m3tal up`, your `my-app` service would be accessible via `http://app.DOMAIN`.

## Quick Demo

*   **Start the M3TAL Dashboard**:
    To specifically start or update only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard Docker Compose files, reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and deploys the dashboard with the appropriate override.

*   **Deploy All M3TAL Stacks and User Services**:
    To orchestrate and deploy all Docker Compose stacks defined in `/docker/`, including the core M3TAL components and any user-defined compose files (e.g., `routing-compose.yml`, `m3tal-compose.yml`, `my-app-compose.yml`), use:
    ```bash
    m3tal up
    ```
    This command will bring up all services according to their respective Docker Compose configurations.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                         | The public-facing HTTP port for services exposed via Traefik.                                                |
| 8080 | M3TAL API daemon (Go)       | Host-local                                     | The internal port the M3TAL API daemon listens on.                                                           |
| 8081 | Traefik dashboard           | Host-local only                                | The internal Traefik dashboard port, accessible only from the host machine.                                  |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.