## M3TAL README Audit Report

**Verdict:** FAILED - Critical information for deployment and operation is missing or unclear.

**Issues:**

1.  **BLOCKER: APT Installation Missing 3-Command Block**
    *   **Issue:** The README states APT installation but only provides a single `sudo apt install -y m3tal` command after the keyring and repo setup. The crucial `sudo apt update` command to refresh package lists is missing from the final install block.
    *   **Required Fix:** Combine the `apt update` and `apt install` commands into a single, robust 3-command block as specified in the GROUND TRUTH.

2.  **BLOCKER: Docker Dependency Statement Incomplete**
    *   **Issue:** The README correctly states that "Docker Engine and Docker Compose V2 are strictly REQUIRED". However, it does not explicitly mention that "Docker Compose V2" is required. It only mentions "Docker Compose".
    *   **Required Fix:** Explicitly state that "Docker Engine + Docker Compose V2" are required to avoid ambiguity and potential issues with users installing older versions of Docker Compose.

3.  **BLOCKER: Deployment Lifecycle Explanation Incomplete**
    *   **Issue:** The README mentions `m3tal up` and the `/docker` directory. However, it does not clearly explain that `/docker` is a symlink to `/opt/m3tal/stack/` and that `m3tal up` operates on all `*-compose.yml` files within that directory. The explanation of how new compose files are added is present but could be more explicit about how `m3tal up` discovers them.
    *   **Required Fix:** Clarify that `/docker` is a symlink to `/opt/m3tal/stack/` and that `m3tal up` processes all `*-compose.yml` files within this directory.

4.  **BLOCKER: Traefik Routing Explanation Lacks Detail**
    *   **Issue:** The README correctly identifies Traefik as the HTTP gateway and mentions labels. However, it fails to explain that Traefik routes based on labels defined in compose files, or that dynamic configuration (e.g., `dynamic/api.yml`) is used for specific routes. The example for exposing a custom service is good, but the underlying mechanism of how Traefik discovers and routes is not fully elucidated.
    *   **Required Fix:** Elaborate on how Traefik uses labels for routing and also mention the role of dynamic configuration files (like `dynamic/api.yml`) in routing services that might not have direct Docker labels.

5.  **WARNING: Port Table Missing Ports 8080 and 8081**
    *   **Issue:** The "Port Map" section lists ports 80 and 8082, but omits ports 8080 (M3TAL Go API daemon) and 8081 (Traefik dashboard).
    *   **Required Fix:** Add ports 8080 and 8081 to the "Port Map" table with their corresponding service and access information.

6.  **WARNING: Service Management Mention Lacks Detail**
    *   **Issue:** The README correctly mentions `systemctl` for managing `m3tal-api.service` and lists status, restart, and logs. However, it doesn't provide a concrete example of how to use `systemctl status m3tal-api.service`.
    *   **Required Fix:** Provide a specific example command for checking the status of the `m3tal-api.service`, e.g., `systemctl status m3tal-api.service`.

7.  **WARNING: Firewall Note Lacks Specificity for `ufw`**
    *   **Issue:** The README mentions reminding users to allow port 80 in `ufw/iptables`. The example provided is `ufw allow 80/tcp`, which is correct, but a similar example for `iptables` would be beneficial for users who do not use `ufw`.
    *   **Required Fix:** Include a basic `iptables` command example for allowing port 80 in addition to the `ufw` example.

8.  **WARNING: Tone - Occasional Marketing Copy**
    *   **Issue:** While generally good, some phrases like "unified Go binary providing the primary command-line interface for all M3TAL operations" lean slightly towards marketing rather than purely technical documentation.
    *   **Required Fix:** Rephrase any sentences that sound like marketing copy into more direct, technical descriptions. For instance, "CLI binary (`/usr/bin/m3tal`): A Go binary installed via APT, providing the M3TAL command-line interface."

9.  **SUGGESTION: Quick Demo Clarity and Completeness**
    *   **Issue:** The "Quick Demo" section is present but could be clearer. It implies `m3tal dash up` is a standalone step before `m3tal up`. It doesn't explicitly state that `m3tal up` is what would deploy Traefik and other stacks. The access to `HOST_IP` for the dashboard in local mode is also not fully explained (e.g., how to find `HOST_IP`).
    *   **Required Fix:** Clarify the order and purpose of `m3tal dash up` and `m3tal up`. Additionally, provide a hint on how a user might determine their `HOST_IP` if they are not running locally (e.g., using `ip addr show` and looking for the primary interface IP).

---

**Detailed Breakdown of Issues:**

**1. BLOCKER: APT Installation Missing 3-Command Block**
*   **Reason:** Without `sudo apt update`, the APT system will not be aware of the newly added repository, and `apt install m3tal` will likely fail or install an outdated version if it was already cached.
*   **Required Fix:**
    ```bash
    # 1. Add the GPG signing key
    curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

    # 2. Add the APT repository
    echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

    # 3. Update package lists and install M3TAL
    sudo apt update && sudo apt install -y m3tal
    ```

**2. BLOCKER: Docker Dependency Statement Incomplete**
*   **Reason:** Users might have Docker Engine but an older version of Docker Compose (v1), which is not compatible with M3TAL. Explicitly mentioning V2 is crucial for avoiding runtime errors.
*   **Required Fix:**
    Modify the "Prerequisites" section to read:
    "**Docker Engine and Docker Compose V2 are strictly REQUIRED and must be installed prior to M3TAL installation and operation. M3TAL internally orchestrates Docker containers using Docker Engine and Docker Compose V2.**"

