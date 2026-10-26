DocCritic Audit Report: M3TAL Platform README.md

**Verdict: FAILED**

**Reason:** The README contains a critical factual error regarding core component communication (Dashboard to API daemon port), which could severely hinder debugging and understanding of the system. Additionally, minor documentation quality issues were identified.

---

### Issue List:

**1. Dashboard-API Communication Port Mismatch**
*   **Classification:** BLOCKER
*   **Description:** The README states that the M3TAL Dashboard container communicates with the API daemon on port `8080`. However, the `m3tal-compose.yml` provided in the Ground Truth clearly specifies `GO_API_URL=${GO_API_URL:-http://host.docker.internal:5050}`, indicating that the dashboard attempts to connect to the API on port `5050`. This discrepancy will lead to confusion and potential debugging issues for users if they refer to the documentation for internal connectivity.
*   **Required Fixes:**
    *   Update the "Components" section of the README to accurately reflect the dashboard's connection port to the API daemon. Change `http://host.docker.internal:8080` to `http://host.docker.internal:5050` for the dashboard's API communication description.
    *   *(Self-correction - note for M3TAL Devs: There also appears to be an inconsistency within the Ground Truth itself where the API daemon is stated to listen on 8080, but the dashboard container attempts to connect to 5050. This underlying system issue should also be addressed to ensure consistency between API daemon's listening port and the dashboard's configured API URL.)*

**2. Duplicate "Prerequisites" Section**
*   **Classification:** WARNING
*   **Description:** The "Prerequisites" section appears twice in the README, once after "Overview" and again after "Filesystem Contract". This repetition is redundant and makes the document less professional and potentially confusing for readers.
*   **Required Fixes:**
    *   Remove one of the "Prerequisites" sections, ideally keeping the first instance to introduce the dependencies early in the document.

---