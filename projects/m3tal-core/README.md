# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is installed via APT. Follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` command or individual `m3tal config set KEY value` commands.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a symbolic link to `/opt/m3tal/stack/`. `/opt/m3tal/stack/` is the canonical source of truth for all stack files, including base compose files, override files, and user-added stack definitions. All user-facing stack operations should target the `/docker/` path.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any necessary environment variables for your new stack are defined in `/etc/m3tal/.env` or set using `m3tal config set`.
3. Run `m3tal up` to deploy all stacks defined in the `/docker/` directory.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Dashboard Access

The M3TAL Dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### Local Mode (Default)

-   **Setting:** `DASHBOARD_EXPOSE_MODE=local`
-   **Behavior:** The dashboard container is directly port-bound to the host machine.
-   **Access:** You will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`. A new user performing a default installation will access the dashboard directly via port 8082.
-   **Requirements:** No Traefik is required for this mode.

### Traefik Mode

-   **Setting:** `DASHBOARD_EXPOSE_MODE=traefik`
-   **Behavior:** Traefik routes traffic to the dashboard container based on domain name labels.
-   **Access:** Access the dashboard via `http://dash.DOMAIN` (where `DOMAIN` is configured in `/etc/m3tal/.env`).
-   **Requirements:** Traefik must be running and configured to route traffic to the dashboard.

## Traefik Gateway

Traefik acts as the primary reverse proxy for exposing services managed by M3TAL. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

### Routing Architecture

Traefik is deployed via `routing-compose.yml` and utilizes both Docker discovery and a file provider for dynamic configuration.

*   **Host Port Binding:** Traefik binds port `80` (and `443` if HTTPS is configured) on the host machine as its HTTP entry point.
*   **Docker Discovery:** Traefik monitors Docker events to discover containers. Services are only exposed if they have specific `traefik.*` labels.
*   **File Provider:** Traefik loads dynamic configuration from files within the `/docker/dynamic/` directory. This allows for fine-grained control over routing rules and is essential for routing requests to services listening on host-local ports.

    For example, a dynamic configuration file like `dynamic/api.yml` can be used to route requests for `api.DOMAIN` to the M3TAL API daemon running on host-local port `8080`:

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

*   **Dashboard Routing:** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests to `dash.DOMAIN` to the dashboard container on port `8082`. This is configured via labels within the dashboard's compose override file.

### Exposing Custom Services via Traefik

To expose a custom user-defined service through Traefik, add the appropriate `traefik.*` labels to its service definition in your Docker Compose file. For instance, to expose a hypothetical `my-app` service listening on port `80` to `app.DOMAIN`:

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

Ensure your custom service is also connected to the `proxy` network if it's intended to be accessible via Traefik.

## Service Management

The M3TAL API daemon is managed by `systemd` as the `m3tal-api.service`. The following commands can be used to manage this service:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart service:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To start the M3TAL dashboard container specifically:

```bash
m3tal dash up
```

This command will fetch the necessary compose files and apply the override based on your `DASHBOARD_EXPOSE_MODE` setting.

To orchestrate and deploy all other stacks in the `/docker/` directory, including any user-defined compose files, use:

```bash
m3tal up
```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port   | Service                     | Access                                       | Description                                                                      |
| :----- | :-------------------------- | :------------------------------------------- | :------------------------------------------------------------------------------- |
| 80     | Traefik HTTP entry point    | Public                                       | The public-facing HTTP port for services exposed via Traefik.                      |
| 8080   | M3TAL API daemon (Go)       | Host-local                                   | The internal port the M3TAL API daemon listens on.                               |
| 8081   | Traefik dashboard           | Host-local only                              | The internal Traefik dashboard port, accessible only from the host machine.      |
| 8082   | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Filesystem Contract

| Path                       | Purpose                                                         |
| :------------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`          | Primary configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`  | SQLite state database. Auto-created by the API daemon.        |
| `/opt/m3tal/stack/`        | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                  | Symlink → `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`       | Dashboard credential store. Managed by `m3tal dashpass`.        |

## Docker Services State

This JSON represents the currently deployed Docker services managed by M3TAL:

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

This JSON lists the environment variables managed by M3TAL, including their keys and default values:

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