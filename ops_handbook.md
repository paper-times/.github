# 📑 Ops-Handbook: PaperTimes Infrastructure

This document outlines the operational specifications for the PaperTimes platform, utilizing a **Global-to-Local** Railway architecture.

---

## 🏗️ 1. Architecture Overview

### A. The Global Project (`papertimes-global`)

**Goal:** Centralized "Command Center" for the PaperTimes team.

* **Admin Portal:** `admin.papertimes.ca` (vite/TanStack).
* **WorkOS Main Instance:** Manages global team auth and client organization provisioning.
* **Global Convex Backend:** Stores client metadata, billing status, and project mapping.

### B. The Client Project Template (`papertimes-client-template`)

**Goal:** A repeatable Railway project definition deployed for every new B2B client.

* **Environments:** Each client project contains its own `Prod`, `Stage`, and `Dev` environments.
* **Isolation:** Each client has a dedicated **Convex Deployment** and **WorkOS Organization**.

---

## 🌐 2. Environment Matrix (Per Client Project)

| Environment | Git Branch | Railway Env | Convex Type | WorkOS Env | URL Pattern |
| --- | --- | --- | --- | --- | --- |
| **Development** | `feat/*` | Ephemeral (PR) | Preview | Test | `[pr].[client].dev.papertimes.ca` |
| **Stage** | `develop` | Staging | Production (Stage) | Test | `stage.[client].papertimes.ca` |
| **Production** | `main` | Production | Production | Live | `app.[client].papertimes.ca` |

---

## 🛠 3. Project Configuration & Services

### 🟢 Shared Global Services

* **Core Admin:** Manages the lifecycle of client projects. Built with **TanStack Query/Router** for high-performance state management.
* **WorkOS Integration:** Handles User Management and SSO onboarding for all clients.
* **Shared Tooling:** Monitoring (Sentry/Logtail) and Cron jobs for global maintenance.

### 🔵 Client-Specific Services

Each Railway project instantiated from the template includes:

1. **Frontend (vite/TanStack):** Connects to the client's specific Convex instance.
2. **Convex Backend:** Dedicated data silo. No cross-client data leakage at the DB level.
3. **Variables:**
* `WORKOS_ORG_ID`: Unique ID for that client's enterprise features.
* `CONVEX_DEPLOY_KEY`: Unique key for the client’s backend deployment.



---

## 🚀 4. Deployment Logic

### A. Provisioning a New Client

1. **Railway:** Deploy a new project from the `papertimes-client-template`.
2. **Convex:** Create a new deployment and map the `CONVEX_DEPLOY_KEY` to the Railway project.
3. **WorkOS:** Provision a new Organization via the Global Admin.
4. **TanStack:** The frontend uses the environment variables to point TanStack Query to the correct Convex URL.

### B. Feature Envs (PRs)

Railway creates an ephemeral environment for every PR.

* **Build Command:**
```bash
npx convex deploy --cmd 'npm run build .....'

```


* **Scope:** PRs use **Convex Preview** deployments which are automatically deleted when the Railway PR environment is closed.

---

## 🔐 5. Security & Isolation

* **Data Residency:** Since each client has their own Convex deployment, data is physically and logically separated.
* **Auth Flow:**
* **User** hits `[client].papertimes.ca`.
* **TanStack Router** checks auth via **WorkOS**.
* **Middleware** validates the `client_id` against the specific Railway environment variables.


* **Global Admin Access:** The Global Admin uses a master WorkOS key to "impersonate" or manage client settings across projects.

---

---

## Observability	
Visibility	Know when something is wrong.
## Automation	
Speed/Consistency	Remove manual bottlenecks.
## Reliability	
Uptime	Meet the business's expectations.
## Security
Protection	Mitigate risk and maintain trust.
## FinOps
Efficiency	Maximize ROI of the tech stack.
