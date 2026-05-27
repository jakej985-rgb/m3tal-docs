# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## APT Installation

To install the M3TAL CLI and API daemon:

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

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing a single entry point for all M3TAL operations. It is installed via APT.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It dynamically discovers services through Docker labels and loads additional routing configurations from a file provider.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for establishing zero-configuration internet access to services.

## Filesystem Contract

The following paths represent the canonical filesystem structure and their respective purposes within the M3TAL system:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`         | Canonical stack directory. Contains Docker Compose files and Traefik dynamic configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The directory `/opt/m3tal/stack/` is the canonical source of truth where all M3TAL-managed Docker Compose stack files reside. The `/docker` directory is a symlink to `/opt/m3tal/stack/`, serving as the user-facing alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. This can be done using `m3tal config wizard` or `m3tal config set KEY value`.
3.  Run `m3tal up` to deploy all Docker Compose stacks, including your newly added one.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a container via `routing-compose.yml`. It binds host port `80` as its primary HTTP entry point (`web`).

For services that are not Docker containers or listen on host-local ports, Traefik utilizes dynamic configuration files. For example, `dynamic/api.yml` routes `api.DOMAIN` to the M3TAL API daemon listening on host-local port `8080` by directing traffic to `http://host.docker.internal:8080`. When `DASHBOARD_EXPOSE_MODE=traefik` is configured, `dash.DOMAIN` is routed to the `m3tal-dashboard` container via Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik

To expose a custom user service, define Traefik labels within its Docker Compose service definition:

```yaml
# my-app-compose.yml
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

After placing this file in `/docker/` and running `m3tal up`, `my-app` will be accessible via `http://app.DOMAIN` (assuming Traefik is running and `DOMAIN` is correctly configured).

## Dashboard Access

The M3TAL Dashboard offers two access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (default setting).
*   **Mechanism**: This mode utilizes the `m3tal-compose.local.yml` override file, which adds a direct Docker port binding of `${DASHBOARD_PORT:-8082}:8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Note**: A new user performing a default M3TAL installation will access the dashboard directly via port `8082`. This mode does not require Traefik for dashboard access.
*   **Best for**: LAN-only setups, initial installation, and local testing.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism**: This mode uses the `m3tal-compose.traefik.yml` override file, which applies Traefik labels to the dashboard container. Traefik then routes `http://dash.DOMAIN` to the dashboard container on its internal port `8082`. This requires Traefik to be running via `m3tal up`.
*   **Access**: Via `http://dash.DOMAIN`.
*   **Best for**: Domain-based setups and environments where multiple services are managed behind a Traefik reverse proxy.

## Quick Demo

To quickly start the M3TAL Dashboard:

*   Execute `m3tal dash up`. This command specifically downloads the latest dashboard Docker Compose files and starts the dashboard container with the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting.

To deploy all M3TAL-managed services and any user-defined Docker Compose stacks within the `/docker/` directory:

*   Execute `m3tal up`. This orchestrates and deploys all active stacks, including the Traefik gateway, Cloudflared, and any other `*-compose.yml` files you have placed in `/docker/`.

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard `systemctl` commands can be used for its operation and status monitoring.

*   **Check service status**: `systemctl status m3tal-api`
*   **Restart the API daemon**: `systemctl restart m3tal-api`
*   **View service logs**: `journalctl -u m3tal-api -f`

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service               | Access                                  | Description                                                                                                                                              |
| :--- | :-------------------- | :-------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point | Public                                  | The public-facing HTTP port for services exposed via Traefik.                                                                                            |
| 8080 | M3TAL API daemon (Go)  | Host-local                              | The internal port the M3TAL API daemon listens on.                                                                                                       |
| 8081 | Traefik dashboard     | Host-local only                         | The internal Traefik dashboard port, accessible only from the host machine.                                                                              |
| 8082 | M3TAL Dashboard       | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.