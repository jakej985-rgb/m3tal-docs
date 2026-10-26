# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary is distributed via APT. Follow these steps to install it:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed by the `m3tal config wizard` command, which guides users through setting essential environment variables. Individual variables can also be set or modified using `m3tal config set KEY value`.

## Filesystem Contract

| Path                          | Purpose                                           |
| :---------------------------- | :------------------------------------------------ |
| `/etc/m3tal/.env`             | Primary configuration file.                       |
| `/var/lib/m3tal/state.db`     | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/`           | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                     | Symlink → `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`          | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory acts as a user-facing alias for `/opt/m3tal/stack/`. All Docker Compose files that define M3TAL-managed stacks, including user-added stacks, should reside in `/opt/m3tal/stack/` and will be automatically symlinked and accessible via `/docker/`.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env`.
3. Run `m3tal up` to deploy the new stack along with all other managed stacks.

## Dashboard Access

The M3TAL dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the default mode for a new installation. In local mode, the dashboard container is directly exposed on the host machine's IP address.

*   **Access:** `http://HOST_IP:8082` (or `http://localhost:8082` if accessing from the host itself).
*   **Requirement:** No additional reverse proxy (like Traefik) is required for dashboard access.
*   **Use Case:** Ideal for local development, testing, or when direct access within a local network is sufficient.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

In Traefik mode, access to the dashboard is managed by the Traefik reverse proxy, allowing for domain-based routing.

*   **Access:** `http://dash.DOMAIN` (where `DOMAIN` is defined in your M3TAL configuration).
*   **Requirement:** Traefik must be running and configured to route traffic to the dashboard.
*   **Use Case:** Suitable for deployments where services are exposed via a domain name and managed by Traefik.

## Traefik Gateway

Traefik is deployed as a container to act as the primary reverse proxy for M3TAL services. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik's static configuration is defined in `traefik.yml` and includes entry points for HTTP (port 80) and HTTPS (port 443, if configured). It utilizes a file provider to load dynamic configuration files from the `/docker/dynamic/` directory, enabling hot-reloading of routing rules without restarting Traefik.

For example, dynamic configuration files in `/docker/dynamic/` can be used to route requests to services listening on host-local ports. The `api.yml` file demonstrates routing requests for `api.DOMAIN` to the M3TAL API daemon running on host-local port `8080` via `http://host.docker.internal:8080`.

```yaml
# Example: dynamic/api.yml
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

Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes `dash.DOMAIN` to the dashboard container on port 8082.

### Exposing Custom User Services via Traefik

To expose a custom user-defined service through Traefik, add the necessary Traefik labels to the service definition within your custom Docker Compose file.

Here's a concrete YAML example for a hypothetical `my-app-compose.yml` exposing a Nginx service:

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
networks:
  proxy:
    external: true
```

## Service Management

The M3TAL API daemon runs as a systemd service. The following commands can be used to manage it:

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo

### Starting the Dashboard

To start only the M3TAL dashboard container:

```bash
m3tal dash up
```

This command specifically targets the dashboard container and will deploy it according to your configured `DASHBOARD_EXPOSE_MODE`.

### Deploying All Stacks

To orchestrate and deploy all M3TAL-managed stacks, including any custom user-defined compose files placed in the `/docker/` directory:

```bash
m3tal up
```

This command invokes `docker compose` across all Compose files found in `/docker/`, ensuring all services are running.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port   | Service                      | Access                                      | Description                                                                                             |
| :----- | :--------------------------- | :------------------------------------------ | :------------------------------------------------------------------------------------------------------ |
| 80     | Traefik HTTP entry point     | Public                                      | The public-facing HTTP port for services exposed via Traefik (in `traefik` mode).                       |
| 8080   | M3TAL API daemon (Go)        | Host-local                                  | The internal port the M3TAL API daemon listens on. Accessible via `host.docker.internal:8080` from containers. |
| 8081   | Traefik dashboard            | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine via `http://127.0.0.1:8081`. |
| 8082   | M3TAL Dashboard              | Direct port (`local`) or via Traefik (`traefik`) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).