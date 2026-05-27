# M3TAL Command Reference

This document provides a comprehensive cheat-sheet for all M3TAL CLI commands, designed for quick reference and efficient system management.

## Table of Contents

1.  [Interactive TUI](#interactive-tui)
2.  [Initialization & Configuration](#initialization--configuration)
3.  [Dashboard Management](#dashboard-management)
4.  [System-wide Operations](#system-wide-operations)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## Interactive TUI

The `sudo m3tal` command launches the M3TAL Interactive TUI Control Center. This provides a menu-driven interface for common operations, acting as a user-friendly alternative to direct CLI commands for certain tasks.

**Usage Example:**

```bash
sudo m3tal
```

---

## Initialization & Configuration

These commands manage the M3TAL configuration file (`/etc/m3tal/.env`), ensuring your system is set up correctly and settings are applied.

### `m3tal init`

Generates the default `/etc/m3tal/.env` configuration file. This command should be used immediately after a fresh installation to establish baseline settings.

**Usage Example:**

```bash
m3tal init
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, the validity of your `.env` file, and checks for port availability to ensure smooth operation.

**Usage Example:**

```bash
m3tal doctor
```

### `m3tal config wizard`

Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file. This is the recommended method for making significant configuration changes.

**Usage Example:**

```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in your `/etc/m3tal/.env` file. This is useful for quickly updating specific configuration parameters.

**Usage Example:**

```bash
m3tal config set DASHBOARD_PORT 8083
```

### `m3tal config get KEY`

Reads and displays the value of a single environment variable from your `/etc/m3tal/.env` file.

**Usage Example:**

```bash
m3tal config get LOG_LEVEL
```

### `m3tal config scan`

Lists all environment variables currently configured across all M3TAL stacks. This provides a comprehensive overview of your system's settings.

**Usage Example:**

```bash
m3tal config scan
```

### `m3tal config list`

Displays the entire contents of your current `/etc/m3tal/.env` file.

**Usage Example:**

```bash
m3tal config list
```

---

## Dashboard Management

These commands are specifically for managing the M3TAL Dashboard container, allowing you to control its state, update it, and view its logs.

### `m3tal dashpass [username] [password]`

Updates the password for a dashboard user. If `username` and `password` are omitted, the command will prompt you interactively.

**Usage Example (interactive):**

```bash
m3tal dashpass
```

**Usage Example (non-interactive):**

```bash
m3tal dashpass admin new_secure_password
```

### `m3tal dash up`

Pulls the latest dashboard Docker Compose configuration from GitHub and then starts the dashboard container. This ensures you are running the most up-to-date version of the dashboard.

**Usage Example:**

```bash
m3tal dash up
```

### `m3tal dash down`

Stops the M3TAL dashboard container.

**Usage Example:**

```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the M3TAL dashboard container.

**Usage Example:**

```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the M3TAL dashboard container in real-time. Press `Ctrl+C` to stop streaming.

**Usage Example:**

```bash
m3tal dash logs
```

### `m3tal dash status`

Displays the current status of the M3TAL dashboard container (e.g., running, exited).

**Usage Example:**

```bash
m3tal dash status
```

---

## System-wide Operations

These commands manage the Docker Compose stacks deployed by M3TAL, allowing you to bring services up, shut them down, and view aggregated logs.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This starts all your defined M3TAL services and any custom stacks you have added.

**Usage Example:**

```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` for all stacks managed by M3TAL. This stops and removes all containers, networks, and volumes associated with the defined stacks.

**Usage Example:**

```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks. This is useful for monitoring the overall health and activity of your deployed services. Press `Ctrl+C` to stop streaming.

**Usage Example:**

```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using standard `systemctl` and `journalctl` commands.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

**Usage Example:**

```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. This is essential for debugging issues with the API daemon. Press `Ctrl+C` to stop streaming.

**Usage Example:**

```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Fallback

In situations where the M3TAL CLI commands do not meet specific needs, you can interact directly with Docker Compose. The M3TAL system utilizes Docker Compose V2, and all stack compose files are located in the `/docker/` directory.

**Example:** To bring up all services defined in `/docker/`, you would typically run:

```bash
docker compose -f /docker/routing-compose.yml -f /docker/m3tal-compose.yml up -d --remove-orphans
```

*(Note: The exact compose files may vary based on your configuration and installed stacks. The `m3tal up` command orchestrates this for you.)*

---