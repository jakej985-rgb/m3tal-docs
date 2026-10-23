# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract

The following paths are critical for M3TAL operation and configuration:

| Path                      | Purpose                                                                      |
| :------------------------ | :--------------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | Primary configuration file. Managed by `m3tal config wizard`.                |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon.                     |
| `/opt/m3tal/stack/`       | Canonical stack directory. Contains compose files and Traefik config.        |
| `/docker`                 | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`      | Dashboard credential store. Managed by `m3tal dashpass`.                     |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a symbolic link to `/opt/m3tal/stack/`. `/opt/m3tal/stack/` is the canonical source of truth directory where all stack files reside, and `/docker` is the user-facing symlink alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any necessary environment variables for your new stack are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3. Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable within `/etc/m3tal/.env`:

### Local Mode (Default)

- **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
- **Mechanism:** This mode utilizes the `m3tal-compose.local.yml` override, which adds a direct port binding to the dashboard container.
- **Access:** The dashboard is accessible directly via `http://HOST_IP:8082` or `http://localhost:8082`.
- **Requirements:** No additional reverse proxy like Traefik is needed.
- **Use Case:** Ideal for local development, home server setups, or initial installations where direct access is preferred. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode

- **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Mechanism:** This mode utilizes the `m3tal-compose.traefik.yml` override, which applies Traefik labels to the dashboard service. Traefik then handles routing based on these labels.
- **Access:** The dashboard is accessible via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
- **Requirements:** Traefik must be running and configured to route traffic to the dashboard container.
- **Use Case:** Suitable for environments where multiple services are managed behind a single reverse proxy and domain-based access is desired.

## Traefik Gateway

The Traefik gateway acts as the primary reverse proxy for services managed by M3TAL, exposing them on host port `80` (and `443` if HTTPS is configured).

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Dynamic configuration files, located within `/opt/m3tal/stack/dynamic/`, are utilized by Traefik's file provider for dynamic routing configurations and hot-reloading. For instance, `dynamic/api.yml` is used to route requests for `api.DOMAIN` to the Go API daemon listening on host-local port `8080` via `http://host.docker.internal:8080`.

The dashboard itself is routed via Traefik when `DASHBOARD_EXPOSE_MODE=traefik` using the label `traefik.http.routers.dashboard.rule=Host(dash.${DOMAIN:-localhost})`, directing traffic to the dashboard container on port `8082`.

### Exposing a Custom User Service via Traefik

To expose a custom user service via Traefik, you must define the appropriate Traefik labels within its Docker Compose file. For example, to expose a hypothetical `my-app` service running an Nginx container on port `80` and accessible via `app.DOMAIN`:

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
```

## Service Management

The M3TAL API daemon runs as a systemd service. The following commands can be used to manage its lifecycle:

- **Check status:** `systemctl status m3tal-api`
- **Restart the service:** `systemctl restart m3tal-api`
- **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly start the M3TAL Dashboard:

1. Run `m3tal dash up`. This command specifically orchestrates and deploys only the dashboard container, using the appropriate override based on your `DASHBOARD_EXPOSE_MODE` setting.

To deploy all M3TAL stacks, including any user-defined stacks placed in `/docker/`:

1. Run `m3tal up`. This command orchestrates and deploys all other stacks in the `/docker/` directory, ensuring all services are running.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port   | Service                           | Access                                       | Description                                                                              |
| :----- | :-------------------------------- | :------------------------------------------- | :--------------------------------------------------------------------------------------- |
| 80     | Traefik HTTP entry point          | Public                                       | The public-facing HTTP port for services exposed via Traefik.                            |
| 8080   | M3TAL API daemon (Go)             | Host-local                                   | The internal port the M3TAL API daemon listens on.                                       |
| 8081   | Traefik dashboard               | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine.              |
| 8082   | M3TAL Dashboard                   | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).