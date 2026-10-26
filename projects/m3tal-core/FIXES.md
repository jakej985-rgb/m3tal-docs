## Audit Verdict

**FAILED**

The README is missing critical information regarding the Docker dependency and the Traefik routing mechanism, making it a BLOCKER for users to successfully install and operate the system. Additionally, there are several areas for improvement in terms of clarity and completeness.

---

## Audit Findings

1.  **BLOCKER: APT installation**
    *   **Issue:** The README correctly provides the 3-command keyring+repo+install block for APT installation.
    *   **Classification:** PASSED

2.  **BLOCKER: Docker dependency**
    *   **Issue:** The README states that Docker Engine and Docker Compose V2 are REQUIRED, but it fails to explicitly mention that Docker Compose V2 is the version required. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.
    *   **Classification:** BLOCKER
    *   **Required Fix:** Explicitly state that Docker Engine and **Docker Compose V2** are required.

3.  **BLOCKER: Deployment lifecycle**
    *   **Issue:** The README explains that `m3tal up` runs `docker compose` across all `*-compose.yml` files in `/docker/`. It also mentions `/docker` is a symlink to `/opt/m3tal/stack/`. However, it does not fully explain how stacks work in terms of the `/docker` directory being the user-facing location for adding new compose files that `m3tal up` will pick up. The current explanation is a bit brief and could be clearer about the mechanism.
    *   **Classification:** BLOCKER
    *   **Required Fix:** Clarify that placing new `*-compose.yml` files directly into the `/docker/` directory will cause `m3tal up` to include them in the deployment process. Explicitly link the `/docker` directory as the user-facing location for managing stacks that `m3tal up` discovers.

4.  **BLOCKER: Traefik routing**
    *   **Issue:** The README provides a good overview of Traefik's role and how it uses labels. However, it misses a crucial detail: how services get exposed via dynamic configuration files (like `dynamic/api.yml`) in addition to labels. The current explanation focuses heavily on labels and doesn't fully convey the capability and necessity of dynamic configuration for certain routes.
    *   **Classification:** BLOCKER
    *   **Required Fix:** Expand the explanation of Traefik routing to include how services can be exposed via dynamic configuration files (e.g., in `/docker/dynamic/`) in addition to Docker labels. Provide a brief example or mention that this is used for specific internal routing scenarios like `api.DOMAIN`.

5.  **WARNING: Port table**
    *   **Issue:** The README lists ports 80, 8080, 8081, and 8082, which aligns with the ground truth.
    *   **Classification:** PASSED

6.  **WARNING: Service management**
    *   **Issue:** The README correctly mentions `systemctl` for managing `m3tal-api.service`.
    *   **Classification:** PASSED

7.  **WARNING: Firewall note**
    *   **Issue:** The README provides a reminder to allow port 80 in `ufw/iptables` when using Traefik.
    *   **Classification:** PASSED

8.  **WARNING: Tone**
    *   **Issue:** The tone is appropriate for technical documentation.
    *   **Classification:** PASSED

9.  **SUGGESTION: Quick demo**
    *   **Issue:** The README includes a functional Quick Start section that guides users through starting the dashboard and then deploying all stacks.
    *   **Classification:** PASSED

---