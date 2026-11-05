# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for all available `m3tal` CLI commands.

---

## Table of Contents

1.  [Launching M3TAL](#launching-m3tal)
2.  [Headless Mode](#headless-mode)
3.  [System Metrics](#system-metrics)
4.  [System Health Check](#system-health-check)
5.  [Configuration Management](#configuration-management)
6.  [Dashboard Management](#dashboard-management)
7.  [Stack Management](#stack-management)
8.  [Systemd Service Management](#systemd-service-management)
9.  [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## Launching M3TAL

### `m3tal`
Launches the M3TAL Linux Desktop Application.

**Example:**
```bash
m3tal
```

### `m3tal ui`
An alias for launching the M3TAL Linux Desktop Application.

**Example:**
```bash
m3tal ui
```

---

## Headless Mode

### `m3tal headless [port]`
Starts M3TAL in headless mode, ideal for home servers or environments without a graphical interface. You can optionally specify a port.

**Example:**
```bash
m3tal headless 8080
```

---

## System Metrics

### `m3tal stats`
Prints instant host system, GPU, thermal, and storage metrics to the terminal.

**Example:**
```bash
m3tal stats
```

---

## System Health Check

### `m3tal doctor`
Performs a pre-flight health check, verifying Docker connectivity, `.env` validity, and port availability.

**Example:**
```bash
m3tal doctor
```

---

## Configuration Management

### `m3tal config wizard`
Launches an interactive wizard to configure the `/etc/m3tal/.env` file.

**Example:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`
Sets a single environment variable in the `.env` file.

**Example:**
```bash
m3tal config set DOMAIN mym3tal.com
```

### `m3tal config get KEY`
Reads a single environment variable from the `.env` file.

**Example:**
```bash
m3tal config get DASHBOARD_PORT
```

### `m3tal config scan`
Lists all environment variables across all configured stacks.

**Example:**
```bash
m3tal config scan
```

### `m3tal config list`
Lists the current contents of the `.env` file.

**Example:**
```bash
m3tal config list
```

---

## Dashboard Management

The M3TAL dashboard provides a web interface for managing your M3TAL instance. Access depends on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

### `m3tal dashpass [username] [password]`
Updates the dashboard user password. If username and password are omitted, the command becomes interactive.

**Example (interactive):**
```bash
m3tal dashpass
```

**Example (with username and password):**
```bash
m3tal dashpass admin new_secure_password
```

### `m3tal dash up`
Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This command respects the `DASHBOARD_EXPOSE_MODE` setting.

**Example:**
```bash
m3tal dash up
```

### `m3tal dash down`
Stops the dashboard container.

**Example:**
```bash
m3tal dash down
```

### `m3tal dash restart`
Restarts the dashboard container.

**Example:**
```bash
m3tal dash restart
```

### `m3tal dash logs`
Streams the dashboard container logs.

**Example:**
```bash
m3tal dash logs
```

### `m3tal dash status`
Shows the status of the dashboard container.

**Example:**
```bash
m3tal dash status
```

---

## Stack Management

These commands manage all Docker Compose stacks deployed within M3TAL.

### `m3tal up`
Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all your deployed services.

**Example:**
```bash
m3tal up
```

### `m3tal down`
Runs `docker compose down` across all configured stacks, stopping all running services.

**Example:**
```bash
m3tal down
```

### `m3tal logs`
Streams aggregated logs from all running M3TAL stacks.

**Example:**
```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service.

### `systemctl status m3tal-api`
Checks the status of the `m3tal-api` service.

**Example:**
```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`
Streams the logs for the `m3tal-api` service in real-time.

**Example:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Fallback

In cases where you need more granular control or are troubleshooting, you can use Docker Compose commands directly. M3TAL manages services using Compose files located in `/docker/`.

### `docker compose up`
Starts services defined in `*-compose.yml` files in the current directory. Use this within `/docker/` to manage individual stacks if needed.

**Example (from within `/docker/`):**
```bash
cd /docker/
docker compose -f my-stack-compose.yml up -d
```

### `docker compose down`
Stops and removes containers, networks, and volumes defined in Compose files.

**Example (from within `/docker/`):**
```bash
cd /docker/
docker compose -f my-stack-compose.yml down
```

### `docker compose ps`
Lists the containers for the services defined in the Compose files.

**Example (from within `/docker/`):**
```bash
cd /docker/
docker compose ps
```

---

## APT Installation

To install or update M3TAL, use the following APT commands:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```