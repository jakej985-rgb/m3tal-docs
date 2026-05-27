# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**

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

## Components

The M3TAL system is composed of the following primary components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the single entrypoint for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the SQLite state database, and exposes API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application running within a Docker container. It listens on internal container port `8082` and communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services via domain names on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, enabling secure, zero-configuration internet access for services.

## Filesystem Contract

The following table outlines the critical file and directory paths used by the M3TAL system:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file. Managed by `m3tal config wizard`.          |
| `/var/lib/m3tal/state.db`   | SQLite state database. Auto-created and managed by the API daemon.     |
| `/opt/m3tal/stack/`         | Canonical directory for Docker Compose files and Traefik configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Dashboard credential store. Managed by `m3tal dashpass`.               |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all M3TAL-managed stack files reside. The `/docker` directory is a symlink alias to `/opt/m3tal/stack/`, providing a convenient, user-facing path for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env` (e.g., using `m3tal config set KEY value`).
3.  Execute `m3tal up` to deploy your new stack along with all other M3TAL-managed services.

## Dashboard Access

The M3TAL dashboard offers two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the default configuration for a new installation.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE` is set to `local`.
*   **Mechanism:** M3TAL applies an override (`m3tal-compose.local.yml`) that creates a direct port binding, typically `${DASHBOARD_PORT:-8082}:8082`.
*   **Access:** The dashboard is directly accessible via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Traefik:** Traefik is not required for access in this mode.
*   **Usage:** Ideal for LAN-only setups, first-time users performing a default installation, and local testing. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

This mode integrates the dashboard with the Traefik reverse proxy.
*   **Configuration:** `DASHBOARD_EXPOSE_MODE` is set to `traefik`.
*   **Mechanism:** M3TAL applies an override (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard container. These labels instruct Traefik to route requests for `dash.DOMAIN` to the dashboard service on its internal port `8082`.
*   **Access:** The dashboard is accessible via `http://dash.DOMAIN` (e.g., `http://dash.localhost` if `DOMAIN` is `localhost`).
*   **Traefik:** Requires Traefik to be running via `m3tal up` for routing to function.
*   **Usage:** Suitable for domain-based setups and environments where multiple services are exposed through a central reverse proxy.

## Traefik Gateway

Traefik operates as the central ingress point for HTTP traffic within the M3TAL system. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik loads its dynamic configuration from the `/docker/dynamic/` directory (which is `/opt/m3tal/stack/dynamic/`). This directory contains YAML files that define additional routers and services, allowing for flexible routing without restarting Traefik. For instance, the `dynamic/api.yml` file is used to route requests for `api.DOMAIN` to the Go API daemon, which listens on the host-local port `8080`, specifically `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, the `dash.DOMAIN` route is configured via labels to point to the `m3tal-dashboard` container.

### Exposing a Custom User Service via Traefik

To expose a custom Docker Compose service (e.g., defined in `my-app-compose.yml`) through Traefik, add the necessary Traefik labels to its service definition:

```yaml
# my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-app-container
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network for Traefik to discover it

networks:
  proxy:
    external: true # This network is managed by Traefik
```
After placing `my-app-compose.yml` in `/docker/` and running `m3tal up`, Traefik will route `http://app.DOMAIN` to your `my-app` container.

## Service Management

The M3TAL API daemon runs as a systemd service, `m3tal-api.service`, and can be managed using standard `systemctl` commands:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL dashboard:

*   To specifically start only the M3TAL dashboard container, execute:
    ```bash
    m3tal dash up
    ```
    This command will download the necessary compose files, configure the dashboard based on your `DASHBOARD_EXPOSE_MODE` setting, and bring up the dashboard container.

*   To orchestrate and deploy all M3TAL-managed stacks, including any user-defined compose files (e.g., `my-stack-compose.yml`) placed in the `/docker/` directory, use the comprehensive deployment command:
    ```bash
    m3tal up
    ```
    This command will scan `/docker/` for all `*-compose.yml` files and deploy them.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
| :--- | :-------------------------- | :--------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.