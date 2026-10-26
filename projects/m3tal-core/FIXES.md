## M3TAL README Audit Report

**Verdict: FAILED**

The README is missing critical information regarding APT installation and the Docker dependency, which are BLOCKER issues. Additionally, several WARNINGs and SUGGESTIONS are present that would improve the overall clarity and usability of the documentation.

---

### Issue List:

1.  **BLOCKER: APT Installation Command Block**
    *   **Description:** The README does not present the complete 3-command block required for APT installation (keyring, repo, install). While individual steps are present, they are not grouped as a single, executable block.
    *   **Impact:** Users cannot perform a straightforward APT installation as the documentation implies.
    *   **Required Fix:** Combine the three individual APT commands into a single, executable code block as per the GROUND TRUTH.

2.  **BLOCKER: Docker Dependency Statement**
    *   **Description:** The README states Docker Engine and Docker Compose V2 are required, but it does not explicitly mention *where* to install them from or that they are prerequisites *before* M3TAL installation.
    *   **Impact:** Users might attempt to install M3TAL before having Docker properly set up, leading to installation failures.
    *   **Required Fix:** Add a sentence or two clarifying that Docker Engine and Docker Compose V2 need to be installed and functioning *before* proceeding with M3TAL installation.

3.  **WARNING: Deployment Lifecycle - `m3tal up` Behavior**
    *   **Description:** The README states `m3tal up` operates on all `*-compose.yml` files in `/docker/`. While it mentions placing new files there, it doesn't clearly explain that `m3tal up` will deploy *all* of them, including the core M3TAL services, not just user-added ones. It also doesn't explicitly mention that `/docker` is a symlink to `/opt/m3tal/stack/`.
    *   **Impact:** Users might misunderstand how `m3tal up` aggregates compose files or where the canonical stack directory resides.
    *   **Required Fix:** Clarify that `m3tal up` processes all compose files in `/docker/` which is a symlink to `/opt/m3tal/stack/`. Explicitly state that this includes M3TAL's core services.

4.  **WARNING: Traefik Routing Configuration Detail**
    *   **Description:** While the README explains Traefik's role and mentions labels for service exposure, it doesn't explicitly detail the *two* primary mechanisms for service exposure as seen in the GROUND TRUTH: Traefik labels directly on service definitions and Traefik's file provider for dynamic configuration (e.g., routing to host-local services). The example for custom services is good, but the explanation of how Traefik itself is configured (static config like `traefik.yml` and dynamic config) is missing.
    *   **Impact:** Users might not fully grasp the flexibility and different ways Traefik routes traffic within M3TAL.
    *   **Required Fix:** Expand the Traefik section to mention both label-based routing and the file provider for dynamic configuration. Briefly explain how `traefik.yml` and the dynamic configuration directory work in conjunction.

5.  **WARNING: Port Table Missing Specificity for Access**
    *   **Description:** The port table lists ports 80, 8080, 8081, and 8082. However, for ports 8080, 8081, and 8082, it doesn't clearly specify their access method (host-local vs. public vs. via Traefik/direct). The description for 8082 is better but could be more precise.
    *   **Impact:** Users might try to access internal ports externally or vice-versa.
    *   **Required Fix:** Enhance the "Access" column in the Port Map table to explicitly state "Host-local", "Public", or "Direct port (local mode) or via Traefik (traefik mode)".

6.  **SUGGESTION: Firewall Note Granularity**
    *   **Description:** The firewall note mentions allowing port 80 in `ufw/iptables` if Traefik is used for public-facing access. It would be beneficial to also remind users to open port 443 if HTTPS is configured, as this is a common setup.
    *   **Impact:** Users might overlook opening port 443 if they are setting up HTTPS.
    *   **Required Fix:** Add a mention of port 443 to the firewall considerations section.

