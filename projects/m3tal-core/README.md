# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

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

## Overview of Components

The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary entry point for all system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It is responsible for managing Docker interactions, the state database, and exposing API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, internally listening on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container that facilitates zero-configuration internet access for services.

## Filesystem Contract

The following table details the critical directories and files within the M3TAL filesystem:

| Path                     | Purpose                                                              |
| :----------------------- | :------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the API daemon.               |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.             |

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                    | Access                                  | Description                                                                                     |
| :--- | :------------------------- | :-------------------------------------- | :---------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public                                  | The public-facing HTTP port for services exposed via Traefik.                                   |
| 8080 | M3TAL API daemon (Go)      | Host-local                              | The internal port the M3TAL API daemon listens on.                                              |
| 8081 | Traefik dashboard          | Host-local only                         | The internal Traefik dashboard port, accessible only from the host machine.                     |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a symlink to `/opt/m3tal/stack/`. `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside, and `/docker` is the user-facing symlink alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are set in `/etc/m3tal/.env` (e.g., using `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all defined stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

When `DASHBOARD_EXPOSE_MODE=local`, the dashboard container is configured with a direct port binding. This is the default behavior for a new user performing a default installation.

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: Uses the `m3tal-compose.local.yml` override file to add a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik gateway is required for this mode. It operates out-of-the-box, suitable for LAN-only setups or initial deployments.

### Traefik Mode

When `DASHBOARD_EXPOSE_MODE=traefik`, the dashboard container is exposed via the Traefik reverse proxy.

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: Uses the `m3tal-compose.traefik.yml` override file to add Traefik labels, allowing Traefik to route traffic from `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (assuming `DOMAIN` is correctly configured in `/etc/m3tal/.env`).
*   **Requirements**: The Traefik gateway must be running (typically started via `m3tal up`) for this access method to function. This mode is suitable for domain-based setups.

## Traefik Gateway

Traefik is deployed as a Docker container via the `routing-compose.yml` file. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik listens on host port `80` (and `443` if HTTPS is configured) as its primary entry point. It also loads dynamic configuration files from `/docker/dynamic/` (which is a symlink to `/opt/m3tal/stack/dynamic/`), allowing for hot-reload of routing rules.

*   **Internal API Routing**: Traefik routes requests for `api.DOMAIN` to the M3TAL Go API daemon, which listens on host-local port `8080`. This is achieved through a dynamic configuration file (e.g., `/docker/dynamic/api.yml`) that defines the rule and points to `http://host.docker.internal:8080`.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.DOMAIN` to the `m3tal-dashboard` container, which internally listens on port `8082`. This routing is managed by Traefik labels within the `m3tal-compose.traefik.yml` override.

### Exposing a Custom Service via Traefik

To expose a custom Docker Compose service through Traefik, add the appropriate labels to your service definition:

```yaml
# Example: /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true" # Enable Traefik for this service
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)" # Define the host rule
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Map to the container's internal port
      - "traefik.http.routers.myapp.entrypoints=web" # Use the 'web' entrypoint (port 80)
      - "traefik.docker.network=proxy" # Ensure service is on the Traefik proxy network
    networks:
      - proxy # Connect to the external proxy network

networks:
  proxy:
    external: true
```

After adding or modifying such a file in `/docker/`, run `m3tal up` to apply the changes.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard systemctl commands are used for its operation:

*   **Check Status**: `systemctl status m3tal-api`
*   **Restart Service**: `systemctl restart m3tal-api`
*   **View Logs**: `journalctl -u m3tal-api -f` (for real-time logs)

## Quick Demo

To quickly get the M3TAL dashboard running and explore the system:

1.  **Start the M3TAL Dashboard**: Execute `m3tal dash up`. This command specifically downloads the latest dashboard Docker Compose files and starts the `m3tal-dashboard` container with the appropriate configuration based on `DASHBOARD_EXPOSE_MODE` in `/etc/m3tal/.env`.
2.  **Access the Dashboard**:
    *   If `DASHBOARD_EXPOSE_MODE=local` (the default), access the dashboard at `http://HOST_IP:8082`.
    *   If `DASHBOARD_EXPOSE_MODE=traefik` and Traefik is running (via `m3tal up`), access it at `http://dash.DOMAIN`.
3.  **Deploy All Stacks**: To orchestrate and deploy all other stacks defined by `*-compose.yml` files within the `/docker/` directory (including any user-defined compose files and the `routing-compose.yml` for Traefik and Cloudflared), run:
    ```bash
    m3tal up
    ```
    This command ensures all configured M3TAL services are running.

## Environment Variables

The core behavior of M3TAL can be configured via environment variables primarily located in `/etc/m3tal/.env`. These can be managed using `m3tal config wizard` or `m3tal config set <KEY> <value>`.

| Key                     | Default Value          | Description                                                    |
| :---------------------- | :--------------------- | :------------------------------------------------------------- |
| `API_PORT`              | `5050`                 | Internal port for the M3TAL API daemon (deprecated, use HTTP_PORT). |
| `DASHBOARD_PORT`        | `8082`                 | Internal port for the M3TAL Dashboard container.               |
| `DASHBOARD_EXPOSE_MODE` | `local`                | Controls how the dashboard is exposed (`local` or `traefik`). |
| `HTTP_PORT`             | `8080`                 | Port for the M3TAL API daemon.                                 |
| `STATE_DIR`             | `./state`              | Directory for the SQLite state database.                       |
| `LOG_LEVEL`             | `info`                 | Logging verbosity (`debug`, `info`, `warn`, `error`).          |
| `DASHBOARD_SECRET`      | `change_me_immediately`| Secret key for dashboard session security.                     |
| `API_TOKEN`             | `change_me_api_token`  | API token for authentication with the M3TAL API.               |
| `ADMIN_PASSWORD`        | `admin_pass`           | Default admin password for the dashboard.                      |
| `NETWORK_NAME`          | `m3tal`                | Default Docker network name used by M3TAL.                     |
| `LOCAL_IP`              | `127.0.0.1`            | Local IP address for internal communications.                  |
| `DOMAIN`                | `localhost`            | Base domain for Traefik routing.                               |
| `VPN_USER`              | `user`                 | Username for VPN services (if applicable).                     |
| `VPN_PASSWORD`          | `password`             | Password for VPN services (if applicable).                     |
| `BASE_STORAGE_PATH`     | `./data`               | Base directory for all data storage.                           |
| `MEDIA_PATH`            | `./data/media`         | Path for media files.                                          |
| `CONFIG_PATH`           | `./data/config`        | Path for configuration files.                                  |
| `DOWNLOADS_PATH`        | `./data/downloads`     | Path for downloaded content.                                   |
| `PUID`                  | `1000`                 | User ID for container processes.                               |
| `PGID`                  | `1000`                 | Group ID for container processes.                              |
| `TZ`                    | `America/Denver`       | Timezone for containers.                                       |
| `TRAEFIK_WEB_PORT`      | `80`                   | Public HTTP port for Traefik.                                  |
| `TRAEFIK_WEBHTTPS_PORT` | `443`                  | Public HTTPS port for Traefik.                                 |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                 | Internal port for Traefik's own dashboard.                     |
| `DEBUG_MODE`            | `false`                | Enables debug mode.                                            |
| `METRICS_ENABLED`       | `true`                 | Enables system metrics collection.                             |