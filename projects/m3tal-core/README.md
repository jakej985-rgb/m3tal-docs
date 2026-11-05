# M3TAL System Documentation

## Overview

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL system is distributed via an APT repository. Follow these steps to install:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core Components

The M3TAL system is comprised of several integrated components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary serving as the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a `systemd` service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API route handling.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the M3TAL API daemon at `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container that establishes a Cloudflare tunnel for secure, zero-configuration internet access to M3TAL services.

## Filesystem Contract

The M3TAL system adheres to a defined filesystem structure for its operation:

| Path                        | Purpose                                                                 |
| :-------------------------- | :---------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary configuration file, managed by `m3tal config wizard`.           |
| `/var/lib/m3tal/state.db`   | SQLite state database, automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all Docker Compose stack operations. |
| `/docker/users.json`        | Dashboard credential store, managed by the `m3tal dashpass` command.    |

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack. The `/opt/m3tal/stack/` directory is the canonical source of truth where all stack files reside, and `/docker` is the user-facing symlink alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:
1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2. Ensure any required environment variables are configured in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3. Run `m3tal up` to deploy all defined stacks, including your new one.

## Dashboard Access

The M3TAL Dashboard can be accessed in two distinct modes, controlled by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=local`
*   **Mechanism**: This mode uses an override (`m3tal-compose.local.yml`) to add a direct port binding, exposing the dashboard container's internal port `8082` to the host machine on port `8082`.
*   **Access**: Directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   **Requirement**: No Traefik gateway is required.
*   **Behavior**: A new user performing a default installation will access the dashboard directly via port `8082`. This is due to `DASHBOARD_EXPOSE_MODE=local` being the default setting.
*   **Use Case**: Ideal for LAN-only setups, initial configurations, and local testing environments.

### Traefik Mode

*   **Configuration**: `DASHBOARD_EXPOSE_MODE=traefik`
*   **Mechanism**: This mode uses an override (`m3tal-compose.traefik.yml`) to add specific Traefik labels to the dashboard service. Traefik interprets these labels to route incoming requests from `dash.DOMAIN` to the dashboard container's internal port `8082`.
*   **Access**: Via `http://dash.DOMAIN` (assuming `DOMAIN` is configured in `/etc/m3tal/.env`).
*   **Requirement**: The Traefik gateway must be running via `m3tal up`.
*   **Use Case**: Suitable for domain-based access, integration with other services behind Traefik, and public-facing deployments.

## Traefik Gateway

Traefik, deployed as a container via `routing-compose.yml`, serves as the reverse proxy for M3TAL. Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds to port `80` on the host machine, serving as the primary HTTP entry point. It loads dynamic configuration from files located in `/docker/dynamic/`, which allows for hot-reloading of routing rules.

*   `dash.DOMAIN` routes to the M3TAL dashboard container (when `DASHBOARD_EXPOSE_MODE=traefik`).
*   The API daemon, running on the host at `http://localhost:8080`, is exposed via Traefik using dynamic configuration. For example, `api.DOMAIN` is routed to the Go API daemon on host-local port `8080` via `http://host.docker.internal:8080` through a dynamic configuration file (e.g., `dynamic/api.yml`).

### Exposing a Custom Service via Traefik

To expose a custom user service through Traefik, add appropriate labels to its service definition in your Docker Compose file (e.g., `my-app-compose.yml`):

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.DOMAIN`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the same network as Traefik
```

After adding this, run `m3tal up` to deploy the changes.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Service Management

The M3TAL API daemon is managed as a `systemd` service named `m3tal-api.service`. Standard `systemctl` commands can be used for its operation:

*   **Check status**: `systemctl status m3tal-api`
*   **Restart service**: `systemctl restart m3tal-api`
*   **View logs**: `journalctl -u m3tal-api -f`

## Quick Demo

To quickly get started with the M3TAL dashboard:

1.  Start the M3TAL dashboard container specifically using:
    ```bash
    m3tal dash up
    ```
    This command downloads the necessary compose files, reads your `DASHBOARD_EXPOSE_MODE` from `/etc/m3tal/.env`, and starts the dashboard with the appropriate configuration.
2.  If you have `DASHBOARD_EXPOSE_MODE=local` (the default), access the dashboard at `http://HOST_IP:8082`.
3.  To deploy all other M3TAL stacks and any custom user-defined compose files located in `/docker/`, execute:
    ```bash
    m3tal up
    ```

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                     | Access                                      | Description                                                                                             |
| :--- | :-------------------------- | :------------------------------------------ | :------------------------------------------------------------------------------------------------------ |
| 80   | Traefik HTTP entry point    | Public                                      | The public-facing HTTP port for services exposed via Traefik.                                           |
| 8080 | M3TAL API daemon (Go)       | Host-local                                  | The internal port the M3TAL API daemon listens on.                                                      |
| 8081 | Traefik dashboard           | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine.                             |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Configuration

The primary configuration file for M3TAL is located at `/etc/m3tal/.env`. This file is managed interactively using the `m3tal config wizard` command, or individual keys can be set with `m3tal config set KEY value`.

### Environment Variables

The M3TAL system utilizes the following environment variables, primarily configured in `/etc/m3tal/.env`:

| Key                 | Default | Description                                                               |
| :------------------ | :------ | :------------------------------------------------------------------------ |
| `API_PORT`          | `5050`  | The port the M3TAL API daemon listens on (internal default is 8080).      |
| `API_TOKEN`         |         | Token for API authentication.                                             |
| `M3TAL_STATE_DB`    |         | Path to the SQLite state database. Defaults to `/var/lib/m3tal/state.db`. |
| `M3TAL_PLUGINS_DIR` |         | Directory for M3TAL plugins.                                              |
| `DASHBOARD_EXPOSE_MODE` | `local` | Controls how the dashboard is exposed (`local` or `traefik`).             |
| `DOMAIN`            |         | Base domain for Traefik routing (e.g., `example.com`).                    |