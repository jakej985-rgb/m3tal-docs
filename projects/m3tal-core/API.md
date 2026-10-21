# M3TAL CLI Command Reference

This document provides a comprehensive overview of all available `m3tal` CLI commands, serving as a cheat-sheet for managing your M3TAL ecosystem.

## Core Commands

These commands are the primary interface for interacting with M3TAL.

### `m3tal` / `m3tal ui`

Launches the M3TAL Linux Desktop Application. This provides a graphical user interface for managing various aspects of M3TAL.

**Usage:**
```bash
m3tal
# or
m3tal ui
```

### `m3tal headless [port]`

Starts M3TAL in headless mode. This is ideal for running M3TAL on a server without a graphical interface. You can optionally specify a port for the API to listen on.

**Usage:**
```bash
m3tal headless
# or to specify a port:
m3tal headless 8080
```

### `m3tal stats`

Prints instant host system, GPU, thermal, and storage metrics to the terminal. This command provides a quick snapshot of your system's performance and health.

**Usage:**
```bash
m3tal stats
```

### `m3tal doctor`

Performs a pre-flight health check of your M3TAL installation. It verifies Docker connectivity, the validity of your `.env` configuration, and checks for port availability.

**Usage:**
```bash
m3tal doctor
```

## Configuration Management

These commands allow you to manage M3TAL's configuration settings, primarily stored in `/etc/m3tal/.env`.

### `m3tal config wizard`

Launches an interactive wizard to configure `/etc/m3tal/.env`. This is the recommended way to set up your initial M3TAL configuration.

**Usage:**
```bash
m3tal config wizard
```

### `m3tal config set KEY VALUE`

Sets a single environment variable in your `/etc/m3tal/.env` file.

**Usage:**
```bash
m3tal config set DASHBOARD_PORT 8082
```

### `m3tal config get KEY`

Reads and prints the value of a single environment variable from your `/etc/m3tal/.env` file.

**Usage:**
```bash
m3tal config get API_PORT
```

### `m3tal config scan`

Lists all environment variables across all configured M3TAL stacks. This helps in understanding the complete configuration landscape.

**Usage:**
```bash
m3tal config scan
```

### `m3tal config list`

Lists the current contents of your `/etc/m3tal/.env` file, showing all configured environment variables and their values.

**Usage:**
```bash
m3tal config list
```

## Dashboard Management

These commands specifically manage the M3TAL dashboard container.

### `m3tal dashpass [username] [password]`

Updates the dashboard user password. If the username and password are not provided as arguments, the command will prompt you interactively.

**Usage:**
```bash
m3tal dashpass admin my_secure_password
# or interactively:
m3tal dashpass
```

### `m3tal dash up`

Pulls the latest dashboard compose configuration from GitHub and then starts the dashboard container. This ensures you are running the most up-to-date dashboard version.

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

Streams the logs from the dashboard container in real-time. This is useful for debugging issues with the dashboard.

**Usage:**
```bash
m3tal dash logs
```

### `m3tal dash status`

Shows the current status of the dashboard container (e.g., running, stopped).

**Usage:**
```bash
m3tal dash status
```

## Stack Management

These commands manage all your M3TAL stacks defined in `/docker/`.

### `m3tal up`

Runs `docker compose up` across all `*-compose.yml` files found in `/docker/`. This starts all your defined M3TAL services and any custom stacks you have added.

**Usage:**
```bash
m3tal up
```

### `m3tal down`

Runs `docker compose down` across all M3TAL stacks. This stops and removes all containers, networks, and volumes defined in your compose files.

**Usage:**
```bash
m3tal down
```

### `m3tal logs`

Streams aggregated logs from all running M3TAL stacks to your terminal. This provides a centralized view of logs from all your services.

**Usage:**
```bash
m3tal logs
```

## Systemd Service Management

M3TAL's API daemon is managed by systemd. You can use `systemctl` to control and monitor its status.

### `systemctl status m3tal-api`

Checks the current status of the `m3tal-api.service`.

**Usage:**
```bash
systemctl status m3tal-api
```

### `journalctl -u m3tal-api -f`

Follows the logs of the `m3tal-api.service` in real-time. This is essential for debugging the API daemon.

**Usage:**
```bash
journalctl -u m3tal-api -f
```

## Docker Compose Fallback

In situations where the `m3tal` CLI might not be sufficient or for advanced control, you can directly use Docker Compose commands. M3TAL utilizes Docker Compose V2 under the hood and operates on compose files within the `/docker/` directory.

### `docker compose up` (for all stacks)

Equivalent to `m3tal up`, this command starts all services defined in `*-compose.yml` files within the `/docker/` directory.

**Usage:**
```bash
cd /docker/
docker compose up -d # -d for detached mode
```

### `docker compose down` (for all stacks)

Equivalent to `m3tal down`, this command stops and removes containers, networks, and volumes for all stacks.

**Usage:**
```bash
cd /docker/
docker compose down
```

### `docker compose ps` (for a specific stack)

Lists the containers for a specific stack defined in a compose file.

**Usage (example for routing stack):**
```bash
cd /docker/
docker compose -f routing-compose.yml ps
```

### `docker compose logs` (for a specific stack)

Streams logs for containers within a specific stack.

**Usage (example for routing stack):**
```bash
cd /docker/
docker compose -f routing-compose.yml logs -f # -f to follow logs
```

## Installation

To install M3TAL via APT:

```bash
# 1. Add the GPG signing key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the APT repository
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Install
sudo apt update && sudo apt install -y m3tal
```