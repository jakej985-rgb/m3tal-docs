## M3TAL DevOps Audit Report

**Auditor:** DocCritic, Senior DevOps Auditor, M3TAL Platform
**Date:** 2023-10-27
**Platform:** M3TAL
**Document Audited:** README.md

---

**Verdict: FAILED** - The README is missing critical information required for successful installation and operation. It relies heavily on implied knowledge and internal tooling that is not adequately explained for a new user.

---

### Issue List:

1.  **BLOCKER: APT Installation Commands Missing**
    *   **Description:** The README does not provide the essential three-command sequence (keyring, repository, install) required for installing M3TAL via APT. This is a fundamental blocker for users intending to install the platform as documented by its installation method.
    *   **Classification:** BLOCKER
    *   **Required Fixes:** Add the correct APT installation block as provided in the GROUND TRUTH:
        ```bash
        curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
        echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list
        sudo apt update && sudo apt install -y m3tal
        ```

2.  **BLOCKER: Docker Dependency Not Stated**
    *   **Description:** The README fails to explicitly state that Docker Engine and Docker Compose V2 are required dependencies for M3TAL to function. While it mentions "Infrastructure Stacks" and hints at Docker, the explicit dependency is missing.
    *   **Classification:** BLOCKER
    *   **Required Fixes:** Add a clear statement early in the README (e.g., in an "Prerequisites" or "Installation" section) that Docker Engine and Docker Compose V2 must be installed and running.

3.  **BLOCKER: Deployment Lifecycle Explanation Missing**
    *   **Description:** The README does not explain how M3TAL manages its Docker stack. Key concepts like the `/docker` directory (and its symlink nature to `/opt/m3tal/stack/`), how `m3tal up` operates across multiple compose files, and how new compose files are added are absent.
    *   **Classification:** BLOCKER
    *   **Required Fixes:**
        *   Explain that M3TAL orchestrates Docker Compose stacks.
        *   Clarify that the user-facing `/docker` directory is a symlink to `/opt/m3tal/stack/`.
        *   Describe that `m3tal up` executes `docker compose` commands against all `*-compose.yml` files found within the stack directory (e.g., `/docker/` or `/opt/m3tal/stack/`).
        *   Explain how adding new compose files to this directory integrates them into the `m3tal up` process.

4.  **BLOCKER: Traefik Routing Explanation Missing**
    *   **Description:** The README does not clearly explain that Traefik serves as the HTTP gateway for M3TAL services. It also lacks information on how services are exposed to Traefik, specifically whether it's via Docker labels or dynamic configuration files.
    *   **Classification:** BLOCKER
    *   **Required Fixes:**
        *   State explicitly that Traefik acts as the primary HTTP gateway.
        *   Explain that services are exposed to Traefik primarily through Docker labels defined in their respective compose files, with Traefik dynamically discovering these services. Mentioning the existence of dynamic configuration files (`dynamic/api.yml`, etc.) for advanced routing would also be beneficial.

5.  **WARNING: Missing Port Table**
    *   **Description:** The README lacks a dedicated table listing the key ports used by M3TAL and its components (80, 8080, 8081, 8082). While some ports are mentioned in the "Infrastructure Stacks" section, a consolidated port map is crucial for network configuration and troubleshooting.
    *   **Classification:** WARNING
    *   **Required Fixes:** Add a "Port Mapping" or "Network Ports" section that clearly lists the following:
        *   Port 80: Traefik HTTP (public access)
        *   Port 8080: M3TAL API (internal/Traefik routing)
        *   Port 8081: Traefik Dashboard (local access only)
        *   Port 8082: M3TAL Dashboard (container port, exposed via host IP or Traefik)

6.  **WARNING: Service Management Not Mentioned**
    *   **Description:** The README does not mention `systemctl` for managing the `m3tal-api.service`. This is a vital piece of information for users who need to check the status, restart, or view logs of the core API daemon.
    *   **Classification:** WARNING
    *   **Required Fixes:** Add a section on "Service Management" that includes commands like:
        *   `sudo systemctl status m3tal-api.service`
        *   `sudo systemctl restart m3tal-api.service`
        *   `sudo journalctl -u m3tal-api.service -f`

7.  **WARNING: Firewall Note Missing**
    *   **Description:** The README does not remind users to configure their firewall (e.g., `ufw` or `iptables`) to allow inbound traffic on port 80, which is essential for external access to services proxied by Traefik.
    *   **Classification:** WARNING
    *   **Required Fixes:** Include a note in the installation or deployment section advising users to ensure port 80 is open in their firewall for public access.

8.  **WARNING: Tone is Marketing Copy**
    *   **Description:** The introduction "Autonomous, self-healing media automation platform" and the overall presentation leans towards marketing language rather than direct technical documentation. While some descriptive language is acceptable, the core of a README should be technical and procedural.
    *   **Classification:** WARNING
    *   **Required Fixes:** Rephrase introductory sentences to be more factual and less promotional. Focus on what M3TAL *is* and *does* from a technical standpoint.

9.  **SUGGESTION: Quick Start Section Missing**
    *   **Description:** The README lacks a "Quick Start" or "Getting Started" section that guides a new user through the absolute minimum steps to get M3TAL up and running. This would greatly improve the initial user experience.
    *   **Classification:** SUGGESTION
    *   **Required Fixes:** Add a "Quick Start" section that covers:
        *   Installing M3TAL (using the corrected APT commands).
        *   Running an initial configuration command (if applicable, e.g., `m3tal config wizard`).
        *   Executing `m3tal up` to start the core services.
        *   How to access the dashboard initially.

---

**Overall Recommendation:** The README needs significant technical detail added to make it a functional guide for installing and operating M3TAL. The current version is insufficient and would likely lead to confusion and failed deployments for new users. The "Deployment" section is particularly weak, only mentioning `python m3tal.py init` without context or further steps.