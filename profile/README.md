# PaperTimes

### **Architecting the Future of B2B Isolation & Scalability.**

Welcome to the **PaperTimes** GitHub Organization. We build high-performance, multi-tenant infrastructure using a unique **Global-to-Local** architecture. Our mission is to provide enterprise-grade data isolation for every client while maintaining a centralized, high-velocity developer experience.

---

## 🏗️ The "Global-to-Local" Philosophy

We don't believe in "noisy neighbor" problems. Every client on the PaperTimes platform gets their own dedicated stack, orchestrated from a central command center.

* **Global Command Center:** A centralized `papertimes-global` project (Convex + WorkOS + TanStack) that manages billing, provisioning, and global metadata.
* **Isolated Client Silos:** Each client is deployed as an independent Railway project with a dedicated Convex backend—ensuring physical and logical data residency.
* **Ephemeral Previews:** Every Pull Request spins up a temporary, full-stack environment including a preview Convex deployment.

---

## 🛠️ The Stack

| Layer | Technology |
| --- | --- |
| **Package Manager** | [pnpm](https://www.google.com/search?q=https://pnpm.io/) |
| **Cloud Infra & Orchestration** | [Railway](https://www.google.com/search?q=https://railway.app/) |
| **Backend / DB** | [Convex](https://www.google.com/search?q=https://www.convex.dev/) |
| **Auth & SSO** | [WorkOS](https://www.google.com/search?q=https://workos.com/) |
| **Frontend** | [Vite](https://www.google.com/search?q=https://vitejs.dev/) + [TanStack (Router/Query)](https://www.google.com/search?q=https://tanstack.com/) |
| **Monitoring** | Railway |

---

## 📂 Core Repositories

* **[`ops-handbook`](https://www.google.com/search?q=%23):** The source of truth. Contains the `infra/` specifications, pnpm workspace scripts, and the `client-template` used for provisioning.
* **[`client-app`](https://www.google.com/search?q=%23):** The template for `[env].[client].papertimes.ca`. Highly optimized Vite + TanStack frontend designed for speed and modularity.





---

## 🌈 Contribution Guidelines

We maintain a strict **Feature Branch** workflow:

* **`feat/*`**: Development and Ephemeral PR environments.
* **`develop`**: Automatically deploys to the **Staging** environment for QA.
* **`main`**: The source of truth for **Production**.


---

## 🧙 Useful Resources

* **Docs:** Internal documentation within each project.
* **Railway Projects:** Check the Railway dashboard for environment health.
* **Convex Dashboard:** For real-time data inspection and mutation logs.

---

