## M3TAL README Audit

**Verdict: FAILED**

The README is missing critical information required for installation and operation, and contains several inaccuracies and omissions when compared to the GROUND TRUTH.

---

### Issue List:

1.  **BLOCKER: APT Installation Block**
    *   **Description:** The README does not provide the necessary three-command sequence (keyring, repo, install) for APT installation. This is a fundamental blocker for users who expect to install M3TAL via their package manager.
    *   **Required Fix:** Include the complete APT installation block as provided in the GROUND TRUTH.

2.  **BLOCKER: Docker Dependency Statement**
    *   **Description:** The README does not explicitly state that Docker Engine and Docker Compose V2 are required. While it mentions "Infrastructure Stacks" and uses Docker images, the explicit dependency is missing.
    *   **Required Fix:** Add a clear statement in the README that Docker Engine and Docker Compose V2 are prerequisites for running M3TAL.

3.  **BLOCKER: Deployment Lifecycle Explanation**
    *   **Description:** The README's deployment section is insufficient. It mentions `python m3tal.py init` but does not explain how M3TAL orchestrates services via Docker, the role of the `/docker` directory (symlinked to `/opt/m3tal/stack/`), how `m3tal up` works across Compose files, or how to add new compose files.
    *   **Required Fix:** Elaborate on the deployment lifecycle. Explain that M3TAL is a Docker orchestrator, that the `/docker` directory is the user-facing entry point to the stack configuration, and that `m3tal up` aggregates and runs all `*-compose.yml` files within this directory. Mention the existence and purpose of the `/opt/m3tal/stack/` directory and the symlink.

4.  **BLOCKER: Traefik Routing Explanation**
    *   **Description:** The README mentions Traefik in the "Routing Stack" but does not explain *how* services are exposed through it. It's unclear if this is via labels, dynamic configuration files, or another mechanism.
    *   **Required Fix:** Clearly explain that Traefik acts as the HTTP gateway and how services are exposed. Detail that this is typically achieved using Docker labels (e.g., `traefik.enable=true`, `traefik.http.routers.service.rule=...`) or dynamic configuration files within Traefik's configuration. Reference the `traefik.yml` and `dynamic` directory from the GROUND TRUTH.

5.  **WARNING: Missing Port Table**
    *   **Description:** While the "Infrastructure Stacks" section lists ports for individual services, a consolidated port table that explicitly lists ports 80, 8080, 8081, and 8082 for general access and management is missing.
    *   **Required Fix:** Add a dedicated "Ports" section or table that lists these common ports and their purpose (e.g., 80 for Traefik public, 8080 for Traefik dashboard (if exposed), 8081 for internal API, 8082 for M3TAL Dashboard). The GROUND TRUTH provides a good example of this.

6.  **WARNING: Service Management Clarity**
    *   **Description:** The README does not mention `systemctl` for managing the `m3tal-api.service`. While the API runs in Docker, the systemd service is the primary way to manage the daemon process on the host.
    *   **Required Fix:** Add a note mentioning that `m3tal-api.service` is managed via `systemctl` and provide basic commands like `systemctl status m3tal-api.service`, `systemctl restart m3tal-api.service`, and `journalctl -u m3tal-api.service -f`.

7.  **WARNING: Firewall Reminder Missing**
    *   **Description:** The README does not include a reminder for users to ensure port 80 is open in their firewall (e.g., `ufw`, `iptables`) for external access.
    *   **Required Fix:** Add a brief note advising users to allow incoming traffic on port 80 through their host firewall.

8.  **WARNING: Marketing Copy Tone**
    *   **Description:** The introductory sentence "Autonomous, self-healing media automation platform" leans towards marketing copy rather than purely technical documentation. While a brief intro is fine, the overall tone should remain technical.
    *   **Required Fix:** Adjust the introductory sentence to be more descriptive of the system's function from a technical standpoint, if possible, or ensure the rest of the document remains strictly technical.

9.  **SUGGESTION: Quick Start Section**
    *   **Description:** The README lacks a "Quick Start" or "Getting Started" section that would allow a new user to perform a basic installation and get a service running quickly. The current deployment section is too high-level and relies on an external script.
    *   **Required Fix:** Add a "Quick Start" section that walks users through the essential steps: installing M3TAL (using the correct APT block), performing initial configuration (e.g., `m3tal config wizard`), and deploying a basic stack (e.g., `m3tal up`).

---

**Additional Notes:**

*   The "Architecture (Source Services)" section seems to list internal Go module paths rather than user-relevant architectural components. This is not a direct audit failure against the criteria but could be confusing.
*   The "Environment Configuration" table is a good start but should ideally be linked to the `.env` file where these variables are actually set and managed.
*   The "Dashboard Access" modes described in the GROUND TRUTH are not reflected in the README at all. This is a significant omission as it dictates how users will access the dashboard. The README currently only implies a single mode.
*   The README's "Deployment" section mentioning `python m3tal.py init` is incorrect based on the GROUND TRUTH, which states M3TAL is installed via APT and uses `m3tal up` for deployment. The `m3tal.py` script and `init` command are not part of the documented installation process.