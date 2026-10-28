# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

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

## System Architecture

The M3TAL system comprises several interconnected components:

*   **CLI binary (`/usr/bin/m3tal`):** A unified Go binary serving as the primary entry point for all system operations, including configuration, service management, and deployment.
*   **API daemon (`m3tal-api.service`):** A Go binary running as a systemd service. It listens on host-local port `8080` and is responsible for managing Docker interactions, persistent state within a SQLite database, and providing internal API routes.
*   **Dashboard container (`m3tal-dashboard`):** A Python/Flask application running within a Docker container. It listens internally on port `8082` and communicates with the M3TAL API daemon, typically at `http://host.docker.internal:8080` from within the container network.
*   **Traefik gateway (`routing-compose.yml`):** A reverse proxy container that exposes services via domain names on host port `80`. It utilizes a file provider for dynamic configuration and automatically discovers Docker services based on labels.
*   **Cloudflared (`routing-compose.yml`):** An optional Cloudflare tunnel container, providing secure, zero-configuration internet access for services.

## Filesystem Contract

The following paths are critical for M3TAL system operation and configuration:

| Path                        | Purpose                                                                                                                                                                                                                            |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file, containing environment variables for M3TAL components. This file is managed by the `m3tal config wizard` and `m3tal config set` commands.                                                          |
| `/var/lib/m3tal/state.db`   | The SQLite state database used by the M3TAL API daemon to persist system state. This file is automatically created and managed by the API daemon.                                                                                |
| `/opt/m3tal/stack/`         | The canonical directory for all Docker Compose stack definition files (`*-compose.yml`) and Traefik dynamic configuration files. This directory is the authoritative source for M3TAL's container orchestrations.                |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path intended for all Docker Compose stack operations, providing a consistent and convenient location for managing deployments.                            |
| `/docker/users.json`        | The credential store for the M3TAL Dashboard, containing hashed user passwords. This file is managed by the `m3tal dashpass` command.                                                                                          |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth where all stack definition files (e.g., `m3tal-compose.yml`, `routing-compose.yml`) and Traefik dynamic configuration files reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a user-facing and consistent path for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard offers two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (Default)

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Behavior:** This mode directly binds the Dashboard container's internal port `8082` to the host machine's port `8082`. It does not rely on Traefik for routing. This is the default setting for new installations.
*   **Access:** A new user performing a default installation will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Use Case:** Ideal for LAN-only setups, first-time users, or local testing environments where direct port access is preferred.

### Traefik Mode

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Behavior:** This mode configures the Dashboard container with Traefik labels, enabling Traefik to automatically discover and route traffic to it. Traefik handles the domain-based routing to the dashboard.
*   **Access:** The Dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.example.com`), provided Traefik is running and properly configured (e.g., via `m3tal up`).
*   **Use Case:** Suited for domain-based setups where multiple services are exposed behind a central reverse proxy like Traefik.

## Traefik Gateway

Traefik functions as the system's reverse proxy, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and binds host port `80` (and `443` if HTTPS is configured) as its primary HTTP (and HTTPS) entry point. It also loads dynamic configuration files from `/docker/dynamic/`, which allows for flexible routing rules that hot-reload upon changes.

*   **API Daemon Routing:** Traefik uses dynamic configuration, such as `/docker/dynamic/api.yml`, to route requests for `api.DOMAIN` to the M3TAL API daemon listening on the host-local port `8080`. This is achieved by routing to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.DOMAIN` to the M3TAL Dashboard container, which internally listens on port `8082`, using the labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik

To expose your own Docker Compose service through Traefik, you need to add specific Traefik labels to its service definition. Here's an example for a hypothetical `my-app-compose.yml`:

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
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN:-localhost}`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy

networks:
  proxy:
    external: true
```
After placing this file in `/docker/` and running `m3tal up`, your `my-app` service would be accessible via `http://app.DOMAIN` (e.g., `http://app.localhost`).

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. You can control and monitor it using standard `systemctl` commands:

*   **Check Status:** `systemctl status m3tal-api`
*   **Restart Service:** `systemctl restart m3tal-api`
*   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get the M3TAL Dashboard up and running for initial interaction:

*   **Start the Dashboard specifically:**
    ```bash
    m3tal dash up
    ```
    This command downloads the latest dashboard Docker Compose configuration and starts the dashboard container. By default, it will operate in **Local Mode**, making the dashboard accessible at `http://HOST_IP:8082`.
*   **Start all M3TAL stacks and user-defined services:**
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all `*-compose.yml` files found in the `/docker/` directory, including the M3TAL dashboard, Traefik, and any user-defined Docker Compose files you have placed there.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                                                                                                                |
| 8080 | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                                                                                                                           |
| 8081 | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine (e.g., `http://localhost:8081`).                                                                  |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE` configured in `/etc/m3tal/.env`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Environment Variables

The M3TAL system relies on environment variables, primarily defined in `/etc/m3tal/.env`, for configuration. These can be managed via `m3tal config wizard` or `m3tal config set KEY value`. Key variables include:

*   **`DASHBOARD_EXPOSE_MODE`**: Controls how the Dashboard is exposed (`local` or `traefik`). Default: `local`.
*   **`DASHBOARD_PORT`**: The internal port the Dashboard container listens on. Default: `8082`.
*   **`DOMAIN`**: The base domain used for Traefik routing (e.g., `example.com`). Default: `localhost`.
*   **`DASHBOARD_SECRET`**: A secret key used by the Dashboard for session management. **MUST be changed from default.** Default: `change_me_immediately`.
*   **`API_TOKEN`**: An authentication token for API access. **MUST be changed from default.** Default: `change_me_api_token`.
*   **`ADMIN_PASSWORD`**: The default administrator password for the Dashboard. **MUST be changed from default.** Default: `admin_pass`.
*   **`PUID` / `PGID`**: User and group IDs used for container filesystem permissions. Default: `1000` / `1000`.
*   **`TZ`**: Timezone setting for containers. Default: `America/Denver`.
*   **`BASE_STORAGE_PATH`**: Base directory for all data storage. Default: `./data`.
*   **`CONFIG_PATH`**: Base directory for configuration files. Default: `./data/config`.
*   **`STATE_DIR`**: Directory for the SQLite state database. Default: `./state`.
*   **`TRAEFIK_WEB_PORT` / `TRAEFIK_WEBHTTPS_PORT`**: Ports Traefik listens on for HTTP/HTTPS. Defaults: `80` / `443`.