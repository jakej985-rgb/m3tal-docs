# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is distributed via APT.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## File System Contract

The M3TAL system utilizes the following key file paths:

| Path                       | Purpose                                                         |
|----------------------------|-----------------------------------------------------------------|
| `/etc/m3tal/.env`          | Primary configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`  | SQLite state database. Auto-created by the API daemon.        |
| `/opt/m3tal/stack/`        | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                  | Symlink → `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`       | Dashboard credential store. Managed by `m3tal dashpass`.        |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a user-facing symlink alias for all stack operations, pointing to `/opt/m3tal/stack/`, which is the canonical source of truth directory where all stack files reside.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env` using `m3tal config wizard` or `m3tal config set KEY value`.
3. Run `m3tal up` to deploy the new stack along with all existing M3TAL-managed stacks.

## Dashboard Access

The M3TAL Dashboard has two access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`:

### Local Mode (Default)

- **Setting:** `DASHBOARD_EXPOSE_MODE=local`
- **Description:** In this mode, the dashboard container is directly exposed on a host port. This mode does not require Traefik to be running.
- **Access:** A new user performing a default installation will access the dashboard directly via port 8082. Access is available at `http://HOST_IP:8082` or `http://localhost:8082`.

### Traefik Mode

- **Setting:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Description:** In this mode, Traefik is used to route traffic to the dashboard container based on a domain name. This mode requires Traefik to be running.
- **Access:** Access is available via `http://dash.DOMAIN`, where `DOMAIN` is configured in `/etc/m3tal/.env`.

## Traefik Gateway

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik is deployed via `routing-compose.yml` and binds port 80 on the host as its HTTP entry point. It utilizes Docker labels for service discovery and a file provider for dynamic routing configuration, which allows for hot-reloading of routing rules without restarting Traefik.

Dynamic configuration files, such as `dynamic/api.yml`, are used to route requests to services listening on host-local ports. For example, `api.DOMAIN` is routed to the Go API daemon listening on host-local port `8080` via `http://host.docker.internal:8080`.

The `dash.DOMAIN` route is configured to direct traffic to the M3TAL Dashboard container when `DASHBOARD_EXPOSE_MODE` is set to `traefik`.

### Example: Exposing a Custom User Service via Traefik

To expose a custom user service (e.g., a hypothetical `my-app` running on port 80 within its container) using Traefik labels, you would add the following to your custom Docker Compose file (e.g., `my-app-compose.yml`) placed in the `/docker/` directory:

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

The M3TAL API daemon is managed by systemd. The service name is `m3tal-api.service`.

You can manage this service using the following `systemctl` commands:

- **Check status:** `systemctl status m3tal-api`
- **Restart the service:** `systemctl restart m3tal-api`
- **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

### Starting the Dashboard

To specifically start only the M3TAL Dashboard container:

```bash
m3tal dash up
```

This command is useful for quickly bringing up the dashboard for immediate access, especially when testing configuration changes or ensuring the dashboard is operational.

### Deploying All Stacks

The `m3tal up` command orchestrates and deploys all Docker Compose stacks defined in the `/docker/` directory. This includes the M3TAL Dashboard, Traefik, Cloudflared (if configured), and any additional user-defined stacks that have been added to the `/docker/` directory.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                                  | Access                                                      | Description                                                                                             |
|------|------------------------------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| 80   | Traefik HTTP entry point                 | Public (when Traefik is in use)                             | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)                    | Host-local                                                  | The internal port the M3TAL API daemon listens on. Routable via Traefik using `api.DOMAIN`.         |
| 8081 | Traefik dashboard                        | Host-local only                                             | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard                          | Direct port (local mode) or via Traefik (traefik mode)      | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).