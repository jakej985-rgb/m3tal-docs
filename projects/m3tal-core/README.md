# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is installed via APT.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Filesystem Contract

The M3TAL system adheres to the following filesystem structure:

| Path                      | Purpose                                                               |
| :------------------------ | :-------------------------------------------------------------------- |
| `/etc/m3tal/.env`         | Primary configuration file. Managed by `m3tal config wizard`.         |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon.              |
| `/opt/m3tal/stack/`       | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                 | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`      | Dashboard credential store. Managed by `m3tal dashpass`.              |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory acts as a user-facing alias that points to the canonical stack source of truth, located at `/opt/m3tal/stack/`. All stack operations, including the deployment of user-defined stacks, should be performed by placing files within the `/docker/` directory.

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.

## Dashboard Access

The M3TAL Dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### Local Mode (Default)
- **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
- **Behavior:** The dashboard container is directly bound to a host port. For a new user performing a default installation, this means the dashboard is accessible directly via port 8082.
- **Access:** `http://HOST_IP:8082` or `http://localhost:8082`.
- **Requirements:** No Traefik is required for access in this mode. This mode is ideal for LAN-only setups, first-time users, and local testing.

### Traefik Mode
- **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Behavior:** Traefik is used to route traffic to the dashboard container based on domain name.
- **Access:** `http://dash.DOMAIN` (requires Traefik to be running and accessible on port 80).
- **Requirements:** Traefik must be running and configured to route to the dashboard. This mode is suited for domain-based setups and managing multiple services behind a reverse proxy.

## Traefik Gateway

Traefik is deployed via `routing-compose.yml` and functions as the primary reverse proxy. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is configured with a static configuration file (`traefik.yml`) and a dynamic configuration directory (`/docker/dynamic/`). The static configuration defines entry points (e.g., port 80 for HTTP) and providers for service discovery. The dynamic configuration files, such as `dynamic/api.yml`, are used for more specific routing rules and can be hot-reloaded by Traefik.

For instance, `dynamic/api.yml` is used to route requests to the M3TAL API daemon:

```yaml
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
This configuration routes requests made to `api.DOMAIN` to the Go API daemon listening on the host-local port `8080`.

The dashboard itself is routed via Traefik labels when `DASHBOARD_EXPOSE_MODE=traefik`, using the rule `Host(dash.${DOMAIN})` and targeting the dashboard container on port `8082`.

To expose a custom user service via Traefik labels, you would add appropriate labels to its service definition within its Docker Compose file. For example, to expose a hypothetical `my-app` service on `app.DOMAIN`:

```yaml
# Example: my-app-compose.yml
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

The M3TAL API daemon (`m3tal-api.service`) is managed by `systemd`. The following commands can be used to manage its lifecycle:

-   **Status:** `systemctl status m3tal-api`
-   **Restart:** `systemctl restart m3tal-api`
-   **View Logs:** `journalctl -u m3tal-api -f`

## Quick Demo

### Starting the Dashboard Specifically

To start only the M3TAL dashboard container:

```bash
m3tal dash up
```
This command specifically manages the dashboard container, downloading the necessary compose files and applying the `DASHBOARD_EXPOSE_MODE` setting.

### Deploying All Stacks

The `m3tal up` command orchestrates and deploys all stacks defined in the `/docker/` directory. This includes the M3TAL control plane, Traefik, and any user-defined compose files present in `/docker/`.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                                 | Access                                                     | Description                                                                                             |
| :--- | :-------------------------------------- | :--------------------------------------------------------- | :------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point                | Public (traefik mode)                                      | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)                   | Host-local                                                 | The internal port the M3TAL API daemon listens on.                                                      |
| 8081 | Traefik dashboard                       | Host-local only                                            | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard                         | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Docker Services State

This JSON describes the core Docker services managed by M3TAL:

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

This JSON lists the primary environment variables managed by M3TAL and their default values:

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