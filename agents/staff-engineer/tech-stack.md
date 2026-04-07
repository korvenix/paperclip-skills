### **Core Engineering Principles**

*These directives are non-negotiable and apply strictly to everyone*

**1. Infrastructure as Code (IaC)**

* **Standard:** 100% of infrastructure must be provisioned and managed using **Terraform**. No manual console provisioning.

**2. Environment Strategy**

* **Isolation:** Strict separation of GCP projects across 3 environments and 1 tooling project per product:
* **Naming Convention:** `korvenix-{product-name}-{env}` (e.g., `korvenix-core-tooling, korvenix-core-dev`, `korvenix-core-test`, `korvenix-core-prod`).
* **Governance:** Organization policies are defined at the folder/org level and strictly inherited by all projects.

**3. Design System (UI/UX)**

* **Standard:** **Google Material 3 (M3)** is mandatory for all user-facing products.
* **Implementation:** Use M3-compliant component libraries (e.g., Material Web, MUI v5+). Must strictly adhere to M3 guidelines for dynamic color, typography, motion, and accessibility.

**4. Technology Stack**

* **Production Core:** Use proven, highly extensible frameworks (e.g., FastAPI, Next.js).
* **Cloud:** Use Google Cloud always, do not use a product that competes with GCP.
* **Governance:** Adopting new technologies requires board approval.

**5. GCP Resource Standards**

* **Compute:** Default to **Cloud Run** unless specific needs dictate otherwise.
* **Data:** Use **Cloud SQL with private IP** for relational data; use **Firestore** for document storage.
* **Networking:** All workloads must be deployed within a VPC using **private subnets**.
* **Security & IAM:**
  * Use **Workload Identity Federation** (never use static Service Account keys).
  * Enable only required APIs (Principle of Least Privilege).
  * All secrets must be stored in **Secret Manager** (never in source control or flat `.env` files).
* **Tagging:** All GCP resources must include the following labels: `env`, `product`, `team`, `managed-by: terraform`.

**6. CI/CD & Quality (Added for Completeness)**

* **Automation:** All infrastructure and application deployments must execute via automated CI/CD pipelines using cloud Build. Assets are promoted between environemnts. CI/CD tooling in Tooling project. No manual deployments.
* **Validation:** All code changes require mandatory peer review (PRs) by CTO and must pass automated unit/integration tests before merging.

**7. Observability & Monitoring (Added for Completeness)**

* **Telemetry:** All production services must emit structured RFC Compliant logs and core metrics to cloud logging