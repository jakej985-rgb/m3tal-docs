# M3TAL Environment Variables Reference

All M3TAL configuration is managed via environment variables, primarily read from `/etc/m3tal/.env` by both the CLI and all Docker Compose stacks. The `m3tal config wizard` and `m3tal config set` commands are the recommended ways to manage these variables.

**Quick Reference Table:**

| Variable Name | Default Value | Description |
|---------------|---------------|-------------|
| **Core** | | |
| `API_PORT` | `5050` | Port for the M3TAL API daemon. |
| `DASHBOARD_PORT` | `8082` | Port for the M3TAL Dashboard. |
| `DASHBOARD_EXPOSE_MODE` | `local` | Controls how the dashboard is exposed (local or traefik). |
| `HTTP_PORT` | `8080` | General HTTP port, often used internally. |
| `STATE_DIR` | `./state` | Directory for M3TAL state files. |
| `LOG_LEVEL` | `info` | Logging verbosity for M3TAL services. |
| `DASHBOARD_SECRET` | `change_me_immediately` | Secret for dashboard authentication (auto-generated on init). |
| `API_TOKEN` | `change_me_api_token` | Token for API authentication (auto-generated on init). |
| `ADMIN_PASSWORD` | `admin_pass` | Password for the default admin user. |
| `PUID` | `1000` | User ID for Docker container processes. |
| `PGID` | `1000` | Group ID for Docker container processes. |
| `TZ` | `America/Denver` | Timezone for M3TAL services. |
| `DEBUG_MODE` | `false` | Enables debug mode for M3TAL services. |
| `METRICS_ENABLED` | `true` | Enables Prometheus metrics collection. |
| **Auth** | | |
| `DASHBOARD_SECRET` | `change_me_immediately` | Secret for dashboard authentication. **This is auto-generated on first `m3tal init` and should not be set manually unless rotating.** |
| `API_TOKEN` | `change_me_api_token` | Token for API authentication. **This is auto-generated on first `m3tal init` and should not be set manually unless rotating.** |
| `ADMIN_PASSWORD` | `admin_pass` | Password for the default admin user. |
| **Network** | | |
| `NETWORK_NAME` | `m3tal` | The name of the Docker network M3TAL services connect to. |
| `LOCAL_IP` | `127.0.0.1` | Local IP address. |
| `DOMAIN` | `localhost` | Base domain for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. |
| `VPN_USER` | `user` | Username for VPN connection. |
| `VPN_PASSWORD` | `password` | Password for VPN connection. |
| **Storage** | | |
| `BASE_STORAGE_PATH` | `./data` | **Controls where media data is stored. Defaults to `/mnt` in production deployments, not `./data` as in the template.** |
| `MEDIA_PATH` | `./data/media` | Path for media storage, relative to `BASE_STORAGE_PATH`. |
| `CONFIG_PATH` | `./data/config` | Path for configuration files, relative to `BASE_STORAGE_PATH`. |
| `DOWNLOADS_PATH` | `./data/downloads` | Path for downloaded files, relative to `BASE_STORAGE_PATH`. |
| `STATE_DIR` | `./state` | Directory for M3TAL state files. |
| **Traefik** | | |
| `TRAEFIK_WEB_PORT` | `80` | The host port Traefik listens on for HTTP traffic. |
| `TRAEFIK_WEBHTTPS_PORT` | `443` | The host port Traefik listens on for HTTPS traffic. |
| `TRAEFIK_DASHBOARD_PORT` | `8080` | The host port for the Traefik dashboard (accessible via `127.0.0.1:8081`). |
| `DOMAIN` | `localhost` | Base domain for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. |
| **VPN** | | |
| `VPN_USER` | `user` | Username for VPN connection. |
| `VPN_PASSWORD` | `password` | Password for VPN connection. |
| **System** | | |
| `PUID` | `1000` | User ID for Docker container processes. |
| `PGID` | `1000` | Group ID for Docker container processes. |
| `TZ` | `America/Denver` | Timezone for M3TAL services. |

---

## Variable Breakdown

All environment variables are read from `/etc/m3tal/.env` by both the CLI and all compose stacks via `--env-file`.

