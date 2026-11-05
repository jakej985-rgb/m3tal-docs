DocCritic Audit Report for M3TAL Platform README.md

**Verdict: FAILED**

**Reason:** The README contains a critical contradiction regarding the M3TAL API daemon's listening port and how it relates to the `API_PORT` configuration variable. This factual discrepancy is a BLOCKER as it prevents users from correctly understanding and potentially configuring a core component of the system. All other essential audit criteria have been met.

---

**Issue List:**

1.  **BLOCKER: Contradictory Information on M3TAL API Daemon Port**
    *   **Description:** The README provides conflicting information regarding the M3TAL API daemon's listening port.
        *   The "Components" section states: "API daemon (`m3tal-api.service`): A Go binary running as a systemd service, listening on host-local port `8080`."
        *   The "Port Map" section lists: "8080 | M3TAL API daemon (Go) | Host-local".
        *   However, the "Configuration" section lists `API_PORT` with a default of `5050` and describes it as "The port the M3TAL API daemon listens on."
    *   **Impact:** A user will be confused about which port the API daemon truly listens on and whether/how to configure it using `API_PORT`. This directly impacts the ability to understand and operate a core system component.
    *   **Required Fixes:**
        *   Clarify the role and default value of `API_PORT`.
        *   If `API_PORT` controls the API daemon's listening port, then the default port in "Components" and "Port Map" should match `API_PORT`'s default (or explicitly state that `8080` is the default *if API_PORT is not set*).
        *   If `API_PORT` refers to a different port or component, its description in the "Configuration" table must be corrected to reflect its actual purpose, and the M3TAL API daemon's port (`8080`) should be presented as fixed or configured by an unlisted mechanism.
        *   Ensure consistency across all sections regarding the API daemon's port configuration.

---
**Summary of Passed Audit Criteria:**

*   **APT installation:** The README correctly shows the 3-command keyring+repo+install block.
*   **Docker dependency:** The README clearly states that Docker Engine + Docker Compose V2 are required.
*   **Deployment lifecycle:** The README explains how stacks work (`/docker` directory, `m3tal up` command, and how to add new compose files).
*   **Traefik routing:** The README explains that Traefik is the HTTP gateway and details how services get exposed (using labels and dynamic configuration).
*   **Port table:** The README includes a comprehensive table listing ports 80, 8080, 8081, and 8082 with descriptions.
*   **Service management:** The README explicitly mentions `systemctl` for managing `m3tal-api.service` and provides example commands.
*   **Firewall note:** The README includes a reminder for users to allow port 80 in their firewall (e.g., `ufw`).
*   **Tone:** The writing maintains a professional, technical, and informative tone throughout, avoiding marketing copy.
*   **Quick demo:** The README provides a "Quick Demo" section that outlines working steps to get started with the M3TAL Dashboard.