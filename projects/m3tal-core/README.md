# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

Use the following APT commands to install the M3TAL CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary M3TAL configuration is managed via the `/etc/m3tal/.env` file. This file can be populated or modified using the `m3tal config wizard` command or individual `m3tal config set KEY value` commands.

## Filesystem Contract

The following paths are critical to M3TAL's operation:

| Path                       | Purpose                                                    |
| :------------------------- | :--------------------------------------------------------- |
| `/etc/m3tal/.env`          | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db`  | SQLite state database. Auto-created by the API daemon.   |
| `/opt/m3tal/stack/`        | Canonical stack directory. Contains compose files and Traefik config. |
| `/docker`                  | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`       | Dashboard credential store. Managed by `m3tal dashpass`.   |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack definition files reside. The `/docker` directory is a user-facing symlink alias that points to `/opt/m3tal/stack/`, consolidating all stack operations under a single, intuitive path for the user.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your new stack are configured in `/etc/m3tal/.env`.
3. Run `m3tal up` to deploy the new stack along with all existing M3TAL stacks.

## Dashboard Access

The M3TAL Dashboard has two distinct access modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable in `/etc/m3tal/.env`.

### Local Mode (Default)

- **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
- **Mechanism:** This mode utilizes an override file (`m3tal-compose.local.yml`) that adds a direct port binding, exposing the dashboard container at `${DASHBOARD_PORT:-8082}` on the host machine.
- **Access:** A new user performing a default installation will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`. This direct access via port 8082 is the default behavior.
- **Requirements:** No additional reverse proxy (like Traefik) is required for this mode. It is suitable for LAN-only setups, first-time users, and local testing environments.

### Traefik Mode

- **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Mechanism:** This mode employs an override file (`m3tal-compose.traefik.yml`) that adds Traefik labels to the dashboard service. Traefik, if running, will then route incoming requests for `dash.${DOMAIN}` to the dashboard container listening on port 8082.
- **Access:** Access is provided through domain routing via Traefik at `http://dash.DOMAIN`.
- **Requirements:** Traefik must be running and configured to route traffic for the specified domain. This mode is ideal for domain-based setups and scenarios where multiple services are managed behind a single reverse proxy.

## Traefik Gateway

Traefik is deployed as a reverse proxy container and is responsible for routing external traffic to various services managed by M3TAL. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik utilizes a file provider to load dynamic configuration files from the `/etc/traefik/dynamic/` directory, allowing for hot-reloading of routing rules without service restarts.

### Dynamic Configuration

Dynamic configuration files, such as `dynamic/api.yml`, are used to route requests to services listening on host-local ports. For example, the `api.DOMAIN` is routed to the Go API daemon (which listens on host-local port `8080`) via the following configuration:

```yaml
http:
  routers:
    api:
      rule: "Host(`api.${DOMAIN}`)"
      service: api
      entrypoints:
        - web

  services:
    api:
      loadBalancer:
        servers:
          - url: "http://host.docker.internal:8080"
```

Similarly, `dash.DOMAIN` routes to the dashboard container when `DASHBOARD_EXPOSE_MODE` is set to `traefik`.

### Exposing Custom User Services

To expose a custom user service via Traefik labels, define the appropriate Traefik labels within your service's Docker Compose definition. For instance, to expose a hypothetical `my-app` service running Nginx on port 80 to the domain `app.DOMAIN`:

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

The M3TAL API daemon (`m3tal-api.service`) is managed by `systemd`. The following commands can be used for service management:

- **Check status:** `systemctl status m3tal-api.service`
- **Restart:** `systemctl restart m3tal-api.service`
- **View logs:** `journalctl -u m3tal-api.service -f`

## Quick Demo

This section outlines basic commands for interacting with M3TAL components.

### Starting the Dashboard

To start only the dashboard container:

```bash
m3tal dash up
```

This command specifically targets the dashboard container, pulling its latest image and starting it according to the `DASHBOARD_EXPOSE_MODE` setting in your `.env` file.

### Orchestrating All Stacks

To orchestrate and deploy all defined stacks, including any custom user stacks present in the `/docker/` directory:

```bash
m3tal up
```

This command is a wrapper around `docker compose` and will process all `*-compose.yml` files found within the `/docker/` directory, bringing all services online.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                                   | Access                                      | Description                                                                         |
|------|-------------------------------------------|---------------------------------------------|-------------------------------------------------------------------------------------|
| 80   | Traefik HTTP entry point                  | Public (traefik mode)                       | The public-facing HTTP port for services exposed via Traefik.                       |
| 8080 | M3TAL API daemon (Go)                     | Host-local                                  | The internal port the M3TAL API daemon listens on.                                  |
| 8081 | Traefik dashboard                         | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine.         |
| 8082 | M3TAL Dashboard                           | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).
