# ADR - Architecture Decision Record
## CloudNativeInventory - Jordan Foose

---

## 1. Azure Hosting: Container Apps vs App Service

**Decision:** Azure Container Apps

**Reasons:**
- Designed natively for containers — no runtime stack selection needed
- Serverless scaling, scales to zero when not in use (cost effective for student subscription)
- Built-in support for Managed Identity and environment variables
- App Service treats containers as secondary to code deployments

**Alternatives considered:** Azure App Service — rejected because it is primarily designed for code deployments and adds unnecessary abstraction over our Docker image.

---

## 2. Container Registry: Azure Container Registry (ACR)

**Decision:** Azure Container Registry (jorjandev.azurecr.io)

**Reasons:**
- Native integration with Azure Container Apps and Managed Identity
- Images stay within the Azure ecosystem, no external dependency
- Admin user credentials can be stored securely in GitHub Secrets

---

## 3. Secret Management: Azure Key Vault

**Decision:** Azure Key Vault (kv-jordan-dev)

**Reasons:**
- Separates secrets from code and version-controlled files
- The secret ExternalServices--VendorApiKey is stored in Key Vault and never committed to the repo
- App reads secrets at runtime via DefaultAzureCredential

---

## 4. Identity & Permissions: Managed Identity + RBAC

**Decision:** System-assigned Managed Identity on the Container App with Key Vault Secrets User role

**Reasons:**
- No credentials stored anywhere — the app authenticates to Key Vault automatically
- Principle of least privilege: only the Key Vault Secrets User role is granted, not full access
- If the Container App is deleted, the identity is deleted with it (no orphaned credentials)

---

## 5. Container Security: Rootless + Multi-stage Build

**Decision:** USER app directive in Dockerfile, multi-stage build

**Reasons:**
- Running as non-root reduces attack surface — a compromised container cannot modify system files
- Multi-stage build means the final image contains only the runtime and published artifacts, not the SDK or build tools
- Smaller image = smaller attack surface and faster pulls

---

## 6. CI/CD Pipeline Design

**Decision:** GitHub Actions with three separate jobs: build-and-test, build-and-push, deploy

**Reasons:**
- Tests must pass before an image is built (quality gate)
- Image is only pushed if tests are green
- Deploy only runs on main branch, not on pull requests
- Each job has a single responsibility, making failures easy to diagnose
- Images are tagged with both latest and the commit SHA for full traceability
