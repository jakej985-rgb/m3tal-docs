# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install M3TAL, follow these steps to add the APT repository and install the `m3tal` package:

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

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary installed via APT, serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host port `8080`. It manages Docker interactions, the internal state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing zero-configuration internet access for exposed services.

## Filesystem Contract

The following paths define the primary filesystem contract for M3TAL operations:

| Path                        | Purpose                                                            |
| :-------------------------- | :----------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary system configuration file, managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite state database, automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | The canonical directory for all Docker Compose stack files and Traefik configuration. |
| `/docker`                   | A symlink pointing to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store, managed by the `m3tal dashpass` command. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth for all Docker Compose stack files. `/docker` is a user-facing symlink alias to `/opt/m3tal/stack/` for simplified stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your new one.

## Dashboard Access

The M3TAL Dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** This mode uses the `m3tal-compose.local.yml` override file, which adds a direct port binding to the dashboard container (`${DASHBOARD_PORT:-8082}:8082`).
*   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements:** No Traefik configuration is required.
*   **Behavior for new users:** A new user performing a default installation will access the dashboard directly via port 8082. This direct access via port 8082 is due to the default setting `DASHBOARD_EXPOSE_MODE=local`.
*   **Use Case:** Ideal for LAN-only setups, first-time users, or local development and testing environments.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** This mode utilizes the `m3tal-compose.traefik.yml` override file, which adds specific Traefik labels to the dashboard service. These labels instruct Traefik to route requests for `dash.DOMAIN` to the dashboard container, which listens internally on port `8082`.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN`.
*   **Requirements:** Traefik must be running and properly configured (e.g., via `m3tal up`).
*   **Use Case:** Suited for domain-based setups where multiple services are exposed behind a single reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a Docker container via `routing-compose.yml`. It binds host port `80` as its HTTP entry point and loads dynamic configuration from files in `/docker/dynamic/`, which are hot-reloaded upon changes.

For internal services, Traefik routes requests to services listening on host-local ports. For example, `api.DOMAIN` is routed to the M3TAL Go API daemon, which listens on host-local port `8080`, via the dynamic configuration in `dynamic/api.yml`. This configuration explicitly maps `api.DOMAIN` to `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` routes to the dashboard container when `DASHBOARD_EXPOSE_MODE=traefik` is set.

### Example: Exposing a Custom User Service via Traefik

To expose a hypothetical `my-app` service through Traefik, add the following labels to its service definition in your Docker Compose file (e.g., `my-app-compose.yml`):

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
networks:
  proxy:
    external: true
```

This configuration tells Traefik to:
*   Enable routing for `my-app`.
*   Route traffic for `app.DOMAIN` to this service.
*   Target port `80` (the default HTTP port for Nginx) within the container.
*   Use the `web` entrypoint, which typically corresponds to host port `80`.
*   Connect the service to the `proxy` network, which Traefik monitors.

## Quick Demo

To quickly get started with the M3TAL Dashboard:

*   To start *only* the dashboard container, use the command:
    ```bash
    m3tal dash up
    ```
    This command specifically manages the dashboard container, downloading the necessary Docker Compose files and starting it with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting.
*   To orchestrate and deploy *all* other stacks, including any user-defined compose files located in `/docker/`, use the general deployment command:
    ```bash
    m3tal up
    ```

## Service Management

The M3TAL API daemon (`m3tal-api.service`) runs as a systemd service. You can manage it using standard `systemctl` commands:

*   **Check status:**
    ```bash
    systemctl status m3tal-api.service
    ```
*   **Restart the service:**
    ```bash
    systemctl restart m3tal-api.service
    ```
*   **View logs:**
    ```bash
    journalctl -u m3tal-api.service -f
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                                       |
| 8080 | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                                                  |
| 8081 | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine.                         |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.