## M3TAL README Audit

**Verdict: FAILED**

The README contains significant gaps in critical deployment and operational information, making it insufficient for a user to successfully install and operate the M3TAL system without referring to external knowledge or the ground truth.

---

### Issue List:

1.  **BLOCKER: APT installation block is incomplete.**
    *   **Classification**: BLOCKER
    *   **Reason**: The README states "M3TAL is installed via APT — not built from source. The correct install is:", then provides the 3-command block correctly. This section is NOT missing, however, the "Core Components" section lists "routing-compose.yml" which implies this file needs to be present *before* `apt install m3tal`. The APT install should be the *only* method of obtaining the core `m3tal` binary and its necessary associated files (like default compose files). The APT installation *should* ensure `routing-compose.yml` and other necessary compose files are installed as part of the package. The README does not clarify this dependency or how `routing-compose.yml` is provided by the APT package itself.
    *   **Required Fix**: Clarify that the APT installation process installs all necessary core compose files, including `routing-compose.yml`, as part of the `m3tal` package.

2.  **BLOCKER: Docker Compose V2 dependency is implied but not explicitly stated as required.**
    *   **Classification**: BLOCKER
    *   **Reason**: The "Prerequisites" section states "Docker Engine and Docker Compose V2 are strictly REQUIRED". However, the "Deployment Lifecycle" section only mentions "Docker Compose V2" and `m3tal up` being a wrapper around `docker compose`. It does not explicitly state that Docker Compose V2 *must* be installed and functional *before* running any M3TAL commands.
    *   **Required Fix**: Ensure the "Prerequisites" clearly states that Docker Engine and Docker Compose V2 must be *installed and operational* before proceeding with M3TAL installation or any `m3tal` commands.

3.  **BLOCKER: Deployment lifecycle explanation lacks crucial details.**
    *   **Classification**: BLOCKER
    *   **Reason**: The README states that `m3tal up` runs `docker compose` across `*-compose.yml` files in `/docker/`. However, it fails to mention:
        *   That `/docker` is a symlink to `/opt/m3tal/stack/`.
        *   The existence and role of override files (`m3tal-compose.local.yml` and `m3tal-compose.traefik.yml`) which are critical for dashboard exposure modes.
        *   How Traefik itself is deployed and managed (it's mentioned as a component but not how `m3tal up` orchestrates it).
        *   The `m3tal dash up` command for specifically managing the dashboard.
    *   **Required Fix**:
        *   Explicitly state that `/docker` is a symlink to `/opt/m3tal/stack/`.
        *   Explain the role of override files in the dashboard access section and link them to `DASHBOARD_EXPOSE_MODE`.
        *   Clarify that `m3tal up` deploys Traefik and other core services based on the files in `/docker/`.
        *   Add the `m3tal dash up` command to the deployment lifecycle and quick demo sections.

4.  **BLOCKER: Traefik routing mechanism is misrepresented and incomplete.**
    *   **Classification**: BLOCKER
    *   **Reason**: The README states Traefik uses "a file provider for dynamic routing configuration" and that services are exposed via "Traefik labels". It then incorrectly states "The API daemon, running on the host at `http://localhost:8080`, is exposed via Traefik using dynamic configuration. For example, `api.DOMAIN` is routed to the Go API daemon on host-local port `8080` via `http://host.docker.internal:8080` through a dynamic configuration file (e.g., `dynamic/api.yml`)."
        *   The ground truth indicates Traefik configuration is *not* dynamic via files in `/docker/dynamic/` in the way described.
        *   The ground truth states Traefik routing is configured *via labels* on the services themselves, not by external dynamic config files for API routing. The API daemon is exposed via a specific override file.
        *   The mention of `routing-compose.yml` as a component is confusing; the ground truth does not show a `routing-compose.yml` file. Traefik is typically part of a larger compose setup.
    *   **Required Fix**:
        *   Remove the reference to `routing-compose.yml` as a separate component.
        *   Correctly explain that Traefik routes services based on **Docker labels** applied to the service definitions in the `*-compose.yml` files within `/docker/`.
        *   For the dashboard, explain that `DASHBOARD_EXPOSE_MODE=traefik` enables Traefik routing via labels on the dashboard service definition, and that this is controlled by `m3tal-compose.traefik.yml`.
        *   Clarify how the API daemon is exposed *if* it's via labels or a specific compose override, rather than dynamic config files not present in the ground truth. The ground truth implies the API is exposed via `API_PORT=5050` and potentially Traefik labels applied via an override.

5.  **WARNING: Port table is missing expected ports and is incomplete.**
    *   **Classification**: WARNING
    *   **Reason**: The README's "Port Map" section is missing the host-local port `5050` for the API daemon (which is specified as `API_PORT=5050` in the ground truth and potentially exposed via Traefik). It also doesn't clearly distinguish between host ports and container internal ports for all entries.
    *   **Required Fix**: Update the port table to include port `5050` with its description and access method. Ensure clarity between host and container ports.

6.  **WARNING: Service management documentation is incomplete.**
    *   **Classification**: WARNING
    *   **Reason**: The README mentions `m3tal-api.service` is managed by `systemctl` and lists basic commands. However, it omits the crucial command for checking the service status and the command for viewing logs.
    *   **Required Fix**: Add `systemctl status m3tal-api` and `journalctl -u m3tal-api -f` to the service management section.

7.  **WARNING: Firewall note is not comprehensive.**
    *   **Classification**: WARNING
    *   **Reason**: The firewall note only reminds users to allow port `80` for Traefik. It doesn't mention other potentially required ports like `8082` (for local dashboard access) or `443` (if HTTPS is implied or configured).
    *   **Required Fix**: Expand the firewall note to include port `8082` for local access and mention that `443` might be necessary if HTTPS is configured.

8.  **WARNING: Tone drifts into marketing copy.**
    *   **Classification**: WARNING
    *   **Reason**: Phrases like "unified Go binary serving as the primary command-line interface for all M3TAL operations" and "orchestrates Docker containers using Docker Engine and Docker Compose V2" are slightly more descriptive than purely technical documentation. While not overtly marketing, it could be more direct and concise.
    *   **Required Fix**: Review the language to ensure it remains strictly technical and instructional. For example, instead of "unified Go binary serving as the primary command-line interface," use "The `m3tal` binary is the CLI tool."

9.  **SUGGESTION: Quick Start section is missing a critical step.**
    *   **Classification**: SUGGESTION
    *   **Reason**: The "Quick Demo" section is a good start, but it incorrectly states that `m3tal dash up` downloads necessary compose files. This command *orchestrates* the dashboard using existing compose files. It also doesn't explicitly mention running `m3tal config wizard` or setting `DASHBOARD_EXPOSE_MODE` as prerequisites for the demo.
    *   **Required Fix**:
        *   Clarify that `m3tal dash up` uses existing compose files.
        *   Suggest running `m3tal config wizard` or ensuring `DASHBOARD_EXPOSE_MODE` is set before running `m3tal dash up` for the demo.
        *   Add a note about accessing the dashboard (either `http://HOST_IP:8082` or `http://dash.DOMAIN`) based on the configured mode.

---