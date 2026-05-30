# M3TAL Media Server

Autonomous, self-healing media automation platform.

## 🏗 Architecture (Source Services)
| Service | Type | Path |
|---------|------|------|
| m3tal (cli) | CLI binary | [cli](./cli) |
| m3tal-api | API binary | [api/cmd](./api/cmd) |
| agents | Core package | [core/agents](./core/agents) |
| auth | Core package | [core/auth](./core/auth) |
| compose | Core package | [core/compose](./core/compose) |
| config | Core package | [core/config](./core/config) |
| containers | Core package | [core/containers](./core/containers) |
| doctor | Core package | [core/doctor](./core/doctor) |
| engine | Core package | [core/engine](./core/engine) |
| events | Core package | [core/events](./core/events) |
| health | Core package | [core/health](./core/health) |
| networking | Core package | [core/networking](./core/networking) |
| orchestrator | Core package | [core/orchestrator](./core/orchestrator) |
| plugins | Core package | [core/plugins](./core/plugins) |
| preflight | Core package | [core/preflight](./core/preflight) |
| queue | Core package | [core/queue](./core/queue) |
| registry | Core package | [core/registry](./core/registry) |
| routing | Core package | [core/routing](./core/routing) |
| services | Core package | [core/services](./core/services) |
| state | Core package | [core/state](./core/state) |
| system | Core package | [core/system](./core/system) |
| client | Shared package | [pkg/client](./pkg/client) |
| cmdutil | Shared package | [pkg/cmdutil](./pkg/cmdutil) |
| config | Shared package | [pkg/config](./pkg/config) |
| models | Shared package | [pkg/models](./pkg/models) |
| output | Shared package | [pkg/output](./pkg/output) |
| system | Shared package | [pkg/system](./pkg/system) |
| middleware | API package | [api/middleware](./api/middleware) |
| tui | Terminal UI | [tui](./tui) |
| webui | Web UI | [webui](./webui) |
| gui | Desktop UI | [gui](./gui) |
| plugins | Deploy artifact | [deploy/plugins](./deploy/plugins) |
| stack | Deploy artifact | [deploy/stack](./deploy/stack) |
| github.com/jakej985-rgb/m3tal-core | Go module | [go.mod](./go.mod) |

## 🐳 Infrastructure Stacks
### Ai Stack
- **ollama** (ollama/ollama:latest)
  - Ports: 11434:11434

### M3tal Stack
- **m3tal-dashboard** (ghcr.io/jakej985-rgb/m3tal-godash:debug)

### Routing Stack
- **traefik** (traefik:latest)
  - Ports: ${TRAEFIK_WEB_PORT:-80}:80, ${TRAEFIK_WEBHTTPS_PORT:-443}:443, 127.0.0.1:8081:8080
- **cloudflared** (cloudflare/cloudflared:latest)

## ⚙️ Environment Configuration
| Variable | Default |
|----------|---------|
| DASHBOARD\_PORT | `8082` |
| DASHBOARD\_EXPOSE\_MODE | `local` |
| HTTP\_PORT | `8080` |
| STATE\_DIR | `./state` |
| LOG\_LEVEL | `info` |
| DASHBOARD\_SECRET | `change\_me\_immediately` |
| API\_TOKEN | `change\_me\_api\_token` |
| ADMIN\_PASSWORD | `admin\_pass` |
| NETWORK\_NAME | `m3tal` |
| LOCAL\_IP | `127.0.0.1` |
| DOMAIN | `localhost` |
| VPN\_USER | `user` |
| VPN\_PASSWORD | `password` |
| BASE\_STORAGE\_PATH | `./data` |
| MEDIA\_PATH | `./data/media` |
| CONFIG\_PATH | `./data/config` |
| DOWNLOADS\_PATH | `./data/downloads` |
| PUID | `1000` |
| PGID | `1000` |
| TZ | `America/Denver` |
| TRAEFIK\_WEB\_PORT | `80` |
| TRAEFIK\_WEBHTTPS\_PORT | `443` |
| TRAEFIK\_DASHBOARD\_PORT | `8080` |
| DEBUG\_MODE | `false` |
| METRICS\_ENABLED | `true` |

## 🚀 Deployment

```bash
python m3tal.py init
```
