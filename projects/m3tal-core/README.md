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

## Configuration

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed using the `m3tal config wizard` command or individual `m3tal config set KEY value` commands.

## Filesystem Contract

The following paths are critical to M3TAL's operation and data management:

| Path                     | Purpose                                                                                              |
| :----------------------- | :--------------------------------------------------------------------------------------------------- |
| `/etc/m3tal/.env`        | Primary configuration file. Managed by `m3tal config wizard`.                                        |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon.                                             |
| `/opt/m3tal/stack/`      | Canonical stack directory. Contains compose files and Traefik dynamic configuration.                 |
| `/docker`                | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations.               |
| `/docker/users.json`     | Dashboard credential store. Managed by `m3tal dashpass`.                                             |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker` directory is a user-facing symlink alias pointing to `/opt/m3tal/stack/`. This means that `/opt/m3tal/stack/` is the canonical source of truth directory where all stack compose files and Traefik dynamic configuration reside.

### Deploying Stacks

To deploy any Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value` as needed.
3. Execute `m3tal up` to start all defined stacks.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.

## Dashboard Access

The M3TAL dashboard can be accessed in two primary modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable within `/etc/m3tal/.env`:

### Local Mode (Default)

- **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
- **Operation:** This mode utilizes an override file (`m3tal-compose.local.yml`) to directly bind the dashboard's internal port to a host port.
- **Access:** You will access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`. A new user performing a default installation will access the dashboard directly via port 8082. No Traefik is required for this mode.
- **Use Case:** Ideal for LAN-only setups, initial testing, and users who prefer direct port access.

### Traefik Mode

- **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
- **Operation:** This mode configures Traefik to route traffic to the dashboard container based on a domain name. Traefik must be running for this mode to function.
- **Access:** You will access the dashboard via `http://dash.DOMAIN`, where `DOMAIN` is configured in your `/etc/m3tal/.env` file.
- **Use Case:** Suitable for domain-based setups and environments where multiple services are managed behind a single reverse proxy.

## Traefik Gateway

Traefik acts as the reverse proxy and service discovery mechanism for M3TAL. It is deployed as a container via `routing-compose.yml`.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik utilizes a file provider to load dynamic configuration files from `/docker/dynamic/`. These files allow for granular routing rules and are hot-reloaded without restarting Traefik.

### Routing Examples

- **API Daemon Routing:** The dynamic configuration file `dynamic/api.yml` (located within `/opt/m3tal/stack/dynamic/`) routes requests from `api.DOMAIN` to the M3TAL API daemon, which listens on host-local port `8080`, via the internal `host.docker.internal` hostname.

  ```yaml
  # /opt/m3tal/stack/dynamic/api.yml
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

- **Dashboard Routing (Traefik Mode):** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik routes requests for `dash.DOMAIN` to the dashboard container. This is configured via labels in `m3tal-compose.traefik.yml`.

- **Custom User Service Exposure:** To expose a custom user service via Traefik labels, add the appropriate labels to its Docker Compose definition within `/docker/`.

  ```yaml
  # Example: /docker/my-app-compose.yml
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

### Traefik Static Configuration

Traefik's static configuration is defined in `traefik.yml` (managed within `/opt/m3tal/stack/`).

```yaml
# /opt/m3tal/stack/traefik.yml
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

## Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using the following commands:

- `systemctl status m3tal-api.service`: Check the status of the API daemon.
- `systemctl restart m3tal-api.service`: Restart the API daemon.
- `journalctl -u m3tal-api.service -f`: View the logs of the API daemon in real-time.

## Quick Demo

This section provides a quick demonstration of M3TAL's core functionalities:

- **Starting the Dashboard:**
  To start only the dashboard container specifically, use the following command:
  ```bash
  m3tal dash up
  ```
  This command orchestrates the deployment of the dashboard container, respecting the `DASHBOARD_EXPOSE_MODE` setting in your `.env` file.

- **Deploying All Stacks:**
  The `m3tal up` command is used to orchestrate and deploy all defined stacks in the `/docker/` directory. This includes the dashboard (if not already running independently) and any other user-defined Docker Compose files present in that directory.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                                          | Access                                                    | Description                                                                 |
| :--- | :----------------------------------------------- | :-------------------------------------------------------- | :-------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point                         | Public                                                    | The public-facing HTTP port for services exposed via Traefik.               |
| 8080 | M3TAL API daemon (Go)                            | Host-local                                                | The internal port the M3TAL API daemon listens on.                          |
| 8081 | Traefik dashboard                                | Host-local only                                           | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard                                  | Direct port (local mode) or via Traefik (traefik mode)    | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.