7.  **SUGGESTION: Marketing Copy Tone**
    *   **Description:** The "Overview" section uses slightly marketing-oriented language ("This document provides technical details and operational procedures..."). While not egregious, the tone could be more purely technical and direct.
    *   **Impact:** Minor deviation from a strictly technical documentation tone.
    *   **Required Fix:** Rephrase the "Overview" to be more direct and technical, e.g., "This document details the technical architecture and operational procedures for the M3TAL system."

8.  **SUGGESTION: Quick Demo Clarity on `m3tal dash up`**
    *   **Description:** The "Quick Demo" section for `m3tal dash up` mentions it downloads "necessary dashboard compose files" and applies overrides. It would be beneficial to explicitly state that this command is for *managing the dashboard container only* and that `m3tal up` is for deploying all other services.
    *   **Impact:** Users might confuse the scope of `m3tal dash up` with `m3tal up`.
    *   **Required Fix:** Add a sentence to the `m3tal dash up` description to clarify its singular purpose of managing the dashboard container, distinct from `m3tal up`.

---

### Required Fixes:

1.  **APT Installation Command Block:**
    Replace the existing three-step APT installation section with the following:
    ```bash
    # Install M3TAL via APT
    curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list
    sudo apt update && sudo apt install -y m3tal
    ```

2.  **Docker Dependency Statement:**
    In the "Prerequisites" section, modify the sentence to:
    "Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed and functioning **prior to M3TAL installation and operation**. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2."

3.  **Deployment Lifecycle - `m3tal up` Behavior:**
    Modify the "Deployment Lifecycle" section to:
    "M3TAL orchestrates Docker containers using Docker Compose V2. The `m3tal up` command is a wrapper around `docker compose` that operates on all `*-compose.yml` files located within the `/docker/` directory. This directory is a symlink to `/opt/m3tal/stack/`, which is the canonical source of truth for all Docker Compose stack files and associated configurations. Running `m3tal up` will deploy all services defined across these compose files, including M3TAL's core components and any user-added stacks."

4.  **Traefik Routing Configuration Detail:**
    In the "Traefik Gateway" section, add the following:
    "Traefik discovers and routes traffic to Docker services in two primary ways:
    *   **Docker Labels:** By interpreting Traefik labels defined directly within the Docker Compose service definitions. This is the most common method for exposing services within the Docker network.
    *   **File Provider:** Traefik also utilizes a file provider for dynamic configuration, reading routing rules and service definitions from files in `/etc/traefik/dynamic`. This is crucial for routing traffic to services not running as Docker containers or to host-local daemons (e.g., the M3TAL API daemon via `dynamic/api.yml`).

    Traefik itself is configured via static configuration (`/etc/traefik/traefik.yml`) and dynamic configurations loaded from the specified directory. Services are not exposed by Traefik by default and require explicit configuration through labels or dynamic files."

5.  **Port Table Missing Specificity for Access:**
    Update the "Port Map" table as follows:

    | Port | Service | Access | Description |
    |------|---------|--------|-------------|
    | 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
    | 8080 | M3TAL API daemon (Go) | Host-local | The internal port the M3TAL API daemon listens on. |
    | 8081 | Traefik dashboard | Host-local only | The internal Traefik dashboard port, accessible only from the host machine. |
    | 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

6.  **Firewall Note Granularity:**
    In the "Firewall Considerations" section, modify the sentence to:
    "If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall (e.g., `ufw allow 80/tcp` and `ufw allow 443/tcp`)."

7.  **Marketing Copy Tone:**
    In the "Overview" section, rephrase the first sentence to:
    "This document details the technical architecture and operational procedures for the M3TAL system."

8.  **Quick Demo Clarity on `m3tal dash up`:**
    In the "Quick Demo" section, under the `m3tal dash up` bullet point, add the following clarification:
    "This command is used exclusively to manage the M3TAL dashboard container and applies the correct override based on your `DASHBOARD_EXPOSE_MODE` setting. For deploying all other M3TAL services and user-defined stacks, use `m3tal up`."