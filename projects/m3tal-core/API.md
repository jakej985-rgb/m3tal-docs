# M3TAL CLI Command Reference

This document provides a comprehensive guide to all available M3TAL CLI commands.

## Table of Contents

* [Getting Started](#getting-started)
* [Interactive Control Center](#interactive-control-center)
* [Configuration Management](#configuration-management)
* [Dashboard Management](#dashboard-management)
* [System-Wide Operations](#system-wide-operations)
* [Systemd Service Management](#systemd-service-management)
* [Direct Docker Compose Fallback](#direct-docker-compose-fallback)

---

## Getting Started

### Initial Setup

On first install, initialize the M3TAL environment by creating the default configuration file.

```bash
m3tal init
```

### Pre-flight Health Check

Run a comprehensive pre-flight check to ensure Docker is accessible, the `.env` file is valid, and necessary ports are available.

```bash
m3tal doctor
```

---

## Interactive Control Center

### Open TUI Control Center

Access the main interactive TUI (Text User Interface) control center, which presents a numbered menu for various operations.

```bash
sudo m3tal
```

---

## Configuration Management

### Configuration Wizard

Launch an interactive wizard to guide you through configuring the `/etc/m3tal/.env` file.

```bash
m3tal config wizard
```

### Set Environment Variable

Set a single environment variable in the `/etc/m3tal/.env` file.

```bash
m3tal config set GO_API_URL http://host.docker.internal:5050
```

### Get Environment Variable

Read a single environment variable from the `/etc/m3tal/.env` file.

```bash
m3tal config get DOMAIN
```

### Scan All Environment Variables

List all environment variables across all configured stacks, including their current values and defaults.

```bash
m3tal config scan
```

### List Current .env File Contents

Display the current contents of the `/etc/m3tal/.env` configuration file.

```bash
m3tal config list
```

---

## Dashboard Management

### Update Dashboard Password

Update the password for a specific dashboard user. If the username and password are not provided as arguments, you will be prompted interactively.

```bash
m3tal dashpass myuser newsecurepassword
```

Or, to be prompted:

```bash
m3tal dashpass
```

### Start Dashboard

Pull the latest dashboard compose configuration from GitHub and then start the dashboard container.

```bash
m3tal dash up
```

### Stop Dashboard

Stop the M3TAL dashboard container.

```bash
m3tal dash down
```

### Restart Dashboard

Restart the M3TAL dashboard container.

```bash
m3tal dash restart
```

### View Dashboard Logs

Stream the logs from the M3TAL dashboard container in real-time.

```bash
m3tal dash logs
```

### Check Dashboard Status

Show the current status of the M3TAL dashboard container.

```bash
m3tal dash status
```

---

## System-Wide Operations

### Bring Up All Stacks

Start all services defined in `*-compose.yml` files located in `/docker/`. This command orchestrates all your deployed stacks.

```bash
m3tal up
```

### Bring Down All Stacks

Stop and remove all containers, networks, and volumes defined in `*-compose.yml` files located in `/docker/`.

```bash
m3tal down
```

### Stream All Logs

Aggregate and stream logs from all running M3TAL stacks to your terminal.

```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon is managed by systemd. You can interact with it using standard `systemctl` and `journalctl` commands.

### Check API Service Status

View the status of the `m3tal-api.service`.

```bash
systemctl status m3tal-api
```

### View API Service Logs

Stream the logs from the `m3tal-api.service` in real-time.

```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Fallback

M3TAL utilizes Docker Compose under the hood. If you need to manage individual stacks or services directly, you can use `docker compose` commands within the `/docker/` directory.

### Example: Starting a Specific Stack

To start a stack defined in `/docker/my-app-compose.yml`:

```bash
cd /docker
docker compose -f my-app-compose.yml up -d
```

### Example: Stopping a Specific Stack

To stop a stack defined in `/docker/my-app-compose.yml`:

```bash
cd /docker
docker compose -f my-app-compose.yml down
```

### Example: Viewing Logs for a Specific Container

To view logs for a container named `my-container`:

```bash
docker compose logs -f my-container
```