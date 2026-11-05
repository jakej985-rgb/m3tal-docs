## M3TAL README Audit Report

**Verdict:** FAILED (Multiple BLOCKER and WARNING issues)

### Issue List:

1.  **BLOCKER: APT Installation Missing**
    *   **Description:** The README does not include the required 3-command APT installation block (keyring, repository, install). This is the primary method for installing M3TAL according to the ground truth.
    *   **Required Fix:** Add the following block to the "Installation" or a dedicated "Getting Started" section:
        ```bash
        curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
        echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list
        sudo apt update && sudo apt install -y m3tal
        ```

2.  **BLOCKER: Docker Dependency Not Stated**
    *   **Description:** The README fails to explicitly state that Docker Engine and Docker Compose V2 are required for M3TAL to function. The ground truth confirms M3TAL is a Docker orchestrator.
    *   **Required Fix:** Add a clear statement in a "Prerequisites" or "Installation" section like: "M3TAL requires Docker Engine and Docker Compose V2 to be installed on your system."

3.  **BLOCKER: Deployment Lifecycle Missing**
    *   **Description:** The README lacks explanation on how "stacks" work, the role of the `/docker` directory (and its symlink to `/opt/m3tal/stack/`), and how `m3tal up` uses `*-compose.yml` files. This is crucial for understanding deployment.
    *   **Required Fix:** Add a section explaining:
        *   That M3TAL is a Docker orchestrator.
        *   The `/docker` directory is a symlink to `/opt/m3tal/stack/`.
        *   `m3tal up` executes `docker compose` across all `*-compose.yml` files within the stack directory.
        *   Mention how new compose files can be added.

4.  **BLOCKER: Traefik Routing Explanation Missing**
    *   **Description:** The README does not explain that Traefik serves as the HTTP gateway or how services are exposed (via labels or dynamic configuration), which is a core component of the M3TAL architecture.
    *   **Required Fix:** Add a section detailing Traefik's role as the reverse proxy and explain that services are exposed using Traefik labels within their compose files or via dynamic configuration files. Referencing the example `traefik.yml` and `dynamic/api.yml` from the ground truth would be beneficial.

5.  **WARNING: Tone - Marketing Copy**
    *   **Description:** The introductory sentence "Autonomous, self-healing media automation platform" leans towards marketing copy rather than technical documentation. While a brief overview is fine, the primary focus should be on technical details.
    *   **Required Fix:** Rephrase the introduction to be more direct and informative about the system's purpose from a technical standpoint. For example: "M3TAL is a Docker-based platform for automating media workflows."

6.  **WARNING: Port Table Incomplete**
    *   **Description:** The README lists Traefik ports and `DASHBOARD_PORT` (8082) but omits `API_PORT` (5050) and `TRAEFIK_DASHBOARD_PORT` (8081) which are present in the ground truth and relevant for users.
    *   **Required Fix:** Update or add a port table to include:
        *   80: Traefik HTTP
        *   443: Traefik HTTPS (if applicable)
        *   5050: M3TAL Go API daemon (host-local)
        *   8080: Traefik Dashboard (host-local only, mapped via `TRAEFIK_DASHBOARD_PORT`)
        *   8081: Traefik Dashboard (host-local, as per ground truth mapping of 127.0.0.1:8081:8080)
        *   8082: M3TAL Dashboard container

7.  **WARNING: Service Management Not Mentioned**
    *   **Description:** The README does not mention how to manage the `m3tal-api.service` (e.g., using `systemctl`), which is crucial for operating the backend.
    *   **Required Fix:** Add a note in a "Service Management" or "Operation" section stating: "The M3TAL API service is managed via systemd. Use commands like `sudo systemctl status m3tal-api`, `sudo systemctl restart m3tal-api`, and `sudo journalctl -u m3tal-api -f` for monitoring."

8.  **WARNING: Firewall Note Missing**
    *   **Description:** The README omits a reminder for users to configure their firewall (e.g., `ufw` or `iptables`) to allow inbound traffic on port 80 (Traefik's public HTTP port).
    *   **Required Fix:** Add a "Firewall Configuration" note, e.g., "Ensure your firewall (e.g., ufw, iptables) allows inbound traffic on port 80 for public access to M3TAL services via Traefik."

9.  **SUGGESTION: Quick Start Missing**
    *   **Description:** The README lacks a "Quick Start" or "Quick Demo" section that guides a user through a minimal, working setup. The current deployment command (`python m3tal.py init`) is too vague and doesn't constitute a full quick start.
    *   **Required Fix:** Add a "Quick Start" section that outlines the essential steps for a basic installation and verification, including:
        *   Installing M3TAL via APT.
        *   Running initial configuration (if applicable, or directing to `.env` setup).
        *   A command like `m3tal up` to start the core services.
        *   How to access the dashboard or a key service.
        *   Crucially, it should demonstrate the two dashboard exposure modes (`local` vs. `traefik`) as this is a key operational aspect.