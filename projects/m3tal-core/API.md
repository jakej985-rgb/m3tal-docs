# M3TAL Command Reference

This document provides a comprehensive reference for all M3TAL CLI commands, serving as a cheat-sheet for managing your M3TAL ecosystem.

---

## Table of Contents

1.  [Core Commands](#core-commands)
2.  [Configuration Management](#configuration-management)
3.  [Dashboard Management](#dashboard-management)
4.  [Stack Management](#stack-management)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Commands (Fallback)](#direct-docker-compose-commands-fallback)

---

## Core Commands

These commands are your primary interface for interacting with the M3TAL system.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application. This is your graphical entry point for managing M3TAL.

*   **Usage:**
    ```bash
    m3tal
    ```
    or
    ```bash
    m3tal ui
    ```

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for home servers or environments without a graphical interface. The optional `port` argument specifies the port to listen on.

*   **Usage:**
    ```bash
    m3tal headless 8080
    ```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This is useful for quick system health checks.

*   **Usage:**
    ```bash
    m3tal stats
    ```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, `.env` file validity, and port availability.

*   **Usage:**
    ```bash
    m3tal doctor
    ```

---

## Configuration Management

Manage your M3TAL environment variables and settings.

### `m3tal config wizard`

Launches an interactive wizard to configure the primary M3TAL configuration file, `/etc/m3tal/.env`.

*   **Usage:**
    ```bash
    m3tal config wizard
    ```

### `m3tal config set KEY VALUE`

Sets a single environment variable in your M3TAL configuration.

*   **Usage:**
    ```bash
    m3tal config set DOMAIN mydomain.com
    ```

### `m3tal config get KEY`

Reads a single environment variable from your M3TAL configuration.

*   **Usage:**
    ```bash
    m3tal config get DASHBOARD_PORT
    ```

### `m3tal config scan`

Lists all environment variables across all active M3TAL stacks, providing a comprehensive overview of your configuration.

*   **Usage:**
    ```bash
    m3tal config scan
    ```

### `m3tal config list`

Lists the current contents of your `.env` file (`/etc/m3tal/.env`), showing all active configuration variables.

*   **Usage:**
    ```bash
    m3tal config list
    ```

---

## Dashboard Management

Commands specifically for managing the M3TAL web dashboard.

### `m3tal dashpass [username] [password]`

Updates the dashboard user's password. If `username` and `password` are omitted, the command becomes interactive.

*   **Usage (interactive):**
    ```bash
    m3tal dashpass
    ```
*   **Usage (with arguments):**
    ```bash
    m3tal dashpass admin mynewsecurepassword
    ```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures your dashboard is running the most recent version and configuration.

*   **Usage:**
    ```bash
    m3tal dash up
    ```

### `m3tal dash down`

Stops the M3TAL dashboard container.

*   **Usage:**
    ```bash
    m3tal dash down
    ```

### `m3tal dash restart`

Restarts the M3TAL dashboard container.

*   **Usage:**
    ```bash
    m3tal dash restart
    ```

### `m3tal dash logs`

Streams the logs from the M3TAL dashboard container in real-time.

*   **Usage:**
    ```bash
    m3tal dash logs
    ```

### `m3tal dash status`

Shows the current status of the M3TAL dashboard container.

*   **Usage:**
    ```bash
    m3tal dash status
    ```

---

## Stack Management

Commands for managing your deployed M3TAL stacks.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This command brings up all your defined services and stacks.

*   **Usage:**
    ```bash
    m3tal up
    ```

### `m3tal down`

Runs `docker compose down` across all stacks managed by M3TAL. This command stops and removes all containers, networks, and volumes associated with your M3TAL stacks.

*   **Usage:**
    ```bash
    m3tal down
    ```

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks to your terminal. This provides a centralized view of your system's activity.

*   **Usage:**
    ```bash
    m3tal logs
    ```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service for robust background operation.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

*   **Usage:**
    ```bash
    systemctl status m3tal-api
    ```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. This is equivalent to `m3tal logs` but specifically for the API daemon.

*   **Usage:**
    ```bash
    journalctl -u m3tal-api -f
    ```

---

## Direct Docker Compose Commands (Fallback)

While the `m3tal` CLI abstracts away most Docker Compose interactions, you can use these direct commands as a fallback or for more granular control.

**Note:** The M3TAL system relies on Docker Engine and Docker Compose V2.

### Bringing up all stacks

This command is equivalent to `m3tal up`.

*   **Usage:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/*.yml up -d
    ```

### Bringing down all stacks

This command is equivalent to `m3tal down`.

*   **Usage:**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/routing-compose.yml -f /docker/*.yml down
    ```

### Managing the dashboard specifically

These commands are equivalent to `m3tal dash up`, `m3tal dash down`, and `m3tal dash restart`.

*   **Usage (Dashboard Up):**
    ```bash
    # Ensure you have downloaded the latest compose files first, e.g., via m3tal dash up itself or manually
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d m3tal-dashboard
    ```
    or if `DASHBOARD_EXPOSE_MODE=traefik`:
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml up -d m3tal-dashboard
    ```

*   **Usage (Dashboard Down):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down m3tal-dashboard
    ```
    or if `DASHBOARD_EXPOSE_MODE=traefik`:
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml down m3tal-dashboard
    ```

*   **Usage (Dashboard Restart):**
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml restart m3tal-dashboard
    ```
    or if `DASHBOARD_EXPOSE_MODE=traefik`:
    ```bash
    docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.traefik.yml restart m3tal-dashboard
    ```

---
