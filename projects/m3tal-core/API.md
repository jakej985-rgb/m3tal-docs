# M3TAL CLI Command Reference

This document provides a comprehensive reference for all available `m3tal` CLI commands, serving as a cheat-sheet for managing your M3TAL environment.

## Table of Contents

* [Getting Started](#getting-started)
* [Core M3TAL Operations](#core-m3tal-operations)
* [Configuration Management](#configuration-management)
* [Dashboard Management](#dashboard-management)
* [System Operations](#system-operations)
* [Systemd Service Management](#systemd-service-management)
* [Direct Docker Compose Commands](#direct-docker-compose-commands)

## Getting Started

### Installation

To install M3TAL, follow these steps:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```

## Core M3TAL Operations

### Launch M3TAL Desktop Application

Launches the M3TAL Linux Desktop Application.

```bash
m3tal
```

Or, equivalently:

```bash
m3tal ui
```

### Launch M3TAL Headless Mode

Starts M3TAL in headless mode, ideal for home servers. You can optionally specify a port.

```bash
m3tal headless 8080
```

### Get System Metrics

Prints instant host system, GPU, thermal, and storage metrics to the terminal.

```bash
m3tal stats
```

### Run Health Check

Performs a pre-flight health check for Docker connectivity, `.env` validity, and port availability.

```bash
m3tal doctor
```

## Configuration Management

### Interactive Configuration Wizard

Launches an interactive wizard to configure `/etc/m3tal/.env`.

```bash
m3tal config wizard
```

### Set Environment Variable

Sets a single environment variable in the `.env` file.

```bash
m3tal config set DASHBOARD_EXPOSE_MODE traefik
```

### Get Environment Variable

Reads a single environment variable from the `.env` file.

```bash
m3tal config get DOMAIN
```

### Scan All Environment Variables

Lists all environment variables across all stacks, including defaults.

```bash
m3tal config scan
```

### List Current `.env` File

Lists the current contents of the `.env` file.

```bash
m3tal config list
```

## Dashboard Management

### Update Dashboard Password

Updates the dashboard user password. This command will be interactive if username and password arguments are omitted.

```bash
m3tal dashpass admin new_secure_password
```

### Start Dashboard

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container.

```bash
m3tal dash up
```

### Stop Dashboard

Stops the dashboard container.

```bash
m3tal dash down
```

### Restart Dashboard

Restarts the dashboard container.

```bash
m3tal dash restart
```

### Stream Dashboard Logs

Streams the logs from the dashboard container.

```bash
m3tal dash logs
```

### Show Dashboard Status

Shows the current status of the dashboard container.

```bash
m3tal dash status
```

## System Operations

### Bring Up All Stacks

Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`.

```bash
m3tal up
```

### Tear Down All Stacks

Runs `docker compose down` across all stacks.

```bash
m3tal down
```

### Stream All Logs

Streams aggregated logs from all running stacks.

```bash
m3tal logs
```

## Systemd Service Management

The M3TAL API daemon runs as a systemd service. You can manage and monitor it using the following commands:

### Check API Service Status

```bash
systemctl status m3tal-api
```

### Restart API Service

```bash
systemctl restart m3tal-api
```

### Follow API Service Logs

```bash
journalctl -u m3tal-api -f
```

## Direct Docker Compose Commands

As a fallback or for more granular control, you can directly use Docker Compose commands on the compose files located in `/docker/`.

### Run Docker Compose Up

This command will execute `docker compose up` within the `/docker/` directory.

```bash
docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml up -d
```
*Note: The specific compose files used depend on your `DASHBOARD_EXPOSE_MODE` and any additional stack files present in `/docker/`.*

### Run Docker Compose Down

This command will execute `docker compose down` within the `/docker/` directory.

```bash
docker compose -f /docker/m3tal-compose.yml -f /docker/m3tal-compose.local.yml down
```
*Note: The specific compose files used depend on your `DASHBOARD_EXPOSE_MODE` and any additional stack files present in `/docker/`.*

### Stream Docker Compose Logs

To stream logs for a specific service (e.g., `m3tal-dashboard`):

```bash
docker compose logs -f m3tal-dashboard
```

### Check Docker Compose Status

To view the status of services managed by Docker Compose:

```bash
docker compose ps
```