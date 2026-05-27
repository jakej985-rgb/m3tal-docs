# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

Follow these steps to install the M3TAL CLI binary:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Configuration

The primary M3TAL configuration is managed through the `/etc/m3tal/.env` file. This file can be modified directly or by using the `m3tal config wizard` command for an interactive setup, and `m3tal config set KEY value` for direct key-value updates.

## Filesystem Contract

The following paths define M3TAL's filesystem structure:

| Path                       | Purpose                                                         |
| :------------------------- | :-------------------------------------------------------------- |
| `/etc/m3tal/.env`          | Primary configuration file, managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`  | SQLite state database, auto-created by the API daemon.        |
| `/opt/m3tal/stack/`        | Canonical stack directory containing compose files and Traefik config. |
| `/docker`                  | Symlink to `/opt/m3tal/stack/`. User-facing path for stack operations. |
| `/docker/users.json`       | Dashboard credential store, managed by `m3tal dashpass`.      |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/docker/` directory is a user-facing alias that symlinks to the canonical stack directory located at `/opt/m3tal/stack/`. All Docker Compose files and related configurations should reside within `/opt/m3tal/stack/` to ensure M3TAL can properly manage them.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any necessary environment variables for your new stack are configured in `/etc/m3tal/.env`. Use `m3tal config wizard` or `m3tal config set KEY value`.
3.  Run `m3tal up` to start all stacks, including your newly added one.

## Dashboard Access

The M3TAL dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` environment variable within `/etc/m3tal/.env`.

### Local Mode (`DASHBOARD_EXPOSE_MODE=local`)

This is the **default** mode for new installations.

-   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
-   **Access:** Direct port binding at `http://HOST_IP:8082` or `http://localhost:8082`.
-   **Description:** In this mode, the dashboard container is directly exposed on the specified host port (defaulting to 8082). No reverse proxy like Traefik is involved for dashboard access. This is ideal for local development, testing, or environments where direct IP access is sufficient. A new user performing a default installation will access the dashboard directly via port 8082.

### Traefik Mode (`DASHBOARD_EXPOSE_MODE=traefik`)

This mode leverages Traefik for domain-based routing.

-   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
-   **Access:** Domain routing at `http://dash.DOMAIN` via Traefik.
-   **Description:** When this mode is enabled, Traefik is configured to route traffic to the dashboard container based on a specific domain name. This requires Traefik to be running and properly configured to listen for requests on port 80 (or 443 for HTTPS).

## Traefik Gateway

Traefik acts as the primary reverse proxy, routing incoming traffic to various M3TAL services and user-defined stacks.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

The Traefik gateway uses a combination of Docker labels for service discovery and dynamic configuration files for specific routing rules.

-   **Traefik Static Configuration (`traefik.yml`):** Defines entry points and providers.
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
-   **Dynamic Configuration:** Located in `/docker/dynamic/` (which symlinks to `/opt/m3tal/stack/dynamic/`), these files provide granular routing. For example, `dynamic/api.yml` routes requests for `api.${DOMAIN}` to the M3TAL API daemon.
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
    This configuration explicitly routes requests for `api.${DOMAIN}` to the M3TAL API daemon, which listens on host-local port `8080`.

-   **Dashboard Routing (Traefik Mode):** When `DASHBOARD_EXPOSE_MODE=traefik`, Traefik labels on the `m3tal-dashboard` service direct traffic to `http://dash.DOMAIN`.

-   **Exposing Custom User Services:** To expose a custom user service via Traefik, add appropriate labels to its service definition in its Docker Compose file. For example, to expose a hypothetical `my-app` service:

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
    This configuration enables Traefik for the `my-app` service, defines a router that listens for requests to `app.DOMAIN`, sets the internal service port to 80, and ensures it uses the `web` entry point. It also connects the service to the `proxy` network, which is typically used by Traefik.

## Service Management

The M3TAL API daemon is managed by `systemd` as `m3tal-api.service`. You can use the following commands to manage its lifecycle:

-   **Check status:** `systemctl status m3tal-api`
-   **Restart service:** `systemctl restart m3tal-api`
-   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo

To quickly start the dashboard container:

1.  Run `m3tal dash up`. This command specifically manages the dashboard container, downloading the necessary compose files and applying the correct override based on your `/etc/m3tal/.env` configuration.

To deploy and manage all M3TAL stacks, including any custom stacks you have added:

1.  Run `m3tal up`. This command orchestrates and deploys all Docker Compose stacks defined in `*-compose.yml` files found within the `/docker/` directory.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port   | Service                                     | Access                                            | Description                                                                                               |
| :----- | :------------------------------------------ | :------------------------------------------------ | :-------------------------------------------------------------------------------------------------------- |
| 80     | Traefik HTTP entry point                    | Public (traefik mode)                             | The public-facing HTTP port for services exposed via Traefik.                                             |
| 8080   | M3TAL API daemon (Go)                       | Host-local                                        | The internal port the M3TAL API daemon listens on.                                                        |
| 8081   | Traefik dashboard                         | Host-local only                                   | The internal Traefik dashboard port, accessible only from the host machine.                               |
| 8082   | M3TAL Dashboard                             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).