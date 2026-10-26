# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

### APT Installation
To install the M3TAL CLI and API daemon via APT:

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
The M3TAL system comprises several key components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the state database, and exposes core API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling zero-configuration internet access for services.

## Filesystem Contract
The following table outlines the critical file paths and their respective purposes within the M3TAL system:

| Path                     | Purpose                                                         |
| :----------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db` | SQLite state database. Automatically created by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configurations. |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.        |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, with `/opt/m3tal/stack/` being the canonical source of truth directory where all stack files reside.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your new one.

## Dashboard Access
The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for new installations).
*   **Mechanism**: Uses an override file (`m3tal-compose.local.yml`) to add a direct port binding, exposing the dashboard container's internal port `8082` directly on the host.
*   **Access**: You can access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: No Traefik gateway is required for this mode. A new user performing a default installation will access the dashboard directly via port `8082`.
*   **Use Case**: Ideal for LAN-only setups, first-time users, and local development environments.

### Traefik Mode
*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: Uses an override file (`m3tal-compose.traefik.yml`) to add Traefik labels to the dashboard service. Traefik then routes requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: Access the dashboard via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
*   **Requirements**: The Traefik gateway must be running (`m3tal up` must have deployed the `routing` stack).
*   **Use Case**: Suited for domain-based setups where multiple services are managed behind a single reverse proxy.

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a container via `routing-compose.yml`. It binds host port `80` (and `443` if HTTPS is configured) as its HTTP entry point. It discovers services automatically through the Docker provider and also loads dynamic configuration from files in `/docker/dynamic/`, allowing for hot-reloading of routing rules.

For example, Traefik routes requests for `api.DOMAIN` to the M3TAL API daemon. Since the API daemon runs directly on the host (not in a Docker container managed by Traefik), this routing is achieved via a dynamic configuration file (e.g., `dynamic/api.yml`) that targets `http://host.docker.internal:8080`. Similarly, `dash.DOMAIN` routes to the dashboard container when `DASHBOARD_EXPOSE_MODE=traefik`, utilizing labels defined directly on the dashboard service in its Docker Compose override.

### Exposing a Custom Service via Traefik
To expose your own Docker Compose service through Traefik, add the following labels to your service definition in its `*-compose.yml` file:

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
      - proxy # Ensure your service is on the 'proxy' network if Traefik is
```

After modifying your Docker Compose file, run `m3tal up` to apply the changes and make your service discoverable by Traefik.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. You can interact with it using standard systemctl commands:

*   **Check Status**: `systemctl status m3tal-api`
*   **Restart Service**: `systemctl restart m3tal-api`
*   **View Logs**: `journalctl -u m3tal-api -f`

## Quick Demo
This section provides a quick way to get the M3TAL Dashboard up and running.

1.  **Start the Dashboard**: To specifically deploy and start only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command downloads the necessary `m3tal-compose.yml` files, reads the `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate Docker Compose override. By default (`DASHBOARD_EXPOSE_MODE=local`), you will then be able to access the dashboard at `http://HOST_IP:8082`.

2.  **Deploy All Stacks**: To orchestrate and deploy all Docker Compose stacks found in the `/docker/` directory (including the routing stack, dashboard, and any user-defined compose files), execute:
    ```bash
    m3tal up
    ```

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------- | :------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                   | The public-facing HTTP port for services exposed via Traefik.                                |
| 8080 | M3TAL API daemon (Go)       | Host-local                               | The internal port the M3TAL API daemon listens on.                                           |
| 8081 | Traefik dashboard           | Host-local only                          | The internal Traefik dashboard port, accessible only from the host machine.                  |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Environment Variables
M3TAL uses environment variables defined in `/etc/m3tal/.env` for configuration. These variables influence the behavior of the API daemon and Docker Compose stacks. The `m3tal config wizard` command assists in managing these settings.

Commonly used environment variables include:

*   `API_PORT`: The port the API daemon listens on (default: `5050` but overridden to `8080` by systemd service).
*   `DASHBOARD_PORT`: The internal port for the Dashboard container (default: `8082`).
*   `DASHBOARD_EXPOSE_MODE`: Controls how the dashboard is exposed (`local` or `traefik`, default: `local`).
*   `DOMAIN`: The base domain used for Traefik routing (default: `localhost`).
*   `PUID`, `PGID`: User and group IDs for container processes (default: `1000`).
*   `TZ`: Timezone setting (default: `America/Denver`).
*   `BASE_STORAGE_PATH`: Base path for persistent data (default: `./data`).
*   `TRAEFIK_WEB_PORT`, `TRAEFIK_WEBHTTPS_PORT`: Public ports for Traefik (default: `80`, `443`).
*   `DASHBOARD_SECRET`, `API_TOKEN`, `ADMIN_PASSWORD`: Security-related credentials.