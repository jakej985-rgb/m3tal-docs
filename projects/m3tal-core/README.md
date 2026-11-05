# M3TAL Media Server

Autonomous, self-healing media automation platform.

## 🏗 Architecture (Source Services)
| Service | Type | Path |
|---------|------|------|
| m3tal-api | API binary | [api/cmd](./api/cmd) |
| events | Core package | [core/events](./core/events) |
| services | Core package | [core/services](./core/services) |
| state | Core package | [core/state](./core/state) |
| system | Core package | [core/system](./core/system) |
| models | Shared package | [pkg/models](./pkg/models) |
| plugins | Shared package | [pkg/plugins](./pkg/plugins) |
| system | Shared package | [pkg/system](./pkg/system) |
| version | Shared package | [pkg/version](./pkg/version) |
| middleware | API package | [api/middleware](./api/middleware) |
| gui | Desktop UI | [gui](./gui) |
| github.com/jakej985-rgb/m3tal-core | Go module | [go.mod](./go.mod) |
## ⚙️ Environment Configuration
| Variable | Default |
|----------|---------|
| API\_PORT | `5050` |
| API\_TOKEN | `N/A` |
| M3TAL\_STATE\_DB | `N/A` |
| M3TAL\_PLUGINS\_DIR | `N/A` |

## 🚀 Deployment

```bash
python m3tal.py init
```
