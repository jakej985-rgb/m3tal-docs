DocCritic Audit Report for M3TAL Platform README.md

**Verdict: FAILED**
The README contains critical inconsistencies regarding core component communication ports which would prevent successful deployment and operation of the M3TAL system.

---

### Issue List

**1. Issue: Inconsistent API Port Information & Dashboard Connectivity Failure**
*   **Classification:** BLOCKER
*   **Description:** The README provides conflicting information about the M3TAL API daemon's listening port.
    *   The "Core Components" section states the API daemon listens on host-local port `8080`.
    *   The "Port Map" table confirms `8080` for the M3TAL API daemon.
    *   The "Configuration" section's Environment Variables table lists `HTTP_PORT` with a default of `8080` as "The M3TAL API daemon's listening port."
    *   However, the same table also lists `API_PORT` with a default of `5050` as "Internal API port." This introduces ambiguity.
    *   Crucially, the GROUND TRUTH reveals that the `m3tal-dashboard` container's `GO_API_URL` environment variable defaults to `http://host.docker.internal:5050`. This means the dashboard will attempt to connect to the API on port `5050` while the API daemon is listening on `8080`. This will result in a complete failure of the dashboard to communicate with the API.
*   **Required Fixes:**
    1.  **System Configuration (GROUND TRUTH):**
        *   Modify `m3tal-compose.yml`: Change `GO_API_URL=${GO_API_URL:-http://host.docker.internal:5050}` to `GO_API_URL=${GO_API_URL:-http://host.docker.internal:8080}` to align the dashboard's default API target with the API daemon's actual listening port.
        *   Re-evaluate the purpose of `API_PORT=5050` in `/etc/m3tal/.env`. If it's unused or redundant with `HTTP_PORT`, remove it from the `.env.example`. If it has a distinct, critical purpose, clarify that purpose and ensure it's used consistently.
    2.  **Documentation (README):**
        *   Remove the `API_PORT` entry from the "Environment Variables" table if it is removed from the system configuration.
        *   If `API_PORT` is retained (after clarification in system config), update its description to clearly differentiate it from `HTTP_PORT` and explain its specific role, ensuring no contradiction. The README must reflect a consistent system.

**2. Issue: Misleading `TRAEFIK_DASHBOARD_PORT` Environment Variable**
*   **Classification:** WARNING
*   **Description:** The "Configuration" section's Environment Variables table lists `TRAEFIK_DASHBOARD_PORT` with a default of `8080` and the description "Internal Traefik dashboard port (host-local `8081`)." While the README attempts to clarify the difference between internal and host ports, the `TRAEFIK_DASHBOARD_PORT` variable in the `/etc/m3tal/.env` (and documented in the README) does not actually control the *host-facing access port* of `8081`. The GROUND TRUTH `routing-compose.yml` explicitly hardcodes `127.0.0.1:8081:8080`, meaning the host port `8081` is not configurable via this environment variable. This can lead to user confusion if they attempt to change `TRAEFIK_DASHBOARD_PORT` expecting to alter the external access port.
*   **Required Fixes:**
    1.  **System Configuration (GROUND TRUTH):**
        *   If the intent is for the host port `8081` to be configurable, modify `routing-compose.yml` to use an environment variable (e.g., `"${TRAEFIK_DASHBOARD_HOST_PORT:-8081}:8080"`) and add `TRAEFIK_DASHBOARD_HOST_PORT=8081` to `.env.example`.
        *   If the host port will remain hardcoded at `8081`, then the `TRAEFIK_DASHBOARD_PORT` variable (referring to the internal port `8080`) should be renamed (e.g., `TRAEFIK_DASHBOARD_INTERNAL_PORT`) to clearly separate it from the host access port, or removed if it serves no configuration purpose.
    2.  **Documentation (README):**
        *   Update the "Environment Variables" table to reflect any changes in the system configuration (new variables, renamed variables).
        *   If the host port remains hardcoded, clarify the `TRAEFIK_DASHBOARD_PORT` description to explicitly state: "This variable specifies the *internal container port* for the Traefik dashboard (`8080`). The host-facing access port is hardcoded to `8081` in `routing-compose.yml` and is not controlled by this variable."

---
The remaining audit criteria passed without issues. The documentation is otherwise comprehensive and well-structured, but the identified BLOCKER issue is critical for system functionality.