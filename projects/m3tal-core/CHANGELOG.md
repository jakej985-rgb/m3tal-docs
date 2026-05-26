# Changelog

All notable changes to the M3TAL core engine will be documented in this file.

---

## [2.0.0] - 2026-05-26
### Added
* SQLite state storage database engine (`state.db`) to persist configuration data.
* Dynamic reverse proxy routing APIs with auto-label generators for Traefik.
* Integrated Compose file lint validator and automated repair tool.
* Unified Agent Platform plugin management system to query catalogs and install add-ons.
* VPN gateway manager with regions, status monitors, and leak checks.

### Changed
* Transitioned core CLI commands to use the v2 SQLite engine API.
* Unified system components monitoring and history metrics.

---

## [1.0.0] - 2025-11-12
### Added
* Initial release of the M3TAL system.
* Go CLI framework (`m3tal`) with interactive TUI menu panels.
* Background daemon manager (`m3tal-api`) executing as a systemd service.
* Dashboard container console built on Flask with users credentials management.
* Out-of-the-box Traefik ingress gateway with Docker networking layers.
