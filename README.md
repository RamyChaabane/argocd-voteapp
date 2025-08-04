# 🚀 argocd-voteapp

This repository defines the **GitOps deployment layer** for the VoteApp project. It contains all Kubernetes manifests for deploying and managing the VoteApp stack across `dev`, `staging`, and `production` environments using **Argo CD** and **Kustomize**.

---

## 📦 Purpose

This repository is the single source of truth for Argo CD. When new commits are pushed here, **Argo CD automatically detects the changes, validates them, and applies them to the target Kubernetes cluster**. It manages all services including:

- 🐳 Backend API (Go, Clean Architecture)
- 🌐 Frontend SPA (Vue.js + NGINX)
- 🧠 Redis caching layer
- 🔭 Full observability stack (Prometheus + Grafana)

It complements:

- [📁 provisioning-voteapp](https://github.com/RamyChaabane/provisioning-voteapp): Infrastructure-as-Code provisioning with Terraform.
- [💻 vote-app](https://github.com/RamyChaabane/VoteApp): The application source code and CI pipeline.

---

## 🗂️ Repository Structure
```
argocd-voteapp/
├── k8s/
│ ├── base/ # Common resources shared across environments
│ │ ├── backend/ # Go backend Deployment, Service, HPA
│ │ ├── frontend/ # Vue frontend Deployment, Service, HPA
│ │ ├── redis/ # Redis Deployment, Service
│ │ ├── ingress.yaml # Shared base ingress
│ │ └── kustomization.yaml # Base kustomization
│ └── overlays/ # Environment-specific customizations
│ ├── dev/
│ ├── stg/
│ └── prd/
├── monitoring/ # Monitoring stack: Prometheus, Grafana, etc.
│ ├── helm-values.yaml
│ └── kustomization.yaml
├── renovate.json # Renovate configuration for image automation
└── README.md
```

---

## 🌐 Multi-Environment Deployments

This repository follows the **Kustomize overlay pattern** to manage multiple environments using environment-specific patches:

| Environment | Path                    | Examples of Differences         |
|-------------|--------------------------|---------------------------------|
| `dev`       | `k8s/overlays/dev/`      | No autoscaling, internal domain |
| `stg`       | `k8s/overlays/stg/`      | Staging domain, preview configs |
| `prd`       | `k8s/overlays/prd/`      | HPA enabled, more replicas, TLS |

Argo CD is configured (via the `provisioning-voteapp` repo) to monitor each overlay separately and sync changes to the correct namespace.

---

## 🔄 GitOps with Argo CD

Each environment is managed as a distinct **Argo CD Application** resource:

- Auto-sync enabled
- Prune enabled (safe by default)
- Deployments are triggered via Renovate automation

When changes are committed here, Argo CD handles reconciliation and applies only what's changed.

---

## 📈 Monitoring Stack

- **Deployed via Kustomize** in the `monitoring/` directory
- Uses **Helm-backed Kustomize patches**
- Tools included:
    - Prometheus (metrics scraping)
    - Grafana (dashboards)
    - Alertmanager (can be wired to Slack/Email)

Configured independently and GitOps-managed as well.

---

## 🤖 Automated Image Updates via Renovate

A key feature is that image versions are **not manually bumped**. Instead:

- The CI pipeline in [`vote-app`](https://github.com/RamyChaabane/VoteApp) pushes a new image version to Docker Hub with a tag format: vYYYY.MM.DD.RunID
- That triggers a custom [repository dispatch](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event) to this repo
- **Renovate** runs in this repo and creates a pull request for each updated image per environment:
  - chore(deps): update vote-backend docker tag to v2025.07.20.123
  - Labels: env::dev
- Auto-merge is only enabled for dev PRs, and Argo CD picks up the change and deploys it

This is a **fully automated CI → GitOps → CD pipeline**.

---

## 👔 Why This Project Stands Out

This repository demonstrates:

- 🔁 True GitOps: Every application change, infra change, and release is traceable and versioned
- 🧱 Clean, scalable Kustomize structure for multi-env support
- 🤖 Seamless automation: CI pipelines build → Renovate bumps images → Argo CD deploys
- 🔍 Observability-first design: Monitoring is part of the GitOps pipeline
- 🔐 Production-aware: HPA, multi-replica, safe pruning, ingress isolation
- 📦 Tooling maturity: Renovate, Argo CD, Kustomize, Helm all working together

**This setup is equivalent to what modern SRE and Platform Engineering teams build in production at scale.**

---

## 🛠️ How to Use

1. Change any manifest in `k8s/base` or `overlays/*`
2. Push to the `main` branch
3. Argo CD will auto-sync based on your commit

For image updates, push new image tags from `vote-app` CI — no manual versioning here.

---

## 🧭 Related Repositories

- [`vote-app`](https://github.com/RamyChaabane/VoteApp)              - Source code and CI/CD pipeline 
- [`provisioning-voteapp`](https://github.com/RamyChaabane/provisioning-voteapp) - Terraform for cluster & cloud provisioning

---
