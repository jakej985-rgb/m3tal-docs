# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary and systemd service are installed via an APT repository.

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

The following table details the critical filesystem paths used by M3TAL:

| Path                     | Purpose                                                      |
| :----------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Automatically created and managed by the M3TAL API daemon. |
| `/opt/m3tal/stack/`      | The canonical directory containing all Docker Compose files and Traefik dynamic configuration. |
| `/docker`                | A symbolic link to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | The credential store for the M3TAL Dashboard. Managed by `m3tal dashpass`. |

## Core Components

The M3TAL system is comprised of several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations and user interactions.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker containers, interacts with the SQLite state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container, internally listening on port `8082`. It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration internet access for exposed services.

## Configuration

M3TAL's primary configuration is managed through environment variables stored in `/etc/m3tal/.env`. These variables dictate various system behaviors, network settings, and component access. The `m3tal config wizard` command provides an interactive interface for managing these settings.

## Environment Variables

The following environment variables are commonly used within the M3TAL system, typically configured in `/etc/m3tal/.env`:

| Key                  | Default Value           | Description                                                 |
| :------------------- | :---------------------- | :---------------------------------------------------------- |
| `API_PORT`           | `5050`                  | Internal port for a component (legacy, use HTTP_PORT).      |
| `DASHBOARD_PORT`     | `8082`                  | Internal port the M3TAL Dashboard container listens on.     |
| `DASHBOARD_EXPOSE_MODE`| `local`                 | Defines how the Dashboard is exposed (`local` or `traefik`).|
| `HTTP_PORT`          | `8080`                  | The port the M3TAL API daemon listens on.                   |
| `STATE_DIR`          | `./state`               | Directory for persistent state files (e.g., SQLite DB).     |
| `LOG_LEVEL`          | `info`                  | Logging verbosity (`debug`, `info`, `warn`, `error`).       |
| `DASHBOARD_SECRET`   | `change_me_immediately` | Secret key for Dashboard session management. **Critical to change.** |
| `API_TOKEN`          | `change_me_api_token`   | API token for authenticating with the M3TAL API. **Critical to change.** |
| `ADMIN_PASSWORD`     | `admin_pass`            | Default password for the M3TAL Dashboard admin user. **Critical to change.** |
| `NETWORK_NAME`       | `m3tal`                 | Name of the Docker network used by M3TAL services.          |
| `LOCAL_IP`           | `127.0.0.1`             | The local IP address, primarily for internal communication. |
| `DOMAIN`             | `localhost`             | Base domain for Traefik routing (e.g., `dash.DOMAIN`).      |
| `VPN_USER`           | `user`                  | User for VPN configurations (if applicable).                |
| `VPN_PASSWORD`       | `password`              | Password for VPN configurations (if applicable).            |
| `BASE_STORAGE_PATH`  | `./data`                | Base directory for persistent data volumes.                 |
| `MEDIA_PATH`         | `./data/media`          | Path for media storage.                                     |
| `CONFIG_PATH`        | `./data/config`         | Path for configuration files.                               |
| `DOWNLOADS_PATH`     | `./data/downloads`      | Path for downloaded content.                                |
| `PUID`               | `1000`                  | User ID for container processes.                            |
| `PGID`               | `1000`                  | Group ID for container processes.                           |
| `TZ`                 | `America/Denver`        | Timezone for containers.                                    |
| `TRAEFIK_WEB_PORT`   | `80`                    | Public HTTP port for Traefik.                               |
| `TRAEFIK_WEBHTTPS_PORT`| `443`                   | Public HTTPS port for Traefik.                              |
| `TRAEFIK_DASHBOARD_PORT`| `8080`                  | Internal port for Traefik's own dashboard.                  |
| `DEBUG_MODE`         | `false`                 | Enables debug logging and features.                         |
| `METRICS_ENABLED`    | `true`                  | Enables system metrics collection.                          |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a symbolic link pointing to `/opt/m3tal/stack/`. This means that `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside, and `/docker/` is the user-facing alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all Docker Compose stacks, including your newly added service.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Behavior:** This mode adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container. It uses the `m3tal-compose.local.yml` override file.
*   **Access:** The M3TAL Dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik gateway is required for this mode.
*   **Audience:** This is the default setting upon a new installation, providing immediate direct access via port `8082` for first-time users, LAN-only setups, or local testing.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Behavior:** This mode applies Traefik labels (via the `m3tal-compose.traefik.yml` override file) to the dashboard container, instructing Traefik to route traffic.
*   **Access:** The M3TAL Dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN` is `localhost`).
*   **Requirements:** Traefik must be running and correctly configured (typically via `m3tal up`) for this mode to function.
*   **Audience:** Suitable for domain-based deployments and environments where multiple services are managed behind a single reverse proxy.

## Traefik Gateway

Traefik functions as the central reverse proxy for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik dynamically loads its configuration, allowing for hot-reloads when changes are detected in its configuration directory (`/docker/dynamic/` on the host, which maps to `/etc/traefik/dynamic` inside the Traefik container).

*   **API Daemon Routing:** Traefik routes requests for `api.DOMAIN` to the M3TAL API daemon, which listens on host-local port `8080`. This is achieved through a dynamic configuration file (e.g., `/docker/dynamic/api.yml`) which maps `api.DOMAIN` to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the dashboard container, which internally listens on port `8082`, using labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik

To expose a custom user-defined Docker Compose service through Traefik, add appropriate labels to its service definition:

```yaml
# Example: my-app-compose.yml in /docker/
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
      - proxy # Ensure your service is on the 'proxy' network

networks:
  proxy:
    external: true # This network must be external and shared with Traefik
```

In this example:
*   `traefik.enable=true` activates Traefik discovery for this service.
*   `traefik.http.routers.myapp.rule=Host(\`app.DOMAIN\`)` defines the domain rule for routing.
*   `traefik.http.services.myapp.loadbalancer.server.port=80` specifies the internal port of the `my-app` container that Traefik should route to.
*   `traefik.http.routers.myapp.entrypoints=web` specifies that this router should listen on the `web` entrypoint (typically port 80).
*   The `proxy` network ensures the service is discoverable by Traefik.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard `systemctl` commands can be used for its operation:

*   **Check Status:** `systemctl status m3tal-api.service`
*   **Restart Service:** `systemctl restart m3tal-api.service`
*   **View Logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo

The `m3tal` CLI provides specific commands for managing components:

*   To start *only* the M3TAL Dashboard container, you can use:
    ```bash
    m3tal dash up
    ```
    This command specifically manages the dashboard container: it downloads the latest compose files, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate Docker Compose override.

*   To orchestrate and deploy *all* Docker Compose stacks (including the dashboard, Traefik, and any user-defined compose files in `/docker/`), use:
    ```bash
    m3tal up
    ```
    This command ensures all services defined by `*-compose.yml` files in the `/docker/` directory are started and managed.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                                                                                                                                                                                                                               |
| 8080 | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                                                                                                                                                                                                                                          |
| 8081 | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine (e.g., `http://localhost:8081`).                                                                                                                                                                                |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. In `local` mode (default), it's directly exposed on `HOST_IP:8082`. In `traefik` mode, it's routed via Traefik as `dash.DOMAIN`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.