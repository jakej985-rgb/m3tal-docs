# CLI & Daemon API Reference

The M3TAL system has two main user-facing interfaces: the CLI client (`m3tal`) and the REST API Daemon (`m3tal-api`).

---

## 1. CLI Command Reference

### Interactive TUI Dashboard
Open the CLI Console interface:
```bash
sudo m3tal
```

### System Configuration
* **Initialize system configuration file:**
  ```bash
  m3tal init
  ```
* **Interactive setup wizard:**
  ```bash
  m3tal config wizard
  ```
* **Configure parameter value:**
  ```bash
  m3tal config set KEY value
  ```
* **Retrieve parameter value:**
  ```bash
  m3tal config get KEY
  ```
* **Print configuration list:**
  ```bash
  m3tal config list
  ```

### Stack Management
* **Start all container stacks:**
  ```bash
  m3tal up
  ```
* **Stop all container stacks:**
  ```bash
  m3tal down
  ```
* **View consolidated service logs:**
  ```bash
  m3tal logs
  ```

### Dashboard Settings
* **Start the dashboard console:**
  ```bash
  m3tal dash up
  ```
* **Stop the dashboard console:**
  ```bash
  m3tal dash down
  ```
* **Change dashboard admin password:**
  ```bash
  sudo m3tal dashpass
  ```

---

## 2. API Daemon REST Endpoint Reference
The `m3tal-api` daemon runs locally as a systemd service listening on `http://127.0.0.1:8080` (or `HTTP_PORT`). All calls must authenticate by sending the header `X-API-Token: YOUR_API_TOKEN` (or query string `?token=...`).

### System Status & Control
* **GET `/api/health`**: Returns system health status.
* **GET `/api/services`**: Returns Docker service containers states.
* **GET `/api/v2/system/metrics`**: Historical CPU, memory, disk, and uptime values.

### Route Settings (v2 Engine)
* **GET `/api/v2/routes`**: Lists all active reverse proxy routes.
* **POST `/api/v2/routes`**: Creates a new route record.
  * **Payload**:
    ```json
    {
      "service": "myapp",
      "domain": "myapp.localhost",
      "port": 80,
      "entrypoints": ["web"]
    }
    ```
* **DELETE `/api/v2/routes/{id}`**: Deletes a route record by database ID.
* **POST `/api/v2/routes/preview`**: Returns Traefik labels without saving them.

### Docker Stack Control (v2 Engine)
* **GET `/api/v2/stacks`**: Lists Docker compose stacks.
* **POST `/api/v2/stacks/{name}/up`**: Deploys/starts a stack.
* **POST `/api/v2/stacks/{name}/down`**: Stops/removes a stack.

### Plugin Management (v2 Engine)
* **GET `/api/v2/plugins`**: Lists active plugins.
* **GET `/api/v2/plugins/catalog`**: Lists available plugins from the remote catalog.
* **POST `/api/v2/plugins/enable`**: Enables a plugin.
* **POST `/api/v2/plugins/disable`**: Disables a plugin.
