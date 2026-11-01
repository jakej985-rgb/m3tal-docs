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

| Path                         | Purpose                                                              |
|------------------------------|----------------------------------------------------------------------|
| `/etc/m3tal/.env`            | Primary configuration file. Managed by `m3tal config wizard`.        |
| `/var/lib/m3tal/state.db`    | SQLite state database. Auto-created by the API daemon.             |
| `/opt/m3tal/stack/`          | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                    | Symlink → `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`         | Dashboard credential store. Managed by `m3tal dashpass`.             |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/docker/` directory is a user-facing symlink alias for all stack operations, pointing to the canonical source of truth directory `/opt/m3tal/stack/` where all stack files reside.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env` using `m3tal config wizard` or `m3tal config set KEY value`.
3. Run `m3tal up` to deploy all stacks in the `/docker/` directory.

## Dashboard Access

The dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the default mode for a new installation.
- **Access Method**: Direct port binding. The dashboard container is exposed directly on `http://HOST_IP:8082` or `http://localhost:8082`.
- **Dependencies**: No Traefik is required for direct access. This mode is suitable for LAN-only setups, first-time users, and local testing.
- **Configuration**: Achieved by using the `m3tal-compose.local.yml` override, which adds a port mapping of `${DASHBOARD_PORT:-8082}:8082` to the `m3tal-dashboard` service.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

This mode leverages Traefik for routing.
- **Access Method**: Domain routing. The dashboard is accessed via `http://dash.DOMAIN`.
- **Dependencies**: Traefik must be running and configured to route traffic.
- **Configuration**: Achieved by using the `m3tal-compose.traefik.yml` override, which adds Traefik labels to the `m3tal-dashboard` service. These labels enable Traefik to route requests for `dash.${DOMAIN}` to the dashboard container on port 8082.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and acts as the primary reverse proxy. It binds port 80 on the host as its HTTP entry point and can also be configured for HTTPS on port 443. Traefik discovers services via Docker labels and utilizes a file provider for dynamic routing configuration, allowing hot-reloading of configurations from the `/docker/dynamic/` directory.

### Dynamic Configuration

Dynamic configuration files, such as `dynamic/api.yml`, are used to route requests to services listening on host-local ports. For instance, requests to `api.DOMAIN` are routed to the Go API daemon listening on host-local port `8080` via `http://host.docker.internal:8080`.

The `dash.DOMAIN` route is specifically configured via Traefik labels within the dashboard's compose file when `DASHBOARD_EXPOSE_MODE=traefik`.

### Exposing a Custom User Service

To expose a custom user service via Traefik labels, you would add Traefik-specific labels to the service definition within its Docker Compose file. For example, a hypothetical `my-app-compose.yml` might look like this:

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

## Service Management — systemd

The M3TAL API daemon is managed by systemd as `m3tal-api.service`. You can control and monitor it using the following commands:

- **Check status**: `systemctl status m3tal-api`
- **Restart service**: `systemctl restart m3tal-api`
- **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

### Starting the Dashboard

To start only the dashboard container, use the following command:

```bash
m3tal dash up
```

This command specifically manages the dashboard container, pulling the necessary compose files and applying the selected `DASHBOARD_EXPOSE_MODE`.

### Deploying All Stacks

The `m3tal up` command orchestrates and deploys all defined stacks. It processes all `*-compose.yml` files found within the `/docker/` directory, including any custom user-defined compose files, and brings them online.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                            | Access                                        | Description                                                                                   |
|------|------------------------------------|-----------------------------------------------|-----------------------------------------------------------------------------------------------|
| 80   | Traefik HTTP entry point           | Public                                        | The public-facing HTTP port for services exposed via Traefik.                                 |
| 8080 | M3TAL API daemon (Go)              | Host-local                                    | The internal port the M3TAL API daemon listens on.                                            |
| 8081 | Traefik dashboard                  | Host-local only                               | The internal Traefik dashboard port, accessible only from the host machine.                   |
| 8082 | M3TAL Dashboard                    | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

---

## M3TAL System Architecture Details

### Components

*   **CLI binary** (`/usr/bin/m3tal`): Unified Go binary installed via APT. Single entrypoint for all operations.
*   **API daemon** (`m3tal-api.service`): Go binary running as a systemd service on port 8080. Manages Docker, state DB, and API routes.
*   **Dashboard container** (`m3tal-dashboard`): Python/Flask container running internally on port 8082. Communicates with the API daemon at `http://host.docker.internal:5050`.
*   **Traefik gateway** (`routing-compose.yml`): Reverse proxy container exposing services by domain name on port 80. Uses file provider for dynamic routing.
*   **Cloudflared** (`routing-compose.yml`): Optional Cloudflare tunnel container for zero-config internet access.

### Docker Services State

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

### Environment Variables State

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