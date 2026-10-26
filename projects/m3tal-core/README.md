# M3TAL System Documentation

## Overview
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

## Firewall Considerations
If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp`).

## Components
The M3TAL system consists of the following core components:
-   **CLI binary** (`/usr/bin/m3tal`): A unified Go binary providing a single entrypoint for all system operations.
-   **API daemon** (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`. It manages Docker interactions, the internal state database, and API routes.
-   **Dashboard container** (`m3tal-dashboard`): A Python/Flask application container running internally on port `8082`. It communicates with the API daemon at `http://host.docker.internal:8080`.
-   **Traefik gateway** (`routing-compose.yml`): A reverse proxy container that exposes services by domain name on host port `80`. It utilizes a file provider for dynamic routing configuration.
-   **Cloudflared** (`routing-compose.yml`): An optional Cloudflare tunnel container, facilitating zero-configuration internet access for exposed services.

## Filesystem Contract
The following table outlines the critical filesystem paths and their purposes within the M3TAL system:

| Path | Purpose |
|------|--------|
| `/etc/m3tal/.env` | Primary configuration file. Managed by `m3tal config wizard`. |
| `/var/lib/m3tal/state.db` | SQLite state database. Auto-created by the API daemon. |
| `/opt/m3tal/stack/` | Canonical stack directory. Contains Docker Compose files and Traefik configuration files. |
| `/docker` | Symlink → `/opt/m3tal/stack/`. This is the user-facing path for all stack operations. |
| `/docker/users.json` | Dashboard credential store. Managed by `m3tal dashpass`. |

## Deployment Lifecycle
M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory, effectively deploying each as an independent stack.

The `/opt/m3tal/stack/` directory is the canonical source of truth where all Docker Compose stack files and associated configurations reside. The `/docker` directory serves as a user-facing symlink alias to `/opt/m3tal/stack/` for all stack operations, simplifying access and management.

### Adding a New Stack
To deploy a new Docker Compose stack:
1.  Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/docker/` directory. This file will be automatically included by `m3tal up`.
2.  Ensure any required environment variables for your stack are defined in `/etc/m3tal/.env` (use `m3tal config wizard` or `m3tal config set KEY value`).
3.  Run `m3tal up` to deploy all currently defined stacks, including your new one.

## Dashboard Access
The M3TAL dashboard supports two distinct access modes, configured via the `DASHBOARD_EXPOSE_MODE` variable in `/etc/m3tal/.env`.

### Local Mode (Default)
-   **Configuration:** `DASHBOARD_EXPOSE_MODE=local`
-   **Access Method:** Direct port binding at `http://HOST_IP:8082` or `http://localhost:8082`.
-   **Details:** This is the default configuration for a new installation. A new user performing a default installation will access the dashboard directly via port 8082. This mode utilizes a Docker Compose override (`m3tal-compose.local.yml`) to bind the dashboard container's internal port `8082` directly to the host's `8082` port. Traefik is not required for access in this mode.
-   **Use Case:** Ideal for LAN-only setups, initial deployment, and local testing environments.

### Traefik Mode
-   **Configuration:** `DASHBOARD_EXPOSE_MODE=traefik`
-   **Access Method:** Domain routing at `http://dash.DOMAIN` via Traefik labels.
-   **Details:** This mode configures the dashboard container with Traefik labels via `m3tal-compose.traefik.yml`. Traefik, which must be running (e.g., via `m3tal up`), will then route requests for `dash.DOMAIN` to the dashboard container's internal port `8082`.
-   **Use Case:** Suited for domain-based access, environments with multiple services behind a reverse proxy, and public internet exposure.

## Traefik Gateway
Traefik is deployed as a Docker container, typically as part of `routing-compose.yml`. It functions as a reverse proxy, binding to host port `80` (and optionally `443` for HTTPS) as its primary HTTP entry point.

Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions. Crucially, services are not exposed by Traefik by default and require `traefik.enable=true` along with other relevant labels to be discoverable and routable.

Beyond Docker label discovery, Traefik leverages file providers for dynamic configuration. This allows for routing requests to services not directly exposed via Docker labels, such as host-local daemons. For example, the `dynamic/api.yml` configuration routes `api.DOMAIN` to the M3TAL Go API daemon listening on the host-local port `8080` via `http://host.docker.internal:8080`. Similarly, when `DASHBOARD_EXPOSE_MODE=traefik`, `dash.DOMAIN` routes to the dashboard container's internal port `8082`.

### Exposing a Custom User Service via Traefik
To expose a custom Docker Compose service via Traefik, add the necessary labels to its service definition:

```yaml
# /docker/my-app-compose.yml
services:
  my-app:
    image: nginx:alpine
    container_name: my-custom-app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`app.${DOMAIN}`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=80"
      - "traefik.http.routers.myapp.entrypoints=web"
    networks:
      - proxy # Ensure your service is on the 'proxy' network for Traefik to find it

networks:
  proxy:
    external: true
```
After placing this file in `/docker/` and running `m3tal up`, Traefik will route `http://app.YOUR_DOMAIN` to your `my-app` container.

## Service Management
The M3TAL API daemon is managed as a systemd service named `m3tal-api.service`. You can control and monitor it using standard `systemctl` commands:

-   **Check status:** `systemctl status m3tal-api`
-   **Restart service:** `systemctl restart m3tal-api`
-   **View logs:** `journalctl -u m3tal-api -f`

## Quick Demo
This section provides basic commands to get started with M3TAL.

-   **Start the M3TAL Dashboard:**
    To specifically start and manage only the M3TAL dashboard container, execute:
    ```bash
    m3tal dash up
    ```
    This command downloads the necessary dashboard compose files, applies the correct override based on your `DASHBOARD_EXPOSE_MODE` setting, and launches the dashboard.

-   **Deploy All M3TAL Stacks and User-Defined Services:**
    To orchestrate and deploy all Docker Compose stacks found within the `/docker/` directory, including the M3TAL control plane components and any custom user-defined compose files, use:
    ```bash
    m3tal up
    ```
    This command will ensure all services configured for M3TAL are running.

## Port Map
The following table lists the primary network ports utilized by the M3TAL system:

| Port | Service | Access | Description |
|------|---------|--------|-------------|
| 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
| 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
| 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
| 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

Note: These are the primary M3TAL-managed ports. User-added Docker Compose stacks may expose additional ports on their own, depending on their configurations.