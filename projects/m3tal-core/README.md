# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system. It describes the core components, deployment mechanisms, access methods, and configuration guidelines necessary for installation, operation, and maintenance.

## Prerequisites

**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

## Installation

To install the M3TAL system, follow these steps to add the APT repository and install the `m3tal` package:

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

## System Architecture

M3TAL is composed of several interoperating components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. This daemon is responsible for managing Docker interactions, persistent state in the SQLite database, and providing API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running as a Docker container, internally listening on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container for establishing Cloudflare tunnels, enabling secure, zero-configuration internet access to services.

### Filesystem Contract

The following table details the critical directories and files within the M3TAL filesystem:

| Path | Purpose |
| :----------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env` | The primary configuration file, containing environment variables. Managed by the `m3tal config wizard` command. |
| `/var/lib/m3tal/state.db` | The SQLite database for storing M3TAL's operational state. Automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/` | The canonical directory containing all Docker Compose stack files and Traefik configuration files. |
| `/docker` | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | The credential store for the M3TAL Dashboard. Managed by the `m3tal dashpass` command. |

## Core Concepts

### Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files reside. The `/docker` directory is a user-facing symlink alias to `/opt/m3tal/stack/` for all stack operations.

#### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to initialize or update all Docker Compose stacks, including your newly added one.

### Dashboard Access

The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

1.  **Local Mode (Default)**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting upon a new installation).
    *   **Mechanism**: A direct port binding is configured, typically `${DASHBOARD_PORT:-8082}:8082`. This mode utilizes an override file (`m3tal-compose.local.yml`).
    *   **Access**: A new user performing a default installation will access the dashboard directly via port `8082` at `http://HOST_IP:8082` or `http://localhost:8082`.
    *   **Requirements**: No Traefik gateway is required for this mode.
    *   **Use Case**: Ideal for LAN-only setups, initial configurations, and local testing.

2.  **Traefik Mode**
    *   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
    *   **Mechanism**: Traefik labels are added to the dashboard container's Docker Compose definition (via `m3tal-compose.traefik.yml`). Traefik interprets these labels to route `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
    *   **Access**: `http://dash.DOMAIN`.
    *   **Requirements**: The Traefik gateway must be running (`m3tal up` will start it if configured).
    *   **Use Case**: Suitable for domain-based access and integrating multiple services behind a single reverse proxy.

### Traefik Gateway

Traefik functions as the central ingress point for M3TAL services. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik loads dynamic configurations from `/docker/dynamic/`, which allows for hot-reloading of routing rules. For instance:

*   **API Daemon Routing**: Traefik routes `api.DOMAIN` to the M3TAL API daemon. This is achieved through a dynamic configuration file (e.g., `dynamic/api.yml`) that routes `http://api.${DOMAIN}` to `http://host.docker.internal:8080`, where the Go API daemon listens.
*   **Dashboard Routing**: When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels on the `m3tal-dashboard` service route `dash.DOMAIN` to the dashboard container listening internally on port `8082`.

**Example: Exposing a Custom User Service via Traefik Labels**

To expose your own Docker Compose service (e.g., `my-app`) through Traefik, add appropriate labels to its service definition in your `my-app-compose.yml` file:

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

This configuration will make your `my-app` service accessible at `http://app.DOMAIN` via Traefik, routing traffic to port `80` of the `my-app` container. The `proxy` network must be defined as external for Traefik to connect to it.

## Service Management

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage its lifecycle using standard `systemctl` commands:

*   **Check Status**: `systemctl status m3tal-api`
*   **Restart Service**: `systemctl restart m3tal-api`
*   **View Logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL Dashboard:

1.  **Start the Dashboard**: Execute `m3tal dash up`. This command specifically downloads the latest dashboard Docker Compose files and starts the `m3tal-dashboard` container with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting (defaulting to local access on port `8082`).
2.  **Access the Dashboard**:
    *   If `DASHBOARD_EXPOSE_MODE=local` (default), open your browser to `http://HOST_IP:8082` or `http://localhost:8082`.
    *   If `DASHBOARD_EXPOSE_MODE=traefik`, ensure Traefik is running via `m3tal up` and then navigate to `http://dash.DOMAIN`.

Note that while `m3tal dash up` manages only the dashboard, the general `m3tal up` command orchestrates and deploys all Docker Compose stacks found in the `/docker/` directory, including any user-defined compose files and system stacks like Traefik.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.