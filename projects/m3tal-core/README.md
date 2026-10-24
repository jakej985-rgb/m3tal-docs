# M3TAL System Documentation

## Overview
This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites
Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation
The M3TAL system is installed via an APT repository.

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
The M3TAL system consists of the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary operating as a systemd service, listening on host-local port `8080`. It manages Docker interactions, a state database, and provides API routes for the system.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask-based Docker container, internally listening on port `8082`. It communicates with the API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container providing Cloudflare tunnel capabilities for zero-configuration internet access to services.

## Filesystem Contract
The following table outlines key paths and their purposes within the M3TAL filesystem:

| Path                     | Purpose                                                    |
|--------------------------|------------------------------------------------------------|
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created by the API daemon.     |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains compose files and Traefik configuration. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.   |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` serves as the canonical source of truth for all stack files. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/` and is the user-facing path for all stack operations.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Execute `m3tal up` to start all configured stacks, including your newly added one.

## Dashboard Access
The M3TAL Dashboard offers two access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local` (This is the default setting for a new installation).
*   **Mechanism:** This mode utilizes the `m3tal-compose.local.yml` override, which directly binds the dashboard container's internal port `8082` to the host's `DASHBOARD_PORT` (defaulting to `8082`).
*   **Access:** A new user performing a default installation will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik configuration is required or involved for this mode.
*   **Use Case:** Ideal for LAN-only setups, initial system testing, and environments where a domain-based reverse proxy is not desired or configured.

### Traefik Mode
*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism:** This mode employs the `m3tal-compose.traefik.yml` override, which applies specific Traefik labels to the dashboard container. These labels instruct Traefik to route requests for `dash.DOMAIN` to the dashboard container's internal port `8082`. Traefik must be running for this mode to function.
*   **Access:** Via `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   **Use Case:** Suited for domain-based deployments and integrating the dashboard alongside other services behind a Traefik reverse proxy.

## Traefik Gateway
Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a container through `routing-compose.yml`. It listens on host port `80` (and `443` if HTTPS is configured) as its primary HTTP entry point. It automatically discovers Docker services and also loads dynamic routing configurations from files within `/docker/dynamic/` for hot-reloading.

For services not managed directly by Docker, such as the M3TAL API daemon listening on a host-local port, dynamic configuration files are used. For example, `dynamic/api.yml` routes `api.DOMAIN` requests to the Go API daemon, which is listening on the host's port `8080`, via `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` routes to the dashboard container based on its Traefik labels.

### Exposing a Custom Service via Traefik
To expose a custom user service through Traefik, add appropriate labels to its service definition in your Docker Compose file:

```yaml
# Example: my-app-compose.yml located in /docker/
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
After placing this file in `/docker/` and running `m3tal up`, Traefik will route `http://app.DOMAIN` to your `my-app` container.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard systemctl commands apply:

*   **Check Status:** `systemctl status m3tal-api`
*   **Restart Service:** `systemctl restart m3tal-api`
*   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo
This section provides a quick overview of deploying M3TAL components.

*   **Deploy the Dashboard (and its dependencies):**
    To start only the M3TAL Dashboard container, use:
    ```bash
    m3tal dash up
    ```
    This command will download the necessary Docker Compose files for the dashboard and start it using the configured `DASHBOARD_EXPOSE_MODE`. If `DASHBOARD_EXPOSE_MODE` is `local` (the default), the dashboard will be accessible directly at `http://HOST_IP:8082`.

*   **Deploy All M3TAL Stacks and User-Defined Services:**
    To deploy all M3TAL-managed Docker Compose stacks (including routing, and any other stacks you've placed in `/docker/`), execute:
    ```bash
    m3tal up
    ```
    This command orchestrates and deploys all `*-compose.yml` files found within the `/docker/` directory, including any user-defined compose files. This is the primary command for full system deployment.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.