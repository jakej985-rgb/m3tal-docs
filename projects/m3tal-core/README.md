# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
Use the following APT commands to install the M3TAL CLI binary and related system components.

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

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary that serves as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary operating as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the state database, and exposes M3TAL's internal API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container running internally on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configurations.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for establishing zero-configuration internet access to internal services.

## Filesystem Contract
The M3TAL system relies on a specific filesystem structure for configuration and operation:

| Path                        | Purpose                                                                                                                                                                          |
| :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | The primary configuration file for M3TAL, managed by the `m3tal config wizard` command. Contains environment variables for system operation.                                         |
| `/var/lib/m3tal/state.db`   | The SQLite state database, automatically created and managed by the API daemon to persist system state.                                                                            |
| `/opt/m3tal/stack/`         | The canonical directory for all Docker Compose files and Traefik dynamic configuration files. This is the source of truth for deployed stacks.                                     |
| `/docker`                   | A symbolic link pointing to `/opt/m3tal/stack/`. This is the user-facing path for managing Docker Compose files and conducting stack operations.                                  |
| `/docker/users.json`        | The credential store for the M3TAL Dashboard, managed by the `m3tal dashpass` command.                                                                                           |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside, and `/docker` is the user-facing symlink alias for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are configured in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` to manage these.
3.  Execute `m3tal up` to deploy all Docker Compose stacks, including your newly added one.

## M3TAL Dashboard Access
The M3TAL Dashboard offers two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for new installations).
*   **Mechanism:** M3TAL applies an override file (`m3tal-compose.local.yml`) that creates a direct port binding.
*   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement:** No Traefik gateway is required for this mode.
*   **Use Case:** A new user performing a default installation will access the dashboard directly via port 8082. This mode is suitable for LAN-only deployments, initial setup, and local testing environments.

### Traefik Mode
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism:** M3TAL applies an override file (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard container, allowing Traefik to route traffic to it.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   **Requirement:** The Traefik gateway must be running (`m3tal up` will deploy it).
*   **Use Case:** Ideal for domain-based setups where multiple services are exposed through a single reverse proxy.

## Traefik Gateway
Traefik is deployed as a Docker container via `routing-compose.yml` and serves as the M3TAL system's primary entry point for HTTP traffic. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds to host port `80` (and `443` if HTTPS is configured) as its public HTTP entry point. It also loads dynamic configuration files from `/docker/dynamic/` (which is symlinked to `/opt/m3tal/stack/dynamic/`), allowing for hot-reloadable routing rules.

For example:
*   Requests to `api.DOMAIN` are routed to the M3TAL API daemon listening on host-local port `8080` via a dynamic configuration file (e.g., `/docker/dynamic/api.yml`) that defines a service mapping `http://host.docker.internal:8080`.
*   Requests to `dash.DOMAIN` are routed to the M3TAL Dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`) by interpreting Traefik labels within `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik
To expose a custom user service through Traefik, add the necessary Traefik labels to its service definition within your Docker Compose file (e.g., `my-app-compose.yml`):

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
    # ... other service configurations ...

networks:
  proxy: # Ensure your service is connected to the 'proxy' network for Traefik discovery
    external: true
```
After modifying your compose file, run `m3tal up` to apply the changes.

## Service Management
The M3TAL API daemon operates as a systemd service, `m3tal-api.service`. You can manage its lifecycle using standard `systemctl` commands:

*   **Check Status:** `systemctl status m3tal-api`
*   **Restart Service:** `systemctl restart m3tal-api`
*   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo
This section outlines basic commands for deploying M3TAL components.

*   **Deploy M3TAL Dashboard:** To specifically start and manage only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command downloads the necessary compose files, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate override.

*   **Deploy All Stacks:** To orchestrate and deploy all other Docker Compose stacks, including the Traefik gateway and any user-defined compose files located in `/docker/`, use:
    ```bash
    m3tal up
    ```
    This command applies `docker compose` operations to all `*-compose.yml` files in the `/docker/` directory.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                    | Access                                        | Description                                                                                                                                |
| :--- | :------------------------- | :-------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public                                        | The public-facing HTTP port for services exposed via Traefik.                                                                              |
| 8080 | M3TAL API daemon (Go)      | Host-local                                    | The internal port the M3TAL API daemon listens on.                                                                                         |
| 8081 | Traefik dashboard          | Host-local only                               | The internal Traefik dashboard port, accessible only from the host machine.                                                                |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`.                             |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.