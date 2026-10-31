# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL system is distributed via an APT repository. Follow these steps to install the core components:

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

## Overview of Components

The M3TAL system is comprised of several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations, installed via APT.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask Docker container that provides a web-based user interface. It runs internally on port `8082` and communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel Docker container for establishing secure, zero-configuration internet access to services.

## Filesystem Contract

The following table details the critical filesystem paths and their purposes within the M3TAL system:

| Path                     | Purpose                                                      |
| :----------------------- | :----------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file for M3TAL. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database. Auto-created and managed by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical directory for all Docker Compose stack files and Traefik dynamic configurations. |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.     |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside. The `/docker` directory serves as a user-facing symlink alias for `/opt/m3tal/stack/` to simplify all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the default configuration for a new M3TAL installation.
*   A direct port binding is configured for the Dashboard container.
*   Access is provided via `http://HOST_IP:8082` or `http://localhost:8082`.
*   No Traefik reverse proxy is required for this mode.
*   A new user performing a default installation will access the dashboard directly via port 8082. This behavior is linked to the default setting `DASHBOARD_EXPOSE_MODE=local`.
*   This mode is suitable for LAN-only setups, first-time users, and local testing.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

*   This mode configures Traefik labels on the Dashboard container, allowing Traefik to route traffic to it.
*   Access is provided via `http://dash.DOMAIN` (e.g., `http://dash.example.com`).
*   Requires the Traefik gateway to be running via `m3tal up`.
*   This mode is suitable for domain-based setups and integrating multiple services behind a reverse proxy.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a container via `routing-compose.yml`. It binds host port `80` as its HTTP entry point and loads dynamic configuration from `/docker/dynamic/` for hot-reloading.

*   **Dynamic Configuration:** Traefik utilizes dynamic configuration files (such as `dynamic/api.yml`) to route requests to services that may not be directly managed by Docker Compose or are listening on host-local ports. For example, `api.DOMAIN` is routed to the M3TAL Go API daemon listening on host-local port `8080` via `http://host.docker.internal:8080`.
*   **Dashboard Routing:** In Traefik mode (`DASHBOARD_EXPOSE_MODE=traefik`), `dash.DOMAIN` routes to the M3TAL Dashboard container (internal port `8082`) via its defined Traefik labels.

### Exposing a Custom Service via Traefik

To expose a custom user service through Traefik, add the necessary Traefik labels to its service definition within your Docker Compose file (e.g., `my-app-compose.yml` located in `/docker/`):

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

Ensure your service is part of the `proxy` network, which Traefik is configured to monitor.

## Service Management

The M3TAL API daemon is managed as a `systemd` service named `m3tal-api.service`. Standard `systemctl` commands can be used for its operation:

*   **Check Status:** `systemctl status m3tal-api.service`
*   **Restart Service:** `systemctl restart m3tal-api.service`
*   **View Logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo

To quickly get started and manage specific M3TAL components:

*   To start only the M3TAL Dashboard container, use:
    `m3tal dash up`
    This command specifically manages the dashboard container: it downloads the latest Dashboard compose files, reads `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate compose override.
*   To deploy and orchestrate all M3TAL core stacks, including any user-defined Docker Compose files placed in the `/docker/` directory, use:
    `m3tal up`
    This command acts as a wrapper around `docker compose` to bring up all `*-compose.yml` files found in the `/docker/` directory.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                    | Access                                        | Description                                                                                             |
| :--- | :------------------------- | :-------------------------------------------- | :------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point   | Public                                        | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)      | Host-local                                    | The internal port the M3TAL API daemon listens on.                                                      |
| 8081 | Traefik dashboard          | Host-local only                               | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration

The primary configuration for M3TAL is managed through environment variables defined in the `/etc/m3tal/.env` file. This file can be interactively managed using the `m3tal config wizard` command, or individual variables can be set using `m3tal config set KEY value`.