### Core

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `API_PORT` | The port on which the M3TAL API daemon listens. | `5050` | `5050` | API daemon, Dashboard |
| `DASHBOARD_PORT` | The port on which the M3TAL Dashboard runs. | `8082` | `8082` | Dashboard |
| `DASHBOARD_EXPOSE_MODE` | Controls how the dashboard is exposed. `local` exposes it directly via a port binding, while `traefik` uses Traefik routing. | `local` | `traefik` | Dashboard, CLI |
| `HTTP_PORT` | A general HTTP port, often used internally for service communication. | `8080` | `8080` | API daemon, Traefik |
| `STATE_DIR` | The directory where M3TAL stores its state database and other persistent files. | `./state` | `/var/lib/m3tal/state` | API daemon, Dashboard |
| `LOG_LEVEL` | The verbosity of logging for M3TAL services. | `info` | `debug` | API daemon, Dashboard |
| `DASHBOARD_SECRET` | A secret used for securing the dashboard. **This is auto-generated on first `m3tal init` and should NOT be set manually unless you are rotating it.** | `change_me_immediately` | `s3cr3t_d4sh_k3y_r0t4t3d` | Dashboard |
| `API_TOKEN` | A token used for authenticating with the M3TAL API. **This is auto-generated on first `m3tal init` and should NOT be set manually unless you are rotating it.** | `change_me_api_token` | `4p1_t0k3n_g3n3r4t3d_0n_1n1t` | API daemon, Dashboard |
| `ADMIN_PASSWORD` | The password for the default administrator user of the M3TAL Dashboard. | `admin_pass` | `my_super_secure_password` | Dashboard |
| `PUID` | The User ID to run Docker container processes as. | `1000` | `1000` | Dashboard |
| `PGID` | The Group ID to run Docker container processes as. | `1000` | `1000` | Dashboard |
| `TZ` | The timezone to be used by M3TAL services. | `America/Denver` | `UTC` | Dashboard |
| `DEBUG_MODE` | Enables debug mode for M3TAL services, providing more verbose output. | `false` | `true` | API daemon, Dashboard |
| `METRICS_ENABLED` | Enables the collection and exposure of Prometheus metrics. | `true` | `false` | API daemon |

### Auth

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `DASHBOARD_SECRET` | A secret used for securing the dashboard. **This is auto-generated on first `m3tal init` and should NOT be set manually unless you are rotating it.** | `change_me_immediately` | `s3cr3t_d4sh_k3y_r0t4t3d` | Dashboard |
| `API_TOKEN` | A token used for authenticating with the M3TAL API. **This is auto-generated on first `m3tal init` and should NOT be set manually unless you are rotating it.** | `change_me_api_token` | `4p1_t0k3n_g3n3r4t3d_0n_1n1t` | API daemon, Dashboard |
| `ADMIN_PASSWORD` | The password for the default administrator user of the M3TAL Dashboard. | `admin_pass` | `my_super_secure_password` | Dashboard |

### Network

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `NETWORK_NAME` | The name of the Docker network that M3TAL services will connect to. | `m3tal` | `m3tal-net` | CLI, Docker Compose |
| `LOCAL_IP` | The local IP address of the host machine. | `127.0.0.1` | `192.168.1.100` | Traefik (internal routing) |
| `DOMAIN` | The base domain name used for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. | `localhost` | `m3tal.local` | Traefik, API daemon (via Traefik config) |
| `VPN_USER` | The username for your VPN connection. | `user` | `myvpnuser` | VPN Client |
| `VPN_PASSWORD` | The password for your VPN connection. | `password` | `mysecurevpnpassword` | VPN Client |

### Storage

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `BASE_STORAGE_PATH` | **Controls where media data is stored. Defaults to `/mnt` in production deployments, not `./data` as in the template.** | `./data` | `/mnt` | Dashboard, CLI |
| `MEDIA_PATH` | The path within `BASE_STORAGE_PATH` where media files are stored. | `./data/media` | `/mnt/media` | Dashboard |
| `CONFIG_PATH` | The path within `BASE_STORAGE_PATH` where configuration files are stored. | `./data/config` | `/mnt/config` | Dashboard |
| `DOWNLOADS_PATH` | The path within `BASE_STORAGE_PATH` where downloaded files are stored. | `./data/downloads` | `/mnt/downloads` | Dashboard |
| `STATE_DIR` | The directory where M3TAL stores its state database and other persistent files. | `./state` | `/var/lib/m3tal/state` | API daemon, Dashboard |

### Traefik

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `TRAEFIK_WEB_PORT` | The host port on which Traefik listens for incoming HTTP traffic. | `80` | `80` | Traefik |
| `TRAEFIK_WEBHTTPS_PORT` | The host port on which Traefik listens for incoming HTTPS traffic. | `443` | `443` | Traefik |
| `TRAEFIK_DASHBOARD_PORT` | The host port to access the Traefik dashboard. Note: This is typically exposed on `127.0.0.1:8080` by default, accessible as `http://127.0.0.1:8081`. | `8080` | `8080` | Traefik |
| `DOMAIN` | The base domain name used for Traefik routing. Setting this enables `dash.DOMAIN` and `api.DOMAIN` routes. | `localhost` | `m3tal.local` | Traefik, API daemon (via Traefik config) |

### VPN

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `VPN_USER` | The username for your VPN connection. | `user` | `myvpnuser` | VPN Client |
| `VPN_PASSWORD` | The password for your VPN connection. | `password` | `mysecurevpnpassword` | VPN Client |

### System

| Name | Description | Default Value | Example Value | Used By |
|---|---|---|---|---|
| `PUID` | The User ID to run Docker container processes as. This is important for file permissions. | `1000` | `1000` | Dashboard |
| `PGID` | The Group ID to run Docker container processes as. This is important for file permissions. | `1000` | `1000` | Dashboard |
| `TZ` | The timezone to be used by M3TAL services. This ensures logs and timestamps are accurate. | `America/Denver` | `UTC` | Dashboard |