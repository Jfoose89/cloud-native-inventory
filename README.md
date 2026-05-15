# CloudNativeInventory

A cloud-native Inventory REST API built with **.NET 9**, containerized with **Docker**, and deployed to **Azure Container Apps** via a **GitHub Actions** CI/CD pipeline. Secrets are managed securely through **Azure Key Vault** using **Managed Identity**.

---

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/inventory` | Returns all products in the inventory |
| GET | `/api/inventory/system/verify-integration` | Verifies that the secret was loaded securely from Key Vault |

---

## Running Locally

**Prerequisites:** .NET 9 SDK, Docker Desktop

**Run the API:**
```bash
dotnet run --project CloudNativeInventory.Api
```

**Run with Docker:**
```bash
docker-compose up --build
```

**Run tests:**
```bash
dotnet test
```

---

## Architecture

- **CloudNativeInventory.Api** — ASP.NET Core 9 Web API with controllers, EF Core InMemory database
- **CloudNativeInventory.Tests** — xUnit test project using Moq

---

## Azure Infrastructure

| Resource | Name | Purpose |
|----------|------|---------|
| Resource Group | `rg-Jordan-dev` | Container for all resources |
| Container Registry | `jorjandev.azurecr.io` | Stores Docker images |
| Container App | `ca-jordan-dev` | Hosts the running container |
| Key Vault | `kv-jordan-dev` | Stores the VendorApiKey secret |

---

## CI/CD Pipeline

- **ci.yml** — Runs on every push and PR: restore, build, test
- **deploy.yml** — Runs on push to main: builds Docker image, pushes to ACR, deploys to Container Apps

---

## Security

- Secrets are stored in **Azure Key Vault** and never committed to the repository
- The Container App uses a **System-assigned Managed Identity** with the **Key Vault Secrets User** role
- The container runs as a **non-root user** (`USER app`) to reduce attack surface
