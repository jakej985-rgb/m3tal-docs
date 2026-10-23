# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

### APT Installation

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract

| Path                       | Purpose                                              |
|----------------------------|------------------------------------------------------|
| `/etc/m3tal/.env`          | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`  | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`        | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                  | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`       | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/docker/` directory is a symbolic link to `/opt/m3tal/stack/`, which serves as the canonical source of truth for all stack definition files.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any necessary environment variables for your stack are configured in `/etc/m3tal/.env` using `m3tal config wizard` or `m3tal config set KEY value`.
3. Execute `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`:

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the default mode. When `DASHBOARD_EXPOSE_MODE` is set to `local` (or unset, as `local` is the default value), the dashboard container is directly exposed on the host machine's network interface. A new user performing a default installation will access the dashboard directly via port 8082.

*   **Access:** `http://HOST_IP:8082` or `http://localhost:8082`
*   **Requirement:** No Traefik is required for this mode.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

In this mode, the dashboard is exposed via the Traefik reverse proxy. Traefik will route traffic to the dashboard based on a domain name.

*   **Access:** `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`)
*   **Requirement:** Traefik must be running and configured to route to the dashboard.

## Traefik Gateway

Traefik is deployed to manage external access to services. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik uses dynamic configuration files located in `/docker/dynamic/` to route requests. For instance, `dynamic/api.yml` is used to route requests for `api.DOMAIN` to the M3TAL API daemon, which listens on host-local port `8080` via `http://host.docker.internal:8080`. The dashboard is routed via `dash.DOMAIN` when `DASHBOARD_EXPOSE_MODE=traefik` through labels defined in `m3tal-compose.traefik.yml`.

Here is a concrete YAML example of how to expose a custom user service, `my-app`, via Traefik labels in a hypothetical `my-app-compose.yml`:

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

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Quick Demo

To quickly start only the dashboard container:

```bash
m3tal dash up
```

This command specifically manages the dashboard container, pulling the latest compose files and applying the appropriate override based on your `/etc/m3tal/.env` configuration.

The `m3tal up` command orchestrates and deploys all other stacks defined in the `/docker/` directory, including any user-defined compose files.

## Service Management

The M3TAL API daemon is managed by systemd. The following commands can be used to control its lifecycle:

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                           | Access                                           | Description                                                                                     |
|------|-----------------------------------|--------------------------------------------------|-------------------------------------------------------------------------------------------------|
| 80   | Traefik HTTP entry point          | Public (when Traefik is enabled)                 | The public-facing HTTP port for services exposed via Traefik.                                   |
| 8080 | M3TAL API daemon (Go)             | Host-local                                       | The internal port the M3TAL API daemon listens on.                                              |
| 8081 | Traefik dashboard                 | Host-local only                                  | The internal Traefik dashboard port, accessible only from the host machine.                     |
| 8082 | M3TAL Dashboard                   | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Docker Services State

The following outlines the known Docker services managed by M3TAL:

```json
[
  {
    "name": "ollama",
    "image": "ollama/ollama:latest",
    "ports": [
      "11434:11434"
    ],
    "stack": "ai"
  },
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

The following is a list of known environment variables and their default values used by M3TAL:

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