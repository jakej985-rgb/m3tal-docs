# M3TAL Command Reference

This document provides a comprehensive cheat-sheet for all M3TAL CLI commands, designed to be your go-to reference for managing the M3TAL ecosystem.

## Table of Contents

- [Core M3TAL Commands](#core-m3tal-commands)
- [Configuration Management (`m3tal config`)](#configuration-management-m3tal-config)
- [Dashboard Management (`m3tal dash`)](#dashboard-management-m3tal-dash)
- [Systemd Service Management](#systemd-service-management)
- [Direct Docker Compose Commands](#direct-docker-compose-commands)
- [APT Installation](#apt-installation)

---

## Core M3TAL Commands

These commands are the primary interface for interacting with the M3TAL system.

### Launch M3TAL Desktop Application

Starts the M3TAL Linux Desktop Application. This is the graphical user interface for managing your M3TAL services.

```bash
m3tal
```

or

```bash
m3tal ui
```

### Launch M3TAL in Headless Mode

Starts M3TAL in headless mode, suitable for servers where a GUI is not needed. You can specify a port for the API to listen on.

```bash
m3tal headless 8080
```

### System Metrics

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This is useful for monitoring your system's health and performance.

```bash
m3tal stats
```

### Pre-flight Health Check

Performs a comprehensive health check of your M3TAL installation, verifying Docker connectivity, `.env` file validity, and port availability.

```bash
m3tal doctor
```

### Run All Stacks

Executes `docker compose up -d` across all `*-compose.yml` files found in `/docker/`. This starts all your defined M3TAL stacks.

```bash
m3tal up
```

### Stop All Stacks

Executes `docker compose down` across all stacks. This stops all containers managed by M3TAL.

```bash
m3tal down
```

### Stream Aggregated Logs

Streams aggregated logs from all currently running M3TAL stacks. This is invaluable for debugging and understanding system behavior.

```bash
m3tal logs
```

---

## Configuration Management (`m3tal config`)

Manage your M3TAL environment variables and configuration files.

### Interactive Configuration Wizard

Launches an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file. This is the recommended way to set up M3TAL for the first time.

```bash
m3tal config wizard
```

### Set a Single Environment Variable

Sets a specific environment variable in your `.env` file.

```bash
m3tal config set DASHBOARD_PORT 8082
```

### Get a Single Environment Variable

Reads and displays the value of a specific environment variable from your `.env` file.

```bash
m3tal config get LOG_LEVEL
```

### Scan All Environment Variables

Lists all environment variables across all configured M3TAL stacks.

```bash
m3tal config scan
```

### List Current `.env` File Contents

Displays the current contents of your `/etc/m3tal/.env` file.

```bash
m3tal config list
```

---

## Dashboard Management (`m3tal dash`)

Commands specifically for managing the M3TAL dashboard container.

### Update Dashboard Password

Updates the password for the dashboard user. If `username` and `password` are omitted, the command will prompt you interactively.

```bash
m3tal dashpass admin new_secure_password123
```

### Update and Start Dashboard

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.

```bash
m3tal dash up
```

### Stop Dashboard

Stops the M3TAL dashboard container.

```bash
m3tal dash down
```

### Restart Dashboard

Restarts the M3TAL dashboard container.

```bash
m3tal dash restart
```

### Stream Dashboard Logs

Streams the logs from the M3TAL dashboard container in real-time.

```bash
m3tal dash logs
```

### Show Dashboard Status

Displays the current status of the M3TAL dashboard container.

```bash
m3tal dash status
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using `systemctl` and view its logs with `journalctl`.

### Check API Service Status

View the current status of the `m3tal-api.service`.

```bash
systemctl status m3tal-api
```

### Restart API Service

Restart the `m3tal-api.service`.

```bash
systemctl restart m3tal-api
```

### View API Service Logs

Stream the logs from the `m3tal-api.service` in real-time.

```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Commands

As a fallback, you can always interact with Docker Compose directly. M3TAL utilizes Docker Compose V2.

### Run Docker Compose Up for Specific Stacks

Starts services defined in compose files within the `/docker/` directory. For example, to start services in `my-stack-compose.yml`:

```bash
docker compose -f /docker/my-stack-compose.yml up -d
```

### Run Docker Compose Down for Specific Stacks

Stops services defined in compose files within the `/docker/` directory. For example, to stop services in `my-stack-compose.yml`:

```bash
docker compose -f /docker/my-stack-compose.yml down
```

### View Logs for a Specific Container

Stream logs for a particular Docker container. For example, to view logs for the `m3tal-dashboard` container:

```bash
docker logs m3tal-dashboard -f
```

---

## APT Installation

This section provides the exact commands for installing M3TAL via APT.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```