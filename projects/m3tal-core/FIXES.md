DocCritic Audit Report: M3TAL Platform README.md

**Verdict:** FAILED - The README contains critical inconsistencies regarding core service communication that would prevent a successful deployment.

---

### Issue List

1.  **BLOCKER: API Daemon Port Inconsistency**
    *   **Description:** The documentation provides conflicting information regarding the M3TAL API daemon's listening port.
        *   The "System Architecture" section explicitly states: "API daemon (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port **8080**."
        *   The "Port Map" table confirms this: "8080 | M3TAL API daemon (Go) | Host-local".
        *   The "Dashboard Access" section, when describing the dashboard container, states: "It communicates with the M3TAL API daemon at `http://host.docker.internal:8080`."
        *   However, the "Environment Variables" section lists `API_PORT` with a default of `5050`, describing it as "Port for the M3TAL API daemon."
        *   Furthermore, the GROUND TRUTH reveals that the `m3tal-dashboard` container's `GO_API_URL` defaults to `http://host.docker.internal:5050`.
    *   **Impact:** This inconsistency will prevent the M3TAL Dashboard from successfully communicating with the API daemon, rendering the system inoperable immediately after deployment. A user attempting to configure `API_PORT` based on the environment variables list would misconfigure the system or find the dashboard non-functional.
    *   **Required Fixes:**
        *   **Highest Priority:** Reconcile the actual API daemon listening port (likely 8080 based on multiple consistent facts) with the dashboard's expected connection port and the `API_PORT` environment variable.
        *   Update the `API_PORT` default value in the "Environment Variables" section of the README to reflect the correct, consistent port (e.g., `8080`).
        *   Ensure the `m3tal-dashboard` container's `GO_API_URL` (in its `m3tal-compose.yml`) is correctly configured to match the API daemon's actual listening port.

2.  **WARNING: Ambiguous `HTTP_PORT` Environment Variable**
    *   **Description:** The `HTTP_PORT` environment variable is listed with a default value of `8080` in the "Environment Variables" section. However, its description states: "Not directly used by M3TAL core, typically Traefik's 80 or other application ports."
    *   **Impact:** This creates confusion, especially since `8080` is the actual host port used by the M3TAL API daemon (as identified in Issue 1). A user might mistakenly try to configure `HTTP_PORT` believing it affects the API daemon, or be confused why a "not used" variable shares a critical port number with a core component.
    *   **Required Fixes:**
        *   Either clarify the precise purpose of `HTTP_PORT` and how it might be used by non-core components, or remove it from the documentation if it truly has no relevance to M3TAL's core operation.
        *   If it *is* meant to be configurable for something, ensure its description and default value do not conflict or cause confusion with the M3TAL API's port.

3.  **SUGGESTION: Incomplete Quick Demo for First-Time Users**
    *   **Description:** The "Quick Demo" section provides steps to start the dashboard (`m3tal dash up`) or all stacks (`m3tal up`), assuming environment variables are already configured (e.g., `DASHBOARD_EXPOSE_MODE`). It does not include the initial configuration step using `m3tal config wizard`, which is stated elsewhere (`/etc/m3tal/.env | Managed by m3tal config wizard`) as crucial for setting up the primary configuration file.
    *   **Impact:** A new user following the "Quick Demo" might encounter issues if their system lacks the necessary `.env` configuration, leading to a less "quick" or frustrating first experience.
    *   **Required Fixes:**
        *   Enhance the "Quick Demo" to include the `m3tal config wizard` step as an essential part of the initial setup, perhaps as step 1 after installation, before running `m3tal dash up` or `m3tal up`. This would provide a more complete "quick start from scratch" experience.