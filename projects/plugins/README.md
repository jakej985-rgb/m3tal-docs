# M3TAL Plugin System

M3TAL implements an active, dynamic plugin architecture allowing you to extend the system core with pre-configured stacks, custom ingress routing middleware, or full backend components.

The plugins catalog is loaded directly from a centralized JSON document and can be queried and toggled programmatically.

---

## Plugin Kinds
Plugins are categorized based on their roles in the ecosystem:
1. **Stack**: A complete multi-container Docker Compose service setup (e.g. `ai` stack, `maintenance` monitoring tools).
2. **Middleware**: Traefik-compatible ingress routing middleware configuration presets (e.g. rate-limiting, basic-auth headers).
3. **Route**: Custom application ingress configurations.
4. **Traefik**: Central reverse-proxy gateway definitions.

---

## Machine-Readable Catalog
The catalog API page serves a dynamic JSON array named `plugins.json` located at the root of the plugins site. Both the dashboard UI and the CLI interface pull from this registry during updates.

Example entry:
```json
{
  "name": "ai",
  "kind": "Stack",
  "description": "Modular AI Addon (Ollama + env profiles)",
  "version": "1.0.0",
  "author": "M3TAL Team",
  "url": "https://raw.githubusercontent.com/jakej985-rgb/m3tal-core/main/deploy/plugins/stacks/ai.yml",
  "composeUrl": "https://raw.githubusercontent.com/jakej985-rgb/m3tal-core/main/deploy/stack/ai-compose.yml"
}
```

---

## Next Steps
* To understand how to construct and submit plugins, see the [Plugin Development Guide](#/plugins/DEVELOPMENT).
* Discover existing options on the [Plugin Catalog Page](/m3tal-plugin-page/).
