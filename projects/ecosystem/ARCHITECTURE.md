# M3TAL Ecosystem Architecture Vision

M3TAL is built from the ground up to follow a modular, multi-repo design model. This page explains the architectural separation of concerns, global directory layout, and design conventions of the platform.

---

## 1. Modular Architecture Strategy
To prevent code duplication, version locking, and bloated dependencies, the platform is divided into independent, single-purpose components:

* **Engine & Logic (`m3tal-core`)**: The system core engine that interacts with the OS (systemd, Docker daemon). No static web assets live here.
* **Unified Documentation (`m3tal-docs`)**: The single source of truth for all manuals, API reference keys, and config parameters.
* **Plugin Catalogs (`m3tal-plugin-page`)**: A showcase registry page that serves a machine-readable JSON listing (`plugins.json`) to control plugins dynamically.
* **Installation Packages (`m3tal-apt-key`)**: An APT package server containing public GPG keys, dist package indices, and helper installation scripts.
* **Global Router Portal (`jakej985-rgb.github.io`)**: The entry-point homepage and router for the entire ecosystem.

---

## 2. Directory Path Standardization
Every project site (Portal, Docs, Plugins, Install) adheres to a uniform structure for consistency and scalability:

```text
/
├── index.html
├── projects/
│   ├── m3tal-core/     # Core system routes/details
│   ├── plugins/        # Plugin guides/showcases
│   ├── apt/            # Apt package info
│   └── ecosystem/      # Global project definitions
└── assets/             # Images, global css, global js
```

---

## 3. Integrated Routing Flow
The web ecosystem operates under the same parent domain using sub-folders (GitHub Pages layout):

```text
   [ Portal Homepage ] (https://jakej985-rgb.github.io/)
           │
     ┌─────┼─────────────────────┐
     ▼     ▼                     ▼
  [ Docs ] [ Plugins Catalog ] [ Installation ]
  (/m3tal-docs/) (/m3tal-plugin-page/) (/m3tal-apt-key/)
```
By sharing a common origin and styling system via `/assets/css/m3tal.css` and `/assets/js/m3tal.js`, the platform feels like a single unified console, while remaining fully decoupled at the git level.
