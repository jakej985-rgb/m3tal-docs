# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system, covering its architecture, deployment, management, and network configuration.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

To install the M3TAL CLI binary and API daemon, use the following APT commands:

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

The M3TAL system adheres to the following filesystem structure:

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik configuration. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Core Components

The M3TAL system comprises the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container running internally on port `8082`. It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for establishing secure, zero-configuration internet access to services.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files (Docker Compose files and Traefik dynamic configurations) reside. The `/docker` path is a user-facing symlink alias to `/opt/m3tal/stack/` for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Setting:** `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism:** This mode utilizes `m3tal-compose.local.yml` to add a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container.
*   **Access:** Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement:** No Traefik gateway is required for this mode.
*   **Behavior:** A new user performing a default M3TAL installation will access the dashboard directly via port `8082`. This behavior is directly linked to the default setting of `DASHBOARD_EXPOSE_MODE=local`.

### Traefik Mode

*   **Setting:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** This mode utilizes `m3tal-compose.traefik.yml` to add Traefik labels to the dashboard service definition, enabling Traefik to route traffic to the dashboard container on its internal port `8082`.
*   **Access:** Via `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN=localhost`).
*   **Requirement:** The Traefik gateway must be running (e.g., deployed via `m3tal up`).

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a Docker container, typically defined in `routing-compose.yml`. It binds host port `80` (and `443` if HTTPS is configured) as its public HTTP entry point.

For services not managed directly by Docker Compose labels (e.g., host-local services), Traefik uses dynamic configuration files. For example, `dynamic/api.yml` located within `/docker/dynamic/` (which maps to `/etc/traefik/dynamic` inside the Traefik container) routes `api.DOMAIN` to the M3TAL Go API daemon. This is achieved by configuring a load balancer that targets `http://host.docker.internal:8080`, directing requests to the M3TAL API daemon listening on host-local port `8080`. Similarly, `dash.DOMAIN` routes to the dashboard container when `DASHBOARD_EXPOSE_MODE=traefik` via labels.

### Exposing a Custom Service via Traefik

To expose a custom user service, define the necessary Traefik labels in its Docker Compose service definition:

```yaml
# my-app-compose.yml (located in /docker/)
services:
  my-app:
    image: nginx:alpine
    container_name: my-app
    restart: unless-stopped
    labels:
      - "traefik.enable=true" # Enable Traefik for this service
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)" # Route requests for app.DOMAIN
      - "traefik.http.services.myapp.loadbalancer.server.port=80" # Target internal container port 80
      - "traefik.http.routers.myapp.entrypoints=web" # Use the 'web' entrypoint (HTTP port 80)
    networks:
      - proxy # Ensure the service is on the 'proxy' network for Traefik discovery

networks:
  proxy:
    external: true # Use the existing external 'proxy' network
```

After placing `my-app-compose.yml` in `/docker/`, run `m3tal up` to deploy the service and make it accessible via Traefik.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`.
Common `systemctl` commands include:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL Dashboard:

1.  **Start the M3TAL Dashboard:**
    ```bash
    m3tal dash up
    ```
    This command specifically manages the M3TAL Dashboard container. It downloads the necessary `m3tal-compose.yml` and override files, then starts the dashboard using the appropriate override (`.local.yml` or `.traefik.yml`) based on your `DASHBOARD_EXPOSE_MODE` setting.

2.  **Access the Dashboard:**
    *   If `DASHBOARD_EXPOSE_MODE=local` (the default), access the dashboard at `http://HOST_IP:8082`.
    *   If `DASHBOARD_EXPOSE_MODE=traefik` and Traefik is running, access it at `http://dash.DOMAIN`.

3.  **Deploy all other services (optional):**
    ```bash
    m3tal up
    ```
    The `m3tal up` command orchestrates and deploys all other stacks defined by `*-compose.yml` files found in the `/docker/` directory, including any user-defined compose files you have added.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration (Environment Variables)

The M3TAL system utilizes various environment variables for configuration, typically managed within `/etc/m3tal/.env`. A selection of key variables and their default values are:

| Key | Default |
|---|---|
| API_PORT | 5050 |
| DASHBOARD_PORT | 8082 |
| DASHBOARD_EXPOSE_MODE | local |
| HTTP_PORT | 8080 |
| STATE_DIR | ./state |
| LOG_LEVEL | info |
| DASHBOARD_SECRET | change_me_immediately |
| API_TOKEN | change_me_api_token |
| ADMIN_PASSWORD | admin_pass |
| NETWORK_NAME | m3tal |
| LOCAL_IP | 127.0.0.1 |
| DOMAIN | localhost |
| VPN_USER | user |
| VPN_PASSWORD | password |
| BASE_STORAGE_PATH | ./data |
| MEDIA_PATH | ./data/media |
| CONFIG_PATH | ./data/config |
| DOWNLOADS_PATH | ./data/downloads |
| PUID | 1000 |
| PGID | 1000 |
| TZ | America/Denver |
| TRAEFIK_WEB_PORT | 80 |
| TRAEFIK_WEBHTTPS_PORT | 443 |
| TRAEFIK_DASHBOARD_PORT | 8080 |
| DEBUG_MODE | false |
| METRICS_ENABLED | true |