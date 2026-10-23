# M3TAL CLI Command Reference

This document provides a comprehensive reference for all M3TAL command-line interface (CLI) commands.

---

## Table of Contents

1.  [Core M3TAL Commands](#core-m3tal-commands)
2.  [Configuration Management](#configuration-management)
3.  [Dashboard Management](#dashboard-management)
4.  [Stack Management](#stack-management)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## Core M3TAL Commands

These commands provide fundamental operations for interacting with the M3TAL system.

### `m3tal ui`

Launches the M3TAL Linux Desktop Application. This is the primary graphical interface for managing M3TAL services.

**Usage Example:**
```bash
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode, suitable for servers without a graphical interface. The optional `port` argument specifies the port for the API.

**Usage Example:**
```bash
m3tal headless 8080
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This command is useful for monitoring the health and performance of your M3TAL host.

**Usage Example:**
```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. This includes checking Docker connectivity, the validity of your `.env` file, and port availability.

**Usage Example:**
```bash
m3tal doctor
```

---

## Configuration Management

These commands allow you to manage the M3TAL environment variables stored in `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to configure the `/etc/m3tal/.env` file. This is the recommended way to set up your initial configuration.

**Usage Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in your `/etc/m3tal/.env` file. Replace `KEY` with the environment variable name and `VALUE` with its desired value.

**Usage Example:**
```bash
m3tal config set DOMAIN myhost.local
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from your `/etc/m3tal/.env` file.

**Usage Example:**
```bash
m3tal config get DASHBOARD_PORT
```

### `m3tal config scan`

Lists all environment variables currently recognized by M3TAL across all managed stacks. This helps in understanding the available configuration options.

**Usage Example:**
```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of your `/etc/m3tal/.env` file, showing all configured environment variables.

**Usage Example:**
```bash
m3tal config list
```

---

## Dashboard Management

These commands specifically manage the M3TAL dashboard container.

### `m3tal dashpass [username] [password]`

Updates the dashboard user password. If `username` and `password` are omitted, the command will prompt you interactively. This command modifies `/docker/users.json`.

**Usage Example (interactive):**
```bash
m3tal dashpass
```

**Usage Example (with arguments):**
```bash
m3tal dashpass admin mysecretpassword123
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you are running the most up-to-date version of the dashboard.

**Usage Example:**
```bash
m3tal dash up
```

### `m3tal dash down`

Stops the dashboard container.

**Usage Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`

Restarts the dashboard container.

**Usage Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`

Streams the logs from the dashboard container in real-time. This is essential for debugging dashboard issues.

**Usage Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the dashboard container (e.g., running, exited).

**Usage Example:**
```bash
m3tal dash status
```

---

## Stack Management

These commands manage all M3TAL services defined in compose files within the `/docker/` directory.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files located in the `/docker/` directory. This command starts all your defined M3TAL services and any custom stacks you have added.

**Usage Example:**
```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all stacks managed by M3TAL. This command stops and removes all containers, networks, and volumes associated with your M3TAL services.

**Usage Example:**
```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all currently running M3TAL stacks to your terminal. This provides a centralized view of logs from all your services.

**Usage Example:**
```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using `systemctl` and `journalctl`.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service` (e.g., active, inactive, failed).

**Usage Example:**
```bash
systemctl status m3tal-api
```

### `systemctl restart m3tal-api`

Restarts the `m3tal-api.service`.

**Usage Example:**
```bash
systemctl restart m3tal-api
```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time using `journalctl`. Press `Ctrl+C` to exit.

**Usage Example:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Fallback

In cases where `m3tal` commands might not cover a specific scenario or for direct manipulation, you can use Docker Compose commands. M3TAL relies on Docker Engine and Docker Compose V2. All stack compose files are located in `/docker/`.

### Starting all services:
```bash
docker compose -f /docker/*.yml up -d
```
*Note: `-d` runs containers in detached mode.*

### Stopping all services:
```bash
docker compose -f /docker/*.yml down
```

### Viewing logs for a specific service (e.g., `m3tal-dashboard`):
```bash
docker compose -f /docker/*.yml logs m3tal-dashboard
```

### Viewing logs for all services:
```bash
docker compose -f /docker/*.yml logs
```

---

## APT Installation

To install or update M3TAL using APT:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```