**3. BLOCKER: Deployment Lifecycle Explanation Incomplete**
*   **Reason:** Users need to understand the actual location of compose files and how the `m3tal up` command discovers them. The symlink relationship is a key piece of internal architecture that impacts user interaction.
*   **Required Fix:**
    In the "Deployment Lifecycle" section, after "The `/docker` directory is a symlink to `/opt/m3tal/stack/`", add:
    "The `m3tal up` command processes all files ending in `*-compose.yml` found within `/opt/m3tal/stack/` (accessed via the `/docker` symlink)."

    In "Adding a New Stack", clarify step 1:
    "1. Place your Docker Compose file (e.g., `my-stack-compose.yml`) directly into the `/opt/m3tal/stack/` directory (or the `/docker/` symlink). This file will be automatically included by `m3tal up`."

**4. BLOCKER: Traefik Routing Explanation Lacks Detail**
*   **Reason:** While labels are mentioned, the dual mechanism of label-based routing and file-based dynamic configuration needs to be explicitly stated for users to understand how services like the API are routed, or how custom dynamic configs would work.
*   **Required Fix:**
    In the "Traefik Gateway" section, after "Traefik automatically discovers and routes traffic to Docker services by interpreting Traefik labels defined within their Docker Compose service definitions.", add:
    "In addition to labels, Traefik also utilizes a file provider to load dynamic routing configurations from files located in the `/docker/dynamic/` directory (which points to `/opt/m3tal/stack/dynamic/`). This allows for advanced routing rules and configurations, such as routing requests for `api.DOMAIN` to the M3TAL Go API daemon listening on the host-local port `8080` via `http://host.docker.internal:8080`, as seen in `dynamic/api.yml`."

**5. WARNING: Port Table Missing Ports 8080 and 8081**
*   **Reason:** Incomplete port information can lead to confusion about which ports are actively used and for what purpose, especially for internal services.
*   **Required Fix:**
    Update the "Port Map" table to include:
    | Port | Service | Access | Description |
    |------|---------|--------|-------------|
    | 80 | Traefik HTTP entry point | Public | The public-facing HTTP port for services exposed via Traefik. |
    | **8080** | **M3TAL API daemon (Go)** | **Host-local** | **The internal port the M3TAL API daemon listens on.** |
    | **8081** | **Traefik dashboard** | **Host-local only** | **The internal Traefik dashboard port, accessible only from the host machine.** |
    | 8082 | M3TAL Dashboard | Direct port (local mode) or via Traefik (traefik mode) | The port the M3TAL Dashboard container listens on internally. Access method depends on `DASHBOARD_EXPOSE_MODE`. |

**6. WARNING: Service Management Mention Lacks Detail**
*   **Reason:** While the commands are listed, a concrete example makes it easier for users to immediately verify the service's status.
*   **Required Fix:**
    In the "Service Management" section, add a specific example:
    "You can manage it using standard `systemctl` commands:
    *   **Check status**: `systemctl status m3tal-api.service` (e.g., `sudo systemctl status m3tal-api.service`)
    *   **Restart service**: `systemctl restart m3tal-api`
    *   **View logs**: `journalctl -u m3tal-api -f`"

**7. WARNING: Firewall Note Lacks Specificity for `ufw`**
*   **Reason:** Users might only be familiar with `iptables` and would benefit from a direct example rather than just a mention.
*   **Required Fix:**
    In the "Firewall Considerations" section, modify the sentence to include an `iptables` example:
    "If you are using Traefik for public-facing access (i.e., `DASHBOARD_EXPOSE_MODE=traefik` or exposing other services via Traefik), ensure that host port `80` (and `443` if HTTPS is configured) is open in your firewall. For `ufw`, this would be `sudo ufw allow 80/tcp`. For `iptables`, a basic command would be `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`."

**8. WARNING: Tone - Occasional Marketing Copy**
*   **Reason:** Technical documentation should be direct and objective. Phrases that sound like marketing can detract from the technical clarity.
*   **Required Fix:**
    In the "Core Components" section, rephrase:
    *   "CLI binary (`/usr/bin/m3tal`): A unified Go binary providing the primary command-line interface for all M3TAL operations, installed via APT."
    to:
    "CLI binary (`/usr/bin/m3tal`): A Go binary installed via APT, providing the M3TAL command-line interface."

**9. SUGGESTION: Quick Demo Clarity and Completeness**
*   **Reason:** The demo steps could be more sequential and clearer about the purpose of each command. Providing hints for `HOST_IP` would greatly improve usability for non-localhost setups.
*   **Required Fix:**
    Update the "Quick Demo" section to read:
    "## Quick Demo

    To quickly get started with M3TAL and see its core functionality:

    1.  After installation, ensure Docker and Docker Compose V2 are running.
    2.  **Start the M3TAL Dashboard specifically:**
        ```bash
        m3tal dash up
        ```
        This command downloads the necessary dashboard Docker Compose files, applies the default `local` access mode (which uses direct port binding), and starts the dashboard container.
    3.  **Access the dashboard directly:**
        Open your web browser and navigate to `http://HOST_IP:8082`.
        *   If you are running M3TAL on the same machine where you are browsing, `HOST_IP` can often be `localhost` or `127.0.0.1`.
        *   If browsing from a different machine on your local network, `HOST_IP` will be the IP address of your M3TAL server (e.g., `192.168.1.100`). You can find this IP address by running `ip addr show` on the M3TAL server and looking for your primary network interface's IP.
    4.  **Deploy the Traefik gateway and other stacks:**
        To deploy the Traefik gateway and any other user-defined Docker Compose stacks located in `/docker/` (which is a symlink to `/opt/m3tal/stack/`), execute:
        ```bash
        m3tal up
        ```
        This command orchestrates and deploys all `*-compose.yml` files in the `/opt/m3tal/stack/` directory, including the `routing-compose.yml` for Traefik and any user-defined compose files you may have added. After this, services exposed via Traefik can be accessed via their configured domain names."