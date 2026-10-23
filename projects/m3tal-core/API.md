# M3TAL CLI Command Reference

This document provides a comprehensive overview of all M3TAL CLI commands, including their usage and examples.

## Table of Contents

- [Core M3TAL Commands](#core-m3tal-commands)
- [Configuration Management](#configuration-management)
- [Dashboard Management](#dashboard-management)
- [Stack Management](#stack-management)
- [Systemd Service Management](#systemd-service-management)
- [Direct Docker Compose Commands](#direct-docker-compose-commands)

---

## Core M3TAL Commands

These commands provide direct access to M3TAL's functionalities and system information.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application. This is the primary graphical interface for interacting with M3TAL.

**Usage:**
```bash
m3tal
```
or
```bash
m3tal ui
```

**Example:**
```bash
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for running on home servers or systems without a graphical interface. Optionally, you can specify a port.

**Usage:**
```bash
m3tal headless [port]
```

**Example:**
```bash
m3tal headless 8080
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This is useful for monitoring the health and performance of your M3TAL host.

**Usage:**
```bash
m3tal stats
```

**Example:**
```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, the validity of your `.env` file, and port availability.

**Usage:**
```bash
m3tal doctor
```

**Example:**
```bash
m3tal doctor
```

---

## Configuration Management

Manage your M3TAL environment variables and configuration files.

### `m3tal config wizard`

Launches an interactive wizard to configure the `/etc/m3tal/.env` file. This is the recommended way to set up your initial M3TAL configuration.

**Usage:**
```bash
m3tal config wizard
```

**Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in your M3TAL configuration.

**Usage:**
```bash
m3tal config set KEY VALUE
```

**Example:**
```bash
m3tal config set DASHBOARD_PORT 8082
```

### `m3tal config get KEY`

Reads a single environment variable from your M3TAL configuration.

**Usage:**
```bash
m3tal config get KEY
```

**Example:**
```bash
m3tal config get DOMAIN
```

### `m3tal config scan`

Lists all environment variables across all active M3TAL stacks. This provides a comprehensive view of your M3TAL configuration.

**Usage:**
```bash
m3tal config scan
```

**Example:**
```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of your `.env` file. This displays all the variables currently defined in your M3TAL configuration.

**Usage:**
```bash
m3tal config list
```

**Example:**
```bash
m3tal config list
```

---

## Dashboard Management

Commands specifically for managing the M3TAL dashboard container.

### `m3tal dashpass [username] [password]`

Updates the M3TAL dashboard user password. If `username` and `password` are omitted, it will prompt for input interactively.

**Usage:**
```bash
m3tal dashpass [username] [password]
```

**Examples:**
```bash
m3tal dashpass admin mysecretpassword
```
or interactively:
```bash
m3tal dashpass
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.

**Usage:**
```bash
m3tal dash up
```

**Example:**
```bash
m3tal dash up
```

### `m3tal dash down`

Stops the M3TAL dashboard container.

**Usage:**
```bash
m3tal dash down
```

**Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the M3TAL dashboard container.

**Usage:**
```bash
m3tal dash restart
```

**Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the M3TAL dashboard container. This is useful for debugging.

**Usage:**
```bash
m3tal dash logs
```

**Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the M3TAL dashboard container.

**Usage:**
```bash
m3tal dash status
```

**Example:**
```bash
m3tal dash status
```

---

## Stack Management

Commands for managing all your deployed M3TAL stacks.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This starts all your deployed M3TAL services.

**Usage:**
```bash
m3tal up
```

**Example:**
```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all your M3TAL stacks. This stops all running M3TAL services.

**Usage:**
```bash
m3tal down
```

**Example:**
```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks. This provides a centralized view of your system logs.

**Usage:**
```bash
m3tal logs
```

**Example:**
```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon (`m3tal-api.service`) is managed by `systemd`.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

**Usage:**
```bash
systemctl status m3tal-api
```

**Example:**
```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. Use `-f` to follow the log output.

**Usage:**
```bash
journalctl -u m3tal-api -f
```

**Example:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Commands

As a fallback and for more granular control, you can use direct `docker compose` commands. M3TAL primarily uses Docker Compose V2.

### Starting all services:
```bash
docker compose -f /opt/m3tal/stack/routing-compose.yml -f /opt/m3tal/stack/m3tal-compose.yml -f /opt/m3tal/stack/m3tal-compose.local.yml up -d
```
*Note: The exact list of compose files might vary based on your `DASHBOARD_EXPOSE_MODE` and other installed stacks. The `m3tal up` command handles this complexity.*

### Stopping all services:
```bash
docker compose -f /opt/m3tal/stack/routing-compose.yml -f /opt/m3tal/stack/m3tal-compose.yml -f /opt/m3tal/stack/m3tal-compose.local.yml down
```

### Starting only the dashboard (if `DASHBOARD_EXPOSE_MODE=local`):
```bash
docker compose -f /opt/m3tal/stack/m3tal-compose.yml -f /opt/m3tal/stack/m3tal-compose.local.yml up -d m3tal-dashboard
```

### Stopping only the dashboard:
```bash
docker compose -f /opt/m3tal/stack/m3tal-compose.yml -f /opt/m3tal/stack/m3tal-compose.local.yml down m3tal-dashboard
```

### Streaming dashboard logs:
```bash
docker compose -f /opt/m3tal/stack/m3tal-compose.yml -f /opt/m3tal/stack/m3tal-compose.local.yml logs -f m3tal-dashboard
```

*Remember to replace `/opt/m3tal/stack/` with your actual M3TAL stack directory if it differs.*