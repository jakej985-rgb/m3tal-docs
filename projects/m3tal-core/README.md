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

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed through the `m3tal config wizard` command or by directly setting environment variables using `m3tal config set KEY value`.

Common environment variables include:

*   `API_PORT`: The port for the M3TAL API daemon.
*   `DASHBOARD_PORT`: The internal port for the M3TAL Dashboard container.
*   `DASHBOARD_EXPOSE_MODE`: Controls how the dashboard is accessed (`local` or `traefik`).
*   `DOMAIN`: The primary domain used for Traefik routing.
*   `PUID`, `PGID`: User and group IDs for container processes.
*   `TZ`: Timezone for containers.

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory acts as the user-facing alias for stack operations, symlinking to `/opt/m3tal/stack/`, which is the canonical source of truth for all stack files.

**To deploy a new Docker Compose stack:**
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` for this purpose.
3.  Run `m3tal up` to deploy all stacks, including your newly added one.

## Dashboard Access

The M3TAL dashboard can be accessed in two primary modes, determined by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the **default mode** for new installations.
- **Access:** The dashboard is directly accessible via port binding at `http://HOST_IP:8082` or `http://localhost:8082`. No Traefik is required for this mode.
- **Configuration:** This mode utilizes an override file (`m3tal-compose.local.yml`) that adds a direct port binding for the dashboard container.
- **Use Case:** Ideal for LAN-only setups, initial local testing, and users who do not require domain-based access. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

- **Access:** The dashboard is accessed via domain routing at `http://dash.DOMAIN` through the Traefik gateway. Traefik must be running for this mode to function.
- **Configuration:** This mode utilizes an override file (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard service, enabling Traefik to route traffic to it.
- **Use Case:** Suitable for domain-based setups and for users who have multiple services managed by M3TAL behind a single reverse proxy.

## Traefik Gateway

Traefik acts as the primary reverse proxy and service discovery mechanism for M3TAL when `DASHBOARD_EXPOSE_MODE=traefik` or when exposing other user-defined services.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

**Key functionalities of Traefik:**

*   **Service Discovery:** Reads Docker labels to identify and configure routes for services.
*   **Dynamic Routing:** Utilizes a file provider to load dynamic configuration files from `/docker/dynamic/`, enabling hot-reloading of routing rules without restarting Traefik.

**Routing Examples:**

*   **API Daemon Routing:** The `api.DOMAIN` requests are routed to the Go API daemon listening on host-local port `8080`. This is configured via `dynamic/api.yml`.

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

*   **Dashboard Routing (Traefik Mode):** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests to `dash.DOMAIN` to the dashboard container.

*   **Exposing Custom User Services:** To expose a custom user service via Traefik labels, add the appropriate labels to your service definition within your user-defined Docker Compose file (e.g., `my-app-compose.yml` placed in `/docker/`).

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
    In this example, `app.DOMAIN` will be routed to the `my-app` service running on port `80`. The `networks: - proxy` line ensures the service is on the network Traefik is monitoring.

**Traefik Configuration:**

*   **Static Configuration (traefik.yml):** Defines entry points and providers.
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
*   **Dynamic Configuration:** Stored in `/docker/dynamic/` and managed by Traefik.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Service Management — systemd

The M3TAL API daemon runs as a systemd service named `m3tal-api.service`. You can manage it using the following commands:

*   **Check status:** `systemctl status m3tal-api`
*   **Restart:** `systemctl restart m3tal-api`
*   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly start the M3TAL dashboard in its default `local` mode:

1.  Run: `m3tal dash up`
    This command specifically orchestrates and deploys only the dashboard container, making it accessible directly via port 8082.

To deploy all available M3TAL stacks, including any custom stacks you have added to the `/docker/` directory:

1.  Run: `m3tal up`
    This command orchestrates and deploys all other stacks in the directory, including any user-defined compose files.

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

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |