# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## APT Installation
```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core Components
The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on port `8080`. It is responsible for managing Docker interactions, persistent state within a SQLite database, and exposing the system's API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application containerized in Docker, listening internally on port `8082`. It communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik Gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy, exposing services via domain names on host port `80`. It utilizes a file provider for dynamic configuration and Docker labels for service discovery.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, providing secure, zero-configuration external access to services without requiring open inbound firewall ports.

## Filesystem Contract
The following paths are critical to the operation and configuration of the M3TAL system:

| Path | Purpose |
|------|---------|
| `/etc/m3tal/.env` | Primary system configuration file, managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | The SQLite database storing M3TAL's operational state, auto-created by the API daemon. |
| `/opt/m3tal/stack/` | The canonical directory where all Docker Compose stack files and Traefik dynamic configurations reside. |
| `/docker` | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json` | The credential store for the M3TAL Dashboard, managed by `m3tal dashpass`. |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` is the canonical source of truth where all Docker Compose stack files and associated configurations reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a convenient and consistent user-facing path for all stack management operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. These can be managed using `m3tal config wizard` or `m3tal config set KEY value`.
3.  Execute `m3tal up` to deploy and start all configured Docker Compose stacks, including your newly added stack.

## Dashboard Access
The M3TAL Dashboard supports two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Access:** Direct port binding at `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Mechanism:** This mode utilizes the `m3tal-compose.local.yml` override, which adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container.
*   **Behavior:** For a new user performing a default installation, the dashboard is directly accessible via `http://HOST_IP:8082`. This mode does not require Traefik to be running.
*   **Use Case:** Ideal for local area network (LAN) setups, initial installations, and direct development or testing environments.

### Traefik Mode
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Access:** Domain routing at `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   **Mechanism:** This mode utilizes the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the dashboard container definition. These labels instruct Traefik to route requests for `dash.${DOMAIN}` to the dashboard container's internal port `8082`.
*   **Prerequisite:** Traefik must be running (typically started via `m3tal up`).
*   **Use Case:** Suited for domain-based access, environments with multiple services behind a reverse proxy, or when leveraging Traefik's advanced routing and TLS capabilities.

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml`. It binds host port `80` as its primary HTTP entry point (`web`). It auto-discovers Docker services and also loads dynamic configuration files from `/docker/dynamic/` (which is symlinked from `/opt/m3tal/stack/dynamic/`).

*   **API Daemon Routing:** Traefik routes `api.DOMAIN` to the M3TAL Go API daemon running on the host-local port `8080`. This is configured via a dynamic configuration file, such as `dynamic/api.yml`, which translates `api.${DOMAIN}` requests to `http://host.docker.internal:8080`.
*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the dashboard container on its internal port `8082` based on labels in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik
To expose a custom user-defined Docker Compose service through Traefik, add appropriate labels to its service definition:

```yaml
# Example: my-app-compose.yml
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
      - proxy

networks:
  proxy:
    external: true
```
This example configures Traefik to route requests for `app.DOMAIN` to the `my-app` container, targeting its internal port `80`.

## Service Management
The M3TAL API daemon is managed as a `systemd` service named `m3tal-api.service`. Standard `systemctl` commands apply for its operation:

*   **Check Status:** `systemctl status m3tal-api`
*   **Restart Service:** `systemctl restart m3tal-api`
*   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo
This section outlines how to start specific M3TAL components or the entire system:

*   **Start the M3TAL Dashboard:** To specifically bring up only the `m3tal-dashboard` container, execute:
    ```bash
    m3tal dash up
    ```
    This command will download the necessary dashboard Compose files and start the dashboard with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting.
*   **Deploy All Stacks:** To orchestrate and deploy all Docker Compose stacks defined in the `/docker/` directory (including M3TAL's core routing, and any user-defined Compose files):
    ```bash
    m3tal up
    ```

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.