# M3TAL Command Reference

This document serves as a comprehensive cheat-sheet for all available M3TAL CLI commands.

---

## Core M3TAL Commands

These commands control the main M3TAL application and its core functionalities.

### `m3tal ui`

Launches the M3TAL Linux Desktop Application. This is the primary GUI interface for managing your M3TAL ecosystem.

**Usage:**
```bash
m3tal ui
```

---

### `m3tal headless [port]`

Starts M3TAL in headless mode, making it accessible over the network without launching the desktop application. This is ideal for home servers or remote management.

**Usage Example:**
```bash
m3tal headless 8080
```
*This starts M3TAL on port 8080. If `port` is omitted, it defaults to the configured API port.*

---

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This command is useful for monitoring the health and performance of your M3TAL host.

**Usage:**
```bash
m3tal stats
```

---

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, the validity of your `.env` configuration, and checks for port availability to ensure smooth operation.

**Usage:**
```bash
m3tal doctor
```

---

## Configuration Management Commands

These commands allow you to manage M3TAL's configuration settings stored in `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to guide you through the configuration of your `/etc/m3tal/.env` file. This is the recommended way to set up M3TAL for the first time or to make significant configuration changes.

**Usage:**
```bash
m3tal config wizard
```

---

### `m3tal config set KEY VALUE`

Sets a single environment variable in your `/etc/m3tal/.env` file.

**Usage Example:**
```bash
m3tal config set DOMAIN mym3tal.local
```
*This command sets the `DOMAIN` environment variable to `mym3tal.local`.*

---

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from your `/etc/m3tal/.env` file.

**Usage Example:**
```bash
m3tal config get DASHBOARD_PORT
```
*This command will output the current value of the `DASHBOARD_PORT` environment variable.*

---

### `m3tal config scan`

Lists all environment variables across all stacks managed by M3TAL, including defaults and currently set values.

**Usage:**
```bash
m3tal config scan
```

---

### `m3tal config list`

Lists the current contents of your `/etc/m3tal/.env` file, displaying all configured environment variables.

**Usage:**
```bash
m3tal config list
```

---

## Dashboard Management Commands

These commands are specifically for managing the M3TAL Dashboard container.

### `m3tal dashpass [username] [password]`

Updates the dashboard user password. If `username` and `password` are omitted, the command becomes interactive, prompting you for the username and new password.

**Usage Examples:**
```bash
m3tal dashpass admin new_secure_password
```
*This command sets the password for the `admin` user to `new_secure_password`.*

```bash
m3tal dashpass
```
*This command will interactively prompt you for the username and password.*

---

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you have the most up-to-date dashboard setup.

**Usage:**
```bash
m3tal dash up
```

---

### `m3tal dash down`

Stops and removes the dashboard container.

**Usage:**
```bash
m3tal dash down
```

---

### `m3tal dash restart`

Restarts the dashboard container.

**Usage:**
```bash
m3tal dash restart
```

---

### `m3tal dash logs`

Streams the logs from the dashboard container in real-time. This is invaluable for debugging issues with the dashboard.

**Usage:**
```bash
m3tal dash logs
```

---

### `m3tal dash status`

Shows the current status of the dashboard container (e.g., running, exited).

**Usage:**
```bash
m3tal dash status
```

---

## Stack Management Commands

These commands manage the Docker Compose stacks deployed within your M3TAL environment.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in the `/docker/` directory. This command brings up all your deployed services.

**Usage:**
```bash
m3tal up
```

---

### `m3tal down`

Runs `docker compose down` across all stacks managed by M3TAL. This command stops and removes all deployed services.

**Usage:**
```bash
m3tal down
```

---

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks to your terminal. This provides a centralized view of your system's logs.

**Usage:**
```bash
m3tal logs
```

---

## Systemd Service Management

The M3TAL API daemon runs as a systemd service. You can manage it using the `systemctl` command.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

**Usage:**
```bash
systemctl status m3tal-api
```

---

### `journalctl -u m3tal-api -f`

Streams the logs from the `m3tal-api.service` in real-time. This is the equivalent of `m3tal logs` for the API daemon.

**Usage:**
```bash
journalctl -u m3tal-api -f
```

---

## Docker Compose Fallback Commands

In situations where direct `m3tal` commands might not be sufficient, you can always interact with Docker Compose directly. M3TAL utilizes Docker Engine and Docker Compose V2.

*Note: These commands assume you are in the directory containing the relevant `compose.yml` files or have the context set correctly.*

### `docker compose up -d`

Starts all services defined in `docker-compose.yml` files in detached mode.

**Usage Example (from `/docker` directory):**
```bash
docker compose -f m3tal-compose.yml -f m3tal-compose.local.yml up -d
```
*This brings up the dashboard using the local override.*

---

### `docker compose down`

Stops and removes containers, networks, and images for services defined in `docker-compose.yml` files.

**Usage Example (from `/docker` directory):**
```bash
docker compose down
```

---

### `docker compose ps`

Lists the containers for the current project.

**Usage Example:**
```bash
docker compose ps
```

---

### `docker compose logs -f [service_name]`

Streams logs from a specific service.

**Usage Example:**
```bash
docker compose logs -f m3tal-dashboard
```

---

### `docker compose pull [service_name]`

Pulls the latest images for specified services.

**Usage Example:**
```bash
docker compose pull m3tal-dashboard
```

---
