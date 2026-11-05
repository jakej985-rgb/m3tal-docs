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

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory acts as a user-facing symlink alias for all stack operations. It points to `/opt/m3tal/stack/`, which is the canonical source of truth directory where all stack files reside.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any necessary environment variables for your stack are configured. Use `m3tal config wizard` for interactive setup or `m3tal config set KEY value` for direct setting.
3. Run `m3tal up` to deploy your new stack alongside existing ones.

## Dashboard Access

The M3TAL Dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (Default)

When `DASHBOARD_EXPOSE_MODE=local` (which is the default setting for new installations), the dashboard is directly exposed on a host port.

- **Access:** `http://HOST_IP:8082` or `http://localhost:8082`
- **Description:** This mode bypasses Traefik for dashboard access. The dashboard container binds directly to port `8082` on the host machine. This is the simplest method for local testing or when running M3TAL on a home server without domain name resolution.

### Traefik Mode

When `DASHBOARD_EXPOSE_MODE=traefik`, the dashboard is accessible via Traefik's routing capabilities.

- **Access:** `http://dash.DOMAIN` (where `DOMAIN` is configured in your `.env` file).
- **Description:** This mode requires Traefik to be running. Traefik will route incoming requests for `dash.DOMAIN` to the dashboard container. This mode is suitable for more complex setups where multiple services are exposed through a single reverse proxy.

## Traefik Gateway

The Traefik gateway serves as a reverse proxy for services managed by M3TAL. It is deployed via `routing-compose.yml`.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik utilizes a file provider to load dynamic configuration from the `/docker/dynamic/` directory. This allows for hot-reloading of routing rules without restarting Traefik. For instance, dynamic configuration files like `dynamic/api.yml` are used to route requests to services listening on host-local ports. Specifically, `api.DOMAIN` is configured to route to the Go API daemon listening on host-local port `8080` via `http://host.docker.internal:8080`.

The dashboard is routed via Traefik labels when `DASHBOARD_EXPOSE_MODE=traefik`.

To expose a custom user service via Traefik labels, ensure your service definition in its Docker Compose file includes the necessary Traefik labels. For example, to expose a hypothetical Nginx service named `my-app` on `app.DOMAIN`:

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

## Service Management

The M3TAL API daemon is managed by systemd as `m3tal-api.service`. You can interact with it using the following `systemctl` commands:

- **Check status:** `systemctl status m3tal-api`
- **Restart:** `systemctl restart m3tal-api`
- **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

1.  **Start the Dashboard:** To specifically launch only the M3TAL dashboard container (useful for initial setup or quick checks), run:
    ```bash
    m3tal dash up
    ```
    This command will ensure the dashboard is running, respecting the `DASHBOARD_EXPOSE_MODE` setting.

2.  **Deploy All Stacks:** To orchestrate and deploy all other stacks defined in the `/docker/` directory (including any user-defined compose files), use:
    ```bash
    m3tal up
    ```
    This command is the primary method for bringing up the entire M3TAL environment.

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

| Path                | Purpose                                          |
|---------------------|--------------------------------------------------|
| `/etc/m3tal/.env`   | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`           | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |