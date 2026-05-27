## M3TAL README Audit

**Verdict:** FAILED - Multiple BLOCKER issues identified.

The README is missing critical information required for a user to successfully install and operate the M3TAL system. Specifically, the APT installation commands are incomplete, the deployment lifecycle for stacks is not fully explained, and the mechanism for Traefik routing is unclear.

---

### Issue List:

1.  **BLOCKER: APT installation commands are incomplete.**
    *   **Description:** The README provides the `curl` and `echo` commands for adding the repository and keyring, but it omits the final `sudo apt update && sudo apt install -y m3tal` command. Without this, a user cannot actually install the M3TAL package.
    *   **Required Fix:** Add the missing `sudo apt update && sudo apt install -y m3tal` command to the APT installation section.

2.  **BLOCKER: Docker dependency is not explicitly stated.**
    *   **Description:** While the README mentions "Docker Engine and Docker Compose V2 are strictly REQUIRED," it does not provide instructions or a clear statement on how to install them. This is a fundamental prerequisite that must be explicitly addressed.
    *   **Required Fix:** Add a clear section or integrate instructions on how to install Docker Engine and Docker Compose V2 on common Linux distributions (e.g., Ubuntu/Debian) before the M3TAL installation steps.

3.  **BLOCKER: Deployment lifecycle for Docker stacks is not fully explained.**
    *   **Description:** The README states that `m3tal up` runs `docker compose` across `*-compose.yml` files in `/docker/`. However, it does not clearly explain how the `/docker` directory relates to the actual stack files. It mentions `/opt/m3tal/stack/` as the canonical source and `/docker` as a symlink. This relationship, and how to add new compose files to `/docker/` (and have them picked up by `m3tal up`), needs explicit clarification. The ground truth indicates `*-compose.yml` files in `/docker/` are processed. The README mentions placing files in `/docker/` which is correct, but the explanation of how `m3tal up` works with these is underdeveloped.
    *   **Required Fix:** Clearly explain that the `/docker` directory is a symlink to `/opt/m3tal/stack/` and that any `*-compose.yml` files placed in `/docker/` will be picked up by `m3tal up`. The current description of "all `*-compose.yml` files located within the `/docker/` directory" is a good start, but the relationship with `/opt/m3tal/stack/` and how new files are discovered needs to be more explicit. The "Adding a New Stack" section does this well.

4.  **BLOCKER: Traefik routing mechanism is not sufficiently explained.**
    *   **Description:** The README states Traefik is the HTTP gateway and "automatically discovers and routes traffic to Docker services by interpreting Traefik labels." However, it lacks detail on *how* this discovery happens beyond a general statement. It doesn't explicitly state that services need `traefik.enable=true` and other labels to be routed, nor does it clearly explain the role of the `proxy` network in Traefik's discovery. The example of exposing a custom service is good but relies on an external `networks: proxy:` definition without fully explaining *why* this network is crucial for Traefik.
    *   **Required Fix:** Clearly state that services must have `traefik.enable=true` and associated labels in their compose files to be discoverable by Traefik. Emphasize the importance of services being on the `proxy` network for Traefik's Docker provider to find them. Clarify that Traefik itself is managed via `m3tal up` within `routing-compose.yml`.

5.  **WARNING: Missing explanation of dashboard access modes and their implications.**
    *   **Description:** While the README has a "Dashboard Access" section, it assumes the user understands how `m3tal-compose.local.yml` and `m3tal-compose.traefik.yml` are applied or how `DASHBOARD_EXPOSE_MODE` actually changes the deployment. It doesn't explicitly state that these are override files that modify the base `m3tal-compose.yml`. The connection between the `.env` variable and the resulting Docker Compose configuration needs to be clearer.
    *   **Required Fix:** Clarify that `m3tal-compose.local.yml` and `m3tal-compose.traefik.yml` are override files that are selectively applied by M3TAL based on the `DASHBOARD_EXPOSE_MODE` setting in `/etc/m3tal/.env`.

6.  **WARNING: Tone is marketing-like in places.**
    *   **Description:** Phrases like "strictly REQUIRED" in the prerequisites and the general tone of the "Components" section lean towards marketing rather than purely technical documentation.
    *   **Required Fix:** Adjust the language to be more neutral and objective. For example, "Docker Engine and Docker Compose V2 are required" is sufficient.

7.  **SUGGESTION: Quick demo could be more comprehensive.**
    *   **Description:** The "Quick Demo" section is very brief. While it shows `m3tal dash up` and `m3tal up`, it could benefit from a step-by-step guide that includes verifying installation, initial configuration (e.g., `m3tal config wizard`), and accessing the dashboard for the first time.
    *   **Required Fix:** Expand the "Quick Demo" section to include a minimal end-to-end flow, such as:
        1.  Install M3TAL.
        2.  Run `m3tal config wizard` (or mention its necessity).
        3.  Run `m3tal up`.
        4.  Explain how to access the dashboard based on the default `DASHBOARD_EXPOSE_MODE`.

---

### Summary of Required Fixes:

1.  **APT Installation:** Add `sudo apt update && sudo apt install -y m3tal` to the installation steps.
2.  **Docker Dependency:** Provide clear instructions for installing Docker Engine and Docker Compose V2.
3.  **Deployment Lifecycle:** Explicitly state that `/docker` is a symlink to `/opt/m3tal/stack/` and that `m3tal up` processes all `*-compose.yml` in `/docker/`.
4.  **Traefik Routing:** Detail that services need `traefik.enable=true` and labels, and that they must be on the `proxy` network for Traefik discovery.
5.  **Dashboard Access Modes:** Explain that `m3tal-compose.local.yml` and `m3tal-compose.traefik.yml` are override files applied based on `DASHBOARD_EXPOSE_MODE`.
6.  **Tone:** Refine marketing-like language to be more technical and objective.
7.  **Quick Demo:** Expand the Quick Demo to include a more complete introductory workflow.