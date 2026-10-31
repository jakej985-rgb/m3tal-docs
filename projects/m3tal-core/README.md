# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

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

## System Components

The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary interface for system operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the state database, and exposes core API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A reverse proxy Docker container that exposes services by domain name on host port `80`. It uses a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container for zero-configuration internet access.

## Filesystem Contract

The following table details critical directories and files within the M3TAL filesystem:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file for environment variables. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`   | SQLite database storing system state. Auto-created by the API daemon.  |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik dynamic configurations. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store for user authentication. Managed by `m3tal dashpass`. |

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory serves as the canonical source of truth where all stack files, including M3TAL's internal ones, reside. The `/docker` directory is a user-facing symlink alias to `/opt/m3tal/stack/`, providing a convenient and consistent location for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`.
3.  Execute `m3tal up` to deploy the new stack alongside existing ones.

## Dashboard Access

The M3TAL Dashboard provides two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local` (this is the default setting after a new installation).
*   **Mechanism**: This mode adds a direct port binding (`${DASHBOARD_PORT:-8082}:8082`) to the dashboard container via `m3tal-compose.local.yml`.
*   **Access**: The dashboard is directly accessible at `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirements**: Traefik is not required for this access method.
*   **Use Case**: Ideal for LAN-only setups, initial configurations, first-time users, and local development. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`.
*   **Mechanism**: This mode applies Traefik labels to the dashboard container via `m3tal-compose.traefik.yml`, enabling Traefik to route `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   **Access**: The dashboard is accessible via `http://dash.DOMAIN`.
*   **Requirements**: The Traefik gateway (`routing-compose.yml`) must be running via `m3tal up`.
*   **Use Case**: Best for domain-based setups where multiple services are exposed behind a central reverse proxy.

## Traefik Gateway

Traefik functions as the primary reverse proxy for M3TAL, automatically discovering and routing traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed as a container from `routing-compose.yml`. It listens on host port `80` (and `443` if configured) and uses a file provider to load dynamic configuration from `/docker/dynamic/` for advanced routing scenarios.

For example, `dynamic/api.yml` is used to route `api.DOMAIN` to the M3TAL Go API daemon, which listens on the host-local port `8080`, using `http://host.docker.internal:8080`:

```yaml
# /docker/dynamic/api.yml
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)"
      service: api
      entryPoints:
        - web

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080"
```

The `dash.DOMAIN` route is handled by Traefik labels directly on the dashboard container, as configured when `DASHBOARD_EXPOSE_MODE=traefik`.

### Exposing a Custom Service via Traefik

To expose your own Docker Compose service using Traefik, add the necessary Traefik labels to its service definition. This example demonstrates how to expose a hypothetical `my-app` service at `http://app.DOMAIN`:

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network for Traefik discovery

networks:
  proxy:
    external: true # Assumes the 'proxy' network is external and managed by M3TAL
```

## Service Management

The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. Standard systemctl commands are used for its operation:

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get the M3TAL Dashboard up and running:

1.  Ensure Docker Engine and Docker Compose V2 are installed.
2.  Install M3TAL using the APT instructions above.
3.  Initialize M3TAL configuration (if not already done):
    `sudo m3tal config wizard`
    (Accept defaults or configure as needed. `DASHBOARD_EXPOSE_MODE` defaults to `local`.)
4.  Start the M3TAL Dashboard container specifically:
    `m3tal dash up`
    This command downloads the necessary dashboard Docker Compose files, applies the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting, and starts the `m3tal-dashboard` container.
5.  Access the dashboard: If `DASHBOARD_EXPOSE_MODE` is `local` (the default), navigate to `http://HOST_IP:8082` in your browser.

To deploy all other M3TAL core services and any user-defined Docker Compose stacks in `/docker/`:

`m3tal up`

This command orchestrates and deploys all `*-compose.yml` files found within the `/docker/` directory, including the Traefik gateway and any custom user-defined compose files.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                    | Access                                | Description                                                          |
| :--- | :------------------------- | :------------------------------------ | :------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point   | Public                                | The public-facing HTTP port for services exposed via Traefik.        |
| 8080 | M3TAL API daemon (Go)      | Host-local                            | The internal port the M3TAL API daemon listens on.                   |
| 8081 | Traefik dashboard          | Host-local only                       | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard            | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.