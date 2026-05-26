# Plugin Development Guide

This guide describes how to construct a M3TAL-compatible plugin and add it to the ecosystem registry.

---

## 1. Directory Structure
A typical plugin definition is written in YAML (for Docker Compose stacks or Traefik middleware). Stacks should include the main services declaration and label definitions.

```text
my-plugin/
├── README.md               # User documentation
├── my-plugin.yml           # Plugin manifest (ingress labels/routing)
└── my-plugin-compose.yml   # Docker Compose services (required for 'Stack' kind)
```

---

## 2. Ingress Router Labels
If your plugin includes a web service that needs to be exposed via Traefik, declare the routing parameters within the labels:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myplugin.rule=Host(`myplugin.${DOMAIN}`)"
  - "traefik.http.services.myplugin.loadbalancer.server.port=80"
  - "traefik.docker.network=proxy"
```

---

## 3. Registering in the Catalog
To add the plugin to the ecosystem database, write a definition block and submit a pull request to append it to `plugins.json` in the `m3tal-plugin-page` repository:

```json
{
  "name": "my-plugin",
  "kind": "Stack",
  "description": "Short summary of what my plugin does",
  "version": "1.0.0",
  "author": "Your Name",
  "url": "https://raw.githubusercontent.com/username/repo/main/my-plugin.yml",
  "composeUrl": "https://raw.githubusercontent.com/username/repo/main/my-plugin-compose.yml"
}
```

---

## 4. Local Installation and Testing
You can manually load and dry-run your custom Compose file through the CLI before submitting:

```bash
# 1. Place the compose yml in the stacks folder
sudo cp my-plugin-compose.yml /docker/my-plugin-compose.yml

# 2. Run the validator
m3tal config scan

# 3. Bring the stack up
m3tal up
```
Once verified, submit your plugin to the catalog to make it installable with `m3tal plugin install my-plugin`.
