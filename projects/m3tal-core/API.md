```markdown
# M3TAL CLI Command Reference

This document provides a comprehensive reference for all `m3tal` CLI commands.

## Table of Contents

- [M3TAL Core Commands](#m3tal-core-commands)
- [Configuration Management](#configuration-management)
- [Dashboard Management](#dashboard-management)
- [Stack Management](#stack-management)
- [Systemd Service Management](#systemd-service-management)
- [Direct Docker Compose Commands (Fallback)](#direct-docker-compose-commands-fallback)

---

## M3TAL Core Commands

These commands provide the fundamental entry points for interacting with the M3TAL ecosystem.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application.

**Usage:**
```bash
m3tal
```
or
```bash
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode. This is ideal for running M3TAL on servers without a graphical interface. The optional `[port]` argument specifies the port to listen on.

**Usage:**
```bash
m3tal headless 8080
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal.

**Usage:**
```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL environment, verifying Docker connectivity, `.env` file validity, and port availability.

**Usage:**
```bash
m3tal doctor
```

---

## Configuration Management

Manage M3TAL's configuration settings, primarily stored in `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file.

**Usage:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in the `.env` file.

**Usage:**
```bash
m3tal config set DOMAIN mym3tal.local
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from the `.env` file.

**Usage:**
```bash
m3tal config get DASHBOARD_PORT
```

### `m3tal config scan`

Lists all environment variables across all managed stacks, providing a comprehensive overview of your configuration.

**Usage:**
```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of the `.env` file in your terminal.

**Usage:**
```bash
m3tal config list
```

---

## Dashboard Management

Commands specific to managing the M3TAL dashboard container.

### `m3tal dashpass [username] [password]`

Updates the dashboard user password. If `[username]` and `[password]` are omitted, the command will prompt for them interactively.

**Usage (interactive):**
```bash
m3tal dashpass
```
**Usage (with arguments):**
```bash
m3tal dashpass admin new_secure_password
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.

**Usage:**
```bash
m3tal dash up
```

### `m3tal dash down`

Stops the dashboard container.

**Usage:**
```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the dashboard container.

**Usage:**
```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the dashboard container in real-time.

**Usage:**
```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the dashboard container.

**Usage:**
```bash
m3tal dash status
```

---

## Stack Management

Commands for managing all M3TAL stacks, including custom user stacks.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command starts all your defined services.

**Usage:**
```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all stacks, stopping all services managed by M3TAL.

**Usage:**
```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks to your terminal.

**Usage:**
```bash
m3tal logs
```

---

## Systemd Service Management

M3TAL's API daemon runs as a systemd service. You can manage and monitor it using standard `systemctl` and `journalctl` commands.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api` service.

**Usage:**
```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api` service in real-time. This is useful for debugging issues with the API daemon.

**Usage:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Commands (Fallback)

In situations where the `m3tal` CLI might not be sufficient or for direct control, you can use Docker Compose commands directly. M3TAL utilizes Docker Compose V2.

**Note:** These commands assume you are in the directory containing the relevant `*-compose.yml` files (typically `/docker/`).

### Starting all services:
```bash
docker compose up -d
```

### Stopping all services:
```bash
docker compose down
```

### Restarting a specific service (e.g., `m3tal-dashboard`):
```bash
docker compose restart m3tal-dashboard
```

### Viewing logs for a specific service (e.g., `m3tal-dashboard`):
```bash
docker compose logs -f m3tal-dashboard
```

### Pulling the latest images for all services:
```bash
docker compose pull
```

### Displaying the status of services:
```bash
docker compose ps
```
```