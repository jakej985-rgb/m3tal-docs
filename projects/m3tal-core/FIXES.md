## M3TAL System Documentation Audit Report

**Auditor:** DocCritic, Senior DevOps Auditor
**Platform:** M3TAL
**Date:** 2023-10-27

### Verdict: FAILED

The README.md contains significant gaps and inaccuracies when compared against the provided Ground Truth, particularly concerning core operational concepts like deployment lifecycle and Traefik routing. These omissions constitute BLOCKER issues that would prevent a user from successfully deploying and operating the M3TAL system.

---

### Issues:

1.  **BLOCKER: APT installation missing keyring+repo+install block**
    *   **Description:** The README claims to provide installation steps but does not contain the complete 3-command block for APT installation (keyring, repository, and install). The provided commands are present, but the issue is that the README *states* it shows the block, implying it's a complete instruction. The ground truth confirms the exact 3 commands are expected in the README for a complete installation guide.
    *   **Required Fix:** Ensure the README explicitly presents the full 3-command sequence as per the ground truth for APT installation. The current README *does* show the commands, so the flagging is based on the *expectation* that this is a complete, self-contained installation instruction. However, to be absolutely clear and avoid any ambiguity, explicitly state "the following three commands are required to install M3TAL via APT".

2.  **BLOCKER: Docker dependency not stated**
    *   **Description:** The README states Docker Engine and Docker Compose V2 are required. However, it does not explicitly state that M3TAL *uses* Docker Engine + Docker Compose V2 internally for orchestration. This is a crucial detail for understanding how M3TAL operates.
    *   **Required Fix:** Modify the "Prerequisites" section to explicitly state that M3TAL internally orchestrates containers using Docker Engine and Docker Compose V2.

3.  **BLOCKER: Deployment lifecycle explanation missing**
    *   **Description:** The README's explanation of the deployment lifecycle is fundamentally flawed and incomplete. It states `m3tal up` runs `docker compose` on all `*-compose.yml` files in `/docker/`, which is partially correct. However, it fails to mention:
        *   The role of the `/docker` directory being a symlink to `/opt/m3tal/stack/`.
        *   That `/opt/m3tal/stack/` is the canonical source of truth.
        *   How Traefik routing is configured via labels or dynamic config files within this structure.
    *   **Required Fix:** Revise the "Deployment Lifecycle" section to accurately reflect that `/docker` is a symlink to `/opt/m3tal/stack/`. Explain that `m3tal up` orchestrates Docker Compose across files in `/opt/m3tal/stack/`. Detail how services are exposed via Traefik labels or dynamic configuration files located within this stack directory.

4.  **BLOCKER: Traefik routing explanation missing critical details**
    *   **Description:** The explanation of Traefik routing is insufficient. While it mentions Traefik as a reverse proxy and the use of labels, it omits:
        *   That Traefik is configured via static (`traefik.yml`) and dynamic (`dynamic/` directory) configuration files.
        *   How Traefik discovers services (explicitly mentioning `traefik.enable=true` and other labels).
        *   The mechanism for routing to host-local services (e.g., API daemon) via dynamic configuration files.
        *   The specific ports Traefik listens on publicly (80/443) and internally (8081 for dashboard).
    *   **Required Fix:** Expand the "Traefik Gateway" section to detail the static and dynamic configuration of Traefik. Clearly explain that services are exposed via Traefik labels (requiring `traefik.enable=true`) and how dynamic configuration files in `/docker/dynamic/` are used for advanced routing, including routing to host-local services. Mention the public ports Traefik listens on.

5.  **WARNING: Port table is incomplete**
    *   **Description:** The "Port Map" table in the README lists ports 80, 8080, 8081, and 8082. However, it does not explicitly mention port 443, which is the standard HTTPS port and is often utilized by Traefik. The ground truth indicates Traefik is configured to listen on 443 if HTTPS is configured.
    *   **Required Fix:** Add port 443 to the "Port Map" table with a description indicating its use for Traefik HTTPS (if configured).

6.  **WARNING: Service management section could be more specific**
    *   **Description:** The README mentions `systemctl` for managing `m3tal-api.service`. While this is correct, it doesn't explicitly state that this is the *primary* method for managing the API daemon. The ground truth confirms `m3tal-api.service` is managed by systemd.
    *   **Required Fix:** Explicitly state that `m3tal-api.service` is managed by systemd and that `systemctl` commands are the standard way to control its lifecycle.

7.  **WARNING: Firewall note is lacking detail**
    *   **Description:** The "Firewall Considerations" section correctly advises to allow port 80. However, it could be more comprehensive by mentioning that port 443 should also be allowed if HTTPS is enabled, as Traefik also listens on this port.
    *   **Required Fix:** Update the "Firewall Considerations" section to include a reminder to allow port 443 if HTTPS is configured.

8.  **WARNING: Tone - Marketing copy present**
    *   **Description:** Phrases like "unified Go binary serving as the single entrypoint for all M3TAL operations" and the general flow of the "Overview" and "Components" sections lean towards marketing language rather than purely technical documentation.
    *   **Required Fix:** Refine the language throughout the README to be more direct, concise, and technically focused, removing any marketing-oriented phrasing. For example, instead of "unified Go binary serving as the single entrypoint," simply state "The CLI binary (`/usr/bin/m3tal`) is the primary command-line interface."

9.  **SUGGESTION: Quick demo section is functional but could be improved**
    *   **Description:** The "Quick Demo" section provides the `m3tal dash up` and `m3tal up` commands. While functional, it could be enhanced by:
        *   Providing example output or confirmation messages.
        *   Briefly explaining what each command *does* in terms of user-facing outcomes (e.g., "This starts the M3TAL Dashboard and makes it accessible at `http://localhost:8082` if `DASHBOARD_EXPOSE_MODE` is `local`.").
        *   Adding a very simple example of how to interact with the system after setup (e.g., "Access the dashboard at your configured IP and port.").
    *   **Required Fix:** Enhance the "Quick Demo" section with brief explanations of what each command achieves and what the user should expect as an outcome or next step.

---