# M3TAL System Documentation

This document provides technical details and operational procedures for the M3TAL system.

## Prerequisites

Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.

## Installation

The M3TAL CLI binary and API daemon are installed via an APT repository.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core Components

The M3TAL system comprises several interconnected components:

*   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing the primary command-line interface for all M3TAL operations.
*   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, maintains the internal state database, and exposes the M3TAL API routes.
*   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container. It runs internally on port `8082` and communicates with the M3TAL API daemon via `http://host.docker.internal:8080`.
*   **Traefik gateway** (`routing-compose.yml`): A Docker container acting as a reverse proxy. It exposes services by domain name on host port `80` and utilizes a file provider for dynamic routing configuration.
*   **Cloudflared** (`routing-compose.yml`): An optional Docker container providing Cloudflare tunnel capabilities for secure, zero-configuration internet access to services.

## Filesystem Contract

The following paths represent critical directories and files within the M3TAL system:

| Path                        | Purpose                                                                |
| :-------------------------- | :--------------------------------------------------------------------- |
| `/etc/m3tal/.env`           | Primary system configuration file. Managed by `m3tal config wizard`.   |
| `/var/lib/m3tal/state.db`   | SQLite state database, automatically created and managed by the API daemon. |
| `/opt/m3tal/stack/`         | Canonical directory for all Docker Compose stack files and Traefik configuration. |
| `/docker`                   | Symlink to `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json`        | Credential store for the M3TAL Dashboard, managed by `m3tal dashpass`. |

## Service Management

The M3TAL API daemon is managed as a `systemd` service.

*   **Check status:** `systemctl status m3tal-api.service`
*   **Restart service:** `systemctl restart m3tal-api.service`
*   **View logs:** `journalctl -u m3tal-api.service -f`

## Deployment Lifecycle

M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files and Traefik configurations reside. The `/docker` directory is a symlink to `/opt/m3tal/stack/`, serving as the user-facing alias for all stack operations.

### Adding a New Stack

To deploy a new Docker Compose stack:

1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables are set in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to start all deployed stacks, including your newly added one.

## Firewall Considerations

If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Traefik Gateway

The Traefik gateway, deployed via `routing-compose.yml`, acts as the reverse proxy for M3TAL. It automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Traefik binds host port `80` as its HTTP entry point. It loads dynamic configuration from files placed in `/docker/dynamic/`, which allows for hot-reloading of routing rules. For instance, `api.DOMAIN` routes requests to the Go API daemon listening on host-local port `8080` by utilizing a dynamic configuration file like `dynamic/api.yml` that points to `http://host.docker.internal:8080`. When `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` is routed to the dashboard container via Traefik labels defined in `m3tal-compose.traefik.yml`.

### Exposing a Custom Service via Traefik

To expose a custom user service (e.g., `my-app`) via Traefik, add appropriate labels to its service definition within your Docker Compose file (e.g., `my-app-compose.yml`). The service must also be connected to the `proxy` network managed by Traefik.

```yaml
# my-app-compose.yml snippet
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
    external: true # Connects to the existing 'proxy' network created by Traefik
```

## Dashboard Access

The M3TAL Dashboard supports two distinct access modes, configured by the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)

*   **`DASHBOARD_EXPOSE_MODE=local`**
*   This is the default configuration for new installations.
*   The dashboard container is configured with a direct port binding: `${DASHBOARD_PORT:-8082}:8082`.
*   Access the dashboard directly via `http://HOST_IP:8082` or `http://localhost:8082`.
*   No Traefik instance is required for this mode. A new user performing a default installation will access the dashboard directly via port 8082.
*   **Best for:** LAN-only deployments, initial setup, or local development without domain routing.

### Traefik Mode

*   **`DASHBOARD_EXPOSE_MODE=traefik`**
*   The dashboard container is configured with Traefik labels (via `m3tal-compose.traefik.yml`) to allow Traefik to route `dash.${DOMAIN}` to the dashboard container on its internal port `8082`.
*   Access the dashboard via `http://dash.DOMAIN`.
*   Requires the Traefik gateway (`routing-compose.yml`) to be running (`m3tal up`).
*   **Best for:** Domain-based setups, integrating with other services behind a reverse proxy.

## Quick Demo

To quickly get the M3TAL dashboard running:

*   **Start the M3TAL Dashboard:**
    ```bash
    m3tal dash up
    ```
    This command specifically downloads the latest dashboard compose files, reads your `DASHBOARD_EXPOSE_MODE` setting, and starts the `m3tal-dashboard` container with the appropriate configuration.
*   **Access the Dashboard:**
    *   If using the default `DASHBOARD_EXPOSE_MODE=local`, navigate to `http://HOST_IP:8082` in your web browser.
    *   If using `DASHBOARD_EXPOSE_MODE=traefik` (and Traefik is running via `m3tal up`), navigate to `http://dash.YOUR_DOMAIN`.

The `m3tal up` command, in contrast, orchestrates and deploys *all* Docker Compose stacks found in the `/docker/` directory, including any custom user-defined compose files, in addition to core M3TAL components.

## Port Map

The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service                     | Access                                      | Description                                                                                                                              |
| :--- | :-------------------------- | :------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------- |
| 80   | Traefik HTTP entry point    | Public                                      | The public-facing HTTP port for services exposed via Traefik.                                                                            |
| 8080 | M3TAL API daemon (Go)       | Host-local                                  | The internal port the M3TAL API daemon listens on.                                                                                       |
| 8081 | Traefik dashboard           | Host-local only                             | The internal Traefik dashboard port, accessible only from the host machine.                                                              |
| 8082 | M3TAL Dashboard             | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`.                           |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.

## Environment Variables

Key environment variables that influence M3TAL system behavior:

| Key                 | Default Value | Description                                                    |
| :------------------ | :------------ | :------------------------------------------------------------- |
| `API_PORT`          | `5050`        | The port on which the M3TAL API daemon listens (though the systemd service uses 8080). |
| `API_TOKEN`         | `(empty)`     | Authentication token for API access.                           |
| `M3TAL_STATE_DB`    | `(empty)`     | Path to the SQLite state database file.                        |
| `M3TAL_PLUGINS_DIR` | `(empty)`     | Directory where M3TAL plugins are stored.                      |