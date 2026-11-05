# M3TAL Environment Variables Reference

This document provides a comprehensive reference for all environment variables used by the M3TAL ecosystem. These variables are typically managed via the `m3tal config wizard` or by directly editing the `/etc/m3tal/.env` file. Both the M3TAL CLI and all Docker Compose stacks read these variables using `--env-file /etc/m3tal/.env`.

## Quick Reference Table

| Variable Name       | Description                                                                                                                                                                          | Default      | Example Value                                         | Used By      |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|-------------------------------------------------------|--------------|
| `API_PORT`          | The port on which the M3TAL API daemon listens.                                                                                                                                      | `5050`       | `8080`                                                | API Daemon   |
| `API_TOKEN`         | An API token used for authenticating requests to the M3TAL API. Auto-generated on first `m3tal init`. Do not set manually unless rotating.                                            | `""`         | `your-secure-api-token-here`                          | API Daemon   |
| `M3TAL_STATE_DB`    | The filesystem path to the M3TAL state database.                                                                                                                                     | `""`         | `/var/lib/m3tal/state.db`                             | API Daemon   |
| `M3TAL_PLUGINS_DIR` | The directory where M3TAL plugins are located.                                                                                                                                       | `""`         | `/opt/m3tal/plugins`                                  | API Daemon   |

---

## Environment Variable Details

### Core

#### `API_PORT`

*   **Description:** The port on which the M3TAL API daemon listens.
*   **Default Value:** `5050`
*   **Example Value:** `8080`
*   **Used By:** API Daemon

#### `API_TOKEN`

*   **Description:** An API token used for authenticating requests to the M3TAL API. This token is **auto-generated on the first `m3tal init`**. Users should **not** set this manually unless they are rotating an existing token.
*   **Default Value:** `""`
*   **Example Value:** `your-secure-api-token-here`
*   **Used By:** API Daemon

#### `M3TAL_STATE_DB`

*   **Description:** The filesystem path to the M3TAL state database. This is where M3TAL stores its operational data.
*   **Default Value:** `""`
*   **Example Value:** `/var/lib/m3tal/state.db`
*   **Used By:** API Daemon

#### `M3TAL_PLUGINS_DIR`

*   **Description:** The directory where M3TAL plugins are located. Plugins extend M3TAL's functionality.
*   **Default Value:** `""`
*   **Example Value:** `/opt/m3tal/plugins`
*   **Used By:** API Daemon

---

**Note:** The provided JSON did not contain variables for Auth, Network, Storage, Traefik, VPN, or System. As per the instructions, only variables present in the JSON are documented. If these categories were intended to have documented variables, they would need to be added to the "Environment Variables State JSON" in the future.