# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is installed via APT. Follow these steps to install M3TAL on your system:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` command or can be updated using `m3tal config set KEY value`.

Key environment variables include:

| Key                    | Default         | Description                                        |
|------------------------|-----------------|----------------------------------------------------|
| `API_PORT`             | `5050`          | Port for the M3TAL API daemon.                     |
| `DASHBOARD_PORT`       | `8082`          | Port for the M3TAL Dashboard container.            |
| `DASHBOARD_EXPOSE_MODE`| `local`         | Controls dashboard access (`local` or `traefik`). |
| `HTTP_PORT`            | `8080`          | Primary HTTP port, often used for internal services. |
| `STATE_DIR`            | `./state`       | Directory for the SQLite state database.           |
| `LOG_LEVEL`            | `info`          | Logging level for M3TAL services.                  |
| `DASHBOARD_SECRET`     | `change_me_immediately` | Secret for dashboard authentication.           |
| `API_TOKEN`            | `change_me_api_token` | Token for API authentication.                      |
| `ADMIN_PASSWORD`       | `admin_pass`    | Password for the administrator user.               |
| `NETWORK_NAME`         | `m3tal`         | Name of the Docker network used by M3TAL.          |
| `LOCAL_IP`             | `127.0.0.1`     | Local IP address for host communication.           |
| `DOMAIN`               | `localhost`     | Default domain for Traefik routing.                |
| `BASE_STORAGE_PATH`    | `./data`        | Base path for M3TAL data storage.                  |
| `CONFIG_PATH`          | `./data/config` | Path for M3TAL configuration files.                |
| `PUID`                 | `1000`          | User ID for running Docker containers.             |
| `PGID`                 | `1000`          | Group ID for running Docker containers.            |
| `TZ`                   | `America/Denver`| Timezone for M3TAL services.                       |
| `TRAEFIK_WEB_PORT`     | `80`            | Host port for Traefik's HTTP entry point.          |
| `TRAEFIK_WEBHTTPS_PORT`| `443`           | Host port for Traefik's HTTPS entry point.         |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory acts as a user-facing symlink alias for all stack operations, pointing to the canonical source of truth located at `/opt/m3tal/stack/`. All Docker Compose stack files, including user-defined ones and M3TAL's internal stack definitions, should reside in `/opt/m3tal/stack/`.

**To deploy a new Docker Compose stack:**
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your new stack are defined in `/etc/m3tal/.env`. You can use `m3tal config wizard` or `m3tal config set KEY value`.
3. Execute `m3tal up` to deploy your new stack along with all other managed stacks.

## Dashboard Access

The M3TAL dashboard has two primary access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### Local mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the **default** mode for a new installation.
- **Access:** Direct port binding at `http://HOST_IP:8082` or `http://localhost:8082`.
- **Requirements:** No Traefik is needed for this mode.
- **Description:** A new user performing a default installation will access the dashboard directly via port 8082, leveraging the default `DASHBOARD_EXPOSE_MODE=local` setting. This mode is ideal for LAN-only setups, first-time users, and local testing environments.

### Traefik mode (`DASHBOARD_EXPOSE_MODE=traefik`)

- **Access:** Domain routing at `http://dash.DOMAIN` via Traefik.
- **Requirements:** Traefik must be running and configured to route traffic.
- **Description:** When `DASHBOARD_EXPOSE_MODE` is set to `traefik`, the dashboard is exposed through Traefik. Traefik labels defined within the dashboard's Compose file will direct traffic for `dash.DOMAIN` to the dashboard container. This mode is suitable for domain-based setups and when managing multiple services behind a reverse proxy.

## Traefik Gateway

Traefik is deployed via `routing-compose.yml` and acts as the central reverse proxy. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik's static configuration is defined in `traefik.yml`:
```yaml
entryPoints:
  web:
    address: ":80"

providers:
  docker:
    exposedByDefault: false
    network: proxy
  file:
    directory: /etc/traefik/dynamic
    watch: true
```

**Dynamic Configuration:** Traefik utilizes dynamic configuration files located in `/docker/dynamic/` for routing requests to services. For instance, the following configuration in `dynamic/api.yml` routes requests for `api.DOMAIN` to the Go API daemon listening on host-local port `8080`:

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

The `dash.DOMAIN` routes to the dashboard container when `DASHBOARD_EXPOSE_MODE=traefik`.

**Exposing a Custom User Service via Traefik:**
To expose a custom user service (e.g., a hypothetical `my-app` service) via Traefik labels, you would include the following within your service definition in a user-provided Docker Compose file (e.g., `my-app-compose.yml` placed in `/docker/`):

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

To quickly start the dashboard container specifically, use:

```bash
m3tal dash up
```

This command will download the necessary Compose files and start the dashboard container based on your `DASHBOARD_EXPOSE_MODE` setting.

The `m3tal up` command orchestrates and deploys all other stacks in the `/docker/` directory, including any user-defined compose files, providing a comprehensive deployment of all managed services.

## Service Management

The M3TAL API daemon (`m3tal-api.service`) is managed by `systemd`. You can use the following commands to manage its lifecycle:

- **Status:** `systemctl status m3tal-api`
- **Restart:** `systemctl restart m3tal-api`
- **Logs:** `journalctl -u m3tal-api -f`

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Filesystem Contract

| Path                     | Purpose                                        |
|--------------------------|------------------------------------------------|
| `/etc/m3tal/.env`        | Primary configuration file, managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`| SQLite state database, auto-created by the API daemon. |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                | Symlink → `/opt/m3tal/stack/`. User-facing path for all stack operations. |
| `/docker/users.json`     | Dashboard credential store, managed by `m3tal dashpass`. |

## Docker Services State

The following JSON outlines the primary Docker services managed by M3TAL:

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