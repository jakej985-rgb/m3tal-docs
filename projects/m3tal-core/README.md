# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is distributed via an APT repository. Follow these steps for installation:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary M3TAL configuration is managed via the `/etc/m3tal/.env` file. This file is populated and updated using the `m3tal config wizard` command or individual `m3tal config set KEY value` commands.

## Filesystem Contract

The following paths define the M3TAL filesystem structure and their respective purposes:

| Path                         | Purpose                                                                 |
| :--------------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`            | Primary configuration file. Managed by `m3tal config wizard`.           |
| `/var/lib/m3tal/state.db`    | SQLite state database. Auto-created by the API daemon.                |
| `/opt/m3tal/stack/`          | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                    | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`         | Dashboard credential store. Managed by `m3tal dashpass`.              |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a user-facing symlink alias that points to the canonical source of truth for all stack definitions, located at `/opt/m3tal/stack/`. Operations performed on `/docker/` are transparently directed to `/opt/m3tal/stack/`.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`.
3.  Run `m3tal up` to deploy the new stack alongside existing M3TAL services.

## Dashboard Access

The M3TAL Dashboard has two primary access modes, configurable via the `DASHBOARD_EXPOSE_MODE` environment variable within `/etc/m3tal/.env`.

### Local Mode (Default)

When `DASHBOARD_EXPOSE_MODE` is set to `local` (which is the default for new installations), the dashboard is directly accessible via its configured port on the host machine.

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
*   **Access:** `http://HOST_IP:8082` or `http://localhost:8082`. The dashboard container binds directly to port 8082 on the host.
*   **Requirements:** No additional services like Traefik are required for dashboard access in this mode.

A new user performing a default installation will access the dashboard directly via port 8082, without needing any domain configuration.

### Traefik Mode

When `DASHBOARD_EXPOSE_MODE` is set to `traefik`, the dashboard is exposed via the Traefik reverse proxy, allowing access through a domain name.

*   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
*   **Access:** `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
*   **Requirements:** Traefik must be running and configured to route traffic for `dash.DOMAIN` to the dashboard container.

## Traefik Gateway

Traefik serves as the primary reverse proxy for M3TAL, managing external access to various services. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik's dynamic configuration allows for flexible routing rules. Dynamic configuration files are located within the `/opt/m3tal/stack/dynamic/` directory and are automatically loaded and watched by Traefik. This enables hot-reloading of routing configurations without restarting Traefik.

For example, Traefik can route requests for `api.DOMAIN` to the M3TAL API daemon, which listens on host-local port `8080`, via a dynamic configuration file like `dynamic/api.yml`:

```yaml
# /opt/m3tal/stack/dynamic/api.yml
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

The dashboard, when in `traefik` mode, is similarly routed to `http://dash.DOMAIN` via Traefik labels defined within its compose file.

To expose a custom user service via Traefik, add the appropriate labels to its service definition within its Docker Compose file. For instance, to expose a hypothetical `my-app` service listening on port 80:

```yaml
# Example: /docker/my-app-compose.yml
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

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Service Management

The M3TAL API daemon is managed by systemd. You can interact with the service using the following commands:

*   `sudo systemctl status m3tal-api` - Check the status of the M3TAL API service.
*   `sudo systemctl restart m3tal-api` - Restart the M3TAL API service.
*   `sudo journalctl -u m3tal-api -f` - View the logs of the M3TAL API service in real-time.

## Quick Demo

To quickly start the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command specifically orchestrates and deploys the dashboard container. The `m3tal up` command, on the other hand, orchestrates and deploys all other stacks defined in the `/docker/` directory, including any user-defined compose files.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port   | Service                   | Access                                    | Description                                                                   |
| :----- | :------------------------ | :---------------------------------------- | :---------------------------------------------------------------------------- |
| 80     | Traefik HTTP entry point  | Public                                    | The public-facing HTTP port for services exposed via Traefik.                 |
| 8080   | M3TAL API daemon (Go)     | Host-local                                | The internal port the M3TAL API daemon listens on.                            |
| 8081   | Traefik dashboard         | Host-local only                           | The internal Traefik dashboard port, accessible only from the host machine.   |
| 8082   | M3TAL Dashboard           | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Docker Services State

The following JSON represents the state of M3TAL-managed Docker services:

```json
[
  {
    "name": "m3tal-dashboard",
    "image": "ghcr.io/jakej985-rgb/m3tal-godash:debug",
    "ports": [],
    "stack": "m3tal"
  },
  {
    "name": "traefik",
    "image": "traefik:latest",
    "ports": [
      "${TRAEFIK_WEB_PORT:-80}:80",
      "${TRAEFIK_WEBHTTPS_PORT:-443}:443",
      "127.0.0.1:8081:8080"
    ],
    "stack": "routing"
  },
  {
    "name": "cloudflared",
    "image": "cloudflare/cloudflared:latest",
    "ports": [],
    "stack": "routing"
  }
]
```

## Environment Variables State

The following JSON outlines the configurable environment variables for the M3TAL system:

```json
[
  {
    "key": "API_PORT",
    "default": "5050"
  },
  {
    "key": "DASHBOARD_PORT",
    "default": "8082"
  },
  {
    "key": "DASHBOARD_EXPOSE_MODE",
    "default": "local"
  },
  {
    "key": "HTTP_PORT",
    "default": "8080"
  },
  {
    "key": "STATE_DIR",
    "default": "./state"
  },
  {
    "key": "LOG_LEVEL",
    "default": "info"
  },
  {
    "key": "DASHBOARD_SECRET",
    "default": "change_me_immediately"
  },
  {
    "key": "API_TOKEN",
    "default": "change_me_api_token"
  },
  {
    "key": "ADMIN_PASSWORD",
    "default": "admin_pass"
  },
  {
    "key": "NETWORK_NAME",
    "default": "m3tal"
  },
  {
    "key": "LOCAL_IP",
    "default": "127.0.0.1"
  },
  {
    "key": "DOMAIN",
    "default": "localhost"
  },
  {
    "key": "VPN_USER",
    "default": "user"
  },
  {
    "key": "VPN_PASSWORD",
    "default": "password"
  },
  {
    "key": "BASE_STORAGE_PATH",
    "default": "./data"
  },
  {
    "key": "MEDIA_PATH",
    "default": "./data/media"
  },
  {
    "key": "CONFIG_PATH",
    "default": "./data/config"
  },
  {
    "key": "DOWNLOADS_PATH",
    "default": "./data/downloads"
  },
  {
    "key": "PUID",
    "default": "1000"
  },
  {
    "key": "PGID",
    "default": "1000"
  },
  {
    "key": "TZ",
    "default": "America/Denver"
  },
  {
    "key": "TRAEFIK_WEB_PORT",
    "default": "80"
  },
  {
    "key": "TRAEFIK_WEBHTTPS_PORT",
    "default": "443"
  },
  {
    "key": "TRAEFIK_DASHBOARD_PORT",
    "default": "8080"
  },
  {
    "key": "DEBUG_MODE",
    "default": "false"
  },
  {
    "key": "METRICS_ENABLED",
    "default": "true"
  }
]
```