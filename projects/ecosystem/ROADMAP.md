# Ecosystem Roadmap

This document outlines the strategic timeline and development goals for the M3TAL platform.

---

## Phase 1: Foundation & Structuring [COMPLETED]
* Define and deploy the multi-repository web presence structure.
* Migrate documentation to a single repository `m3tal-docs`.
* Establish the shared premium dark-theme stylesheet (`m3tal.css`) and global loading script (`m3tal.js`).
* Interlink the portal home, package server, and plugin registry.

---

## Phase 2: CLI Integration [IN PROGRESS]
* Integrate `m3tal-core` CLI commands to dynamically load plugin listings from `/m3tal-plugin-page/plugins.json`.
* Implement GPG signature validation for the plugins schema.
* Introduce an automated setup wizard to link local Docker instances to the new `m3tal-project` Google Cloud environment.

---

## Phase 3: Developer Tools & Extensions [Q3 2026]
* Launch a standardized boilerplate repository for third-party plugin creators.
* Add support for system configuration backup to cloud storage buckets (directly utilizing the new Google Cloud Platform billing setup).
* Implement real-time service metrics reporting via Vertex AI anomalies detection.

---

## Phase 4: Decentralized Control Console [Q4 2026]
* Introduce a mobile-responsive dashboard client.
* Support multi-node Docker swarm and Kubernetes clustering configurations from the same configuration file.
