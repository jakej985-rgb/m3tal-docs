```markdown
# M3TAL CLI Command Reference

This document provides a comprehensive cheat-sheet for all available M3TAL CLI commands.

## Table of Contents

1.  [Core M3TAL Commands](#core-m3tal-commands)
2.  [Configuration Management](#configuration-management)
3.  [Dashboard Management](#dashboard-management)
4.  [Stack Management](#stack-management)
5.  [Systemd Service Management](#systemd-service-management)
6.  [Direct Docker Compose Commands (Fallback)](#direct-docker-compose-commands-fallback)
7.  [APT Installation](#apt-installation)

---

## Core M3TAL Commands

These commands provide essential M3TAL functionality for launching the application, managing its headless mode, and checking system health.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application with a graphical user interface.

**Usage:**
```bash
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode, ideal for server environments without a graphical interface. You can optionally specify a port for the headless service.

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

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, `.env` file validity, and port availability.

**Usage:**
```bash
m3tal doctor
```

---

## Configuration Management

Manage M3TAL's environment variables and configuration files.

### `m3tal config wizard`

Launches an interactive wizard to configure the main M3TAL configuration file located at `/etc/m3tal/.env`.

**Usage:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in the `/etc/m3tal/.env` file.

**Usage:**
```bash
m3tal config set DOMAIN mym3tal.local
```

### `m3tal config get KEY`

Reads and displays the value of a single environment variable from `/etc/m3tal/.env`.

**Usage:**
```bash
m3tal config get DOMAIN
```

### `m3tal config scan`

Lists all environment variables across all configured M3TAL stacks, including those inherited from defaults.

**Usage:**
```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of the `/etc/m3tal/.env` file.

**Usage:**
```bash
m3tal config list
```

---

## Dashboard Management

Commands specifically for managing the M3TAL dashboard container.

### `m3tal dashpass [username] [password]`

Updates the dashboard user password. If `username` and `password` arguments are omitted, the command will prompt for them interactively.

**Usage (interactive):**
```bash
m3tal dashpass
```

**Usage (with arguments):**
```bash
m3tal dashpass admin new_secure_password
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This command respects the `DASHBOARD_EXPOSE_MODE` setting in your `.env` file.

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

Commands for managing all your deployed M3TAL stacks.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command starts all your configured services.

**Usage:**
```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all configured stacks, stopping all services managed by M3TAL.

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

M3TAL's backend API daemon is managed by `systemd`. You can interact with it using `systemctl` and `journalctl`.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

**Usage:**
```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. Use `Ctrl+C` to exit the stream.

**Usage:**
```bash
journalctl -u m3tal-api -f
```

---

## Direct Docker Compose Commands (Fallback)

In situations where M3TAL CLI commands might not be sufficient, you can directly use Docker Compose commands. M3TAL orchestrates these commands from the `/docker/` directory.

**Note:** Always ensure you are in the `/docker/` directory or specify the correct compose file paths.

### Starting all services:
```bash
docker compose up -d
```

### Stopping all services:
```bash
docker compose down
```

### Starting specific stack (e.g., `routing-compose.yml`):
```bash
docker compose -f routing-compose.yml up -d
```

### Stopping specific stack (e.g., `routing-compose.yml`):
```bash
docker compose -f routing-compose.yml down
```

### Viewing logs for a specific service (e.g., `m3tal-dashboard`):
```bash
docker compose logs -f m3tal-dashboard
```

---

## APT Installation

This section provides the exact commands required to install M3TAL via APT.

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```
```