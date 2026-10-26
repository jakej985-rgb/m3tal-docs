**Verdict: FAILED**

The README provides a comprehensive and technically sound overview of the M3TAL platform, covering most critical aspects for installation and operation. However, it lacks a consolidated reference for key environment variables, which is a genuine gap for users seeking to understand or manually configure the system beyond what the `m3tal config wizard` might offer interactively.

---

### Audit Issue List

1.  **Issue: Lack of Key Environment Variable Documentation**
    *   **Classification:** WARNING
    *   **Reason:** While the README correctly identifies `/etc/m3tal/.env` as the primary configuration file and mentions `m3tal config wizard` for its management, it fails to list and briefly describe the most important environment variables (e.g., `DOMAIN`, `API_PORT`, `DASHBOARD_PORT`, `BASE_STORAGE_PATH`, `PUID`, `PGID`, `TZ`, `TRAEFIK_WEB_PORT`) that a user might need to set or understand. The `GROUND TRUTH`'s `.env.example` shows that such a reference is an integral part of the system's intended configuration documentation. Without this, users would have to run the `config wizard` blindly, infer from system behavior, or inspect source/example files to understand the full scope and purpose of M3TAL's configurable options. This makes the documentation incomplete for comprehensive operational understanding and manual configuration.
    *   **Required Fix:** Add a dedicated "Configuration Variables" section to the README. This section should list and briefly describe all significant environment variables that can be configured in `/etc/m3tal/.env`, along with their default values or typical uses. This list should be derived from the `.env.example` provided in the `GROUND TRUTH` and include, but not be limited to: `API_PORT`, `DASHBOARD_PORT`, `DASHBOARD_EXPOSE_MODE`, `HTTP_PORT`, `DOMAIN`, `NETWORK_NAME`, `LOCAL_IP`, `BASE_STORAGE_PATH`, `CONFIG_PATH`, `PUID`, `PGID`, `TZ`, `TRAEFIK_WEB_PORT`, `TRAEFIK_WEBHTTPS_PORT`, `DASHBOARD_SECRET`, `API_TOKEN`, and `ADMIN_PASSWORD`.

---

**Summary of Passed Criteria:**

The README successfully addresses the following audit criteria:

*   **APT installation:** The 3-command keyring+repo+install block is present and correct.
*   **Docker dependency:** Clearly states Docker Engine + Docker Compose V2 are required.
*   **Deployment lifecycle:** Explains how stacks work (`/docker` dir, `m3tal up`, adding new compose files) comprehensively.
*   **Traefik routing:** Clearly explains Traefik as the HTTP gateway and how services are exposed via labels and dynamic configuration, including examples.
*   **Port table:** Lists ports 80, 8080, 8081, 8082 with descriptions and access methods.
*   **Service management:** Mentions `systemctl` for `m3tal-api.service` and provides essential commands.
*   **Firewall note:** Reminds users to allow port 80 (and 443) in ufw/iptables for public access.
*   **Tone:** The writing is professional, technical, and avoids marketing copy.
*   **Quick demo:** Includes a working Quick Start section that leverages `m3tal dash up` and explains dashboard access modes.