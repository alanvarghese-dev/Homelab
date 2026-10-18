# 🏠 Kubernetes Homelab

A declarative, GitOps-driven homelab designed for **Kubernetes tinkering and experimentation**—prioritizing practical, cost-effective, and low-friction workflows across local environments and cloud infrastructure. Provisioned with **Terraform**, reconciled continuously with **Flux CD**, and developed inside reproducible **DevPod** containers backed by **Rancher Desktop**.

---

> **🎯 Purpose**: Provide a clean, reproducible sandbox for testing Kubernetes workloads, networking, and GitOps workflows locally and in the cloud with zero manual drift.

---

## 🛠 Tech Stack & Tools

| Tool | Purpose |
| :--- | :--- |
| **[Kubernetes](https://kubernetes.io/)** | Container orchestration runtime (Local k3s & Cloud AKS) |
| **[Rancher Desktop](https://rancherdesktop.io/)** | Host container engine & local Kubernetes provider (exposing `docker.sock`) |
| **[Flux CD](https://fluxcd.io/)** | GitOps continuous delivery and cluster state synchronization |
| **[Terraform](https://www.terraform.io/)** | Infrastructure as Code (IaC) for Azure cloud provisioning |
| **[DevPod](https://devpod.sh/)** | Reproducible, containerized developer environments (`.devcontainer`) |
| **[Mise](https://mise.jdx.dev/)** | Runtime and CLI tool manager (`helm`, `gh`, `docker-cli`, `uv`, etc.) |

---

## 📁 Repository Structure

```text
.
├── .devcontainer/                             # DevPod / DevContainer configuration (binds docker.sock)
├── Azure-infrastructure-provisioning-terraform/ # Terraform IaC for Azure cloud infrastructure
├── apps/                                      # Application workloads
│   ├── base/                                  # Common Kustomize bases
│   ├── staging/                               # Staging overlays
│   └── production/                            # Production overlays
├── clusters/                                  # Flux CD cluster root entrypoints
│   ├── staging/                               # Staging cluster configuration
│   └── production/                            # Production cluster configuration
├── infrastructure/                            # Core cluster infrastructure & dependencies
│   ├── configs/                               # Cluster-wide configurations
│   └── controllers/                           # Ingress, cert-manager, external-dns, etc.
├── monitoring/                                # Observability stack
│   ├── configs/                               # Dashboards, alerts, and scrape configs
│   └── controllers/                           # Prometheus, Grafana, Loki, etc.
├── docs/                                      # Architecture notes and homelab documentation
└── mise.toml                                  # Tool versioning configuration
```

---

## 🚀 Getting Started

> [!NOTE]
> This repository is in the initial setup phase. Follow the steps below to prepare your environment.

### 1. Prerequisites

Make sure you have the following installed on your host machine:
- **[Rancher Desktop](https://rancherdesktop.io/)**: Acts as the host container engine (dockerd/moby) and local Kubernetes provider. Ensure the Docker socket (`/var/run/docker.sock`) is accessible.
- **[DevPod](https://devpod.sh/)**: Drives the development workspace container.
- **[Mise](https://mise.jdx.dev/)** (optional): Manages CLI tool versions defined in `mise.toml`.

Install local CLI tools using Mise:
```bash
mise install
```

### 2. Development Environment (DevPod)

Launch the workspace in DevPod. DevPod builds the container from [`.devcontainer/Dockerfile`](file:///Users/iti/Documents/My%20Mac%20Docu/HOMELAB/.devcontainer/Dockerfile) and binds the host's `/var/run/docker.sock` from Rancher Desktop, giving you full Docker and Kubernetes access inside the container:
```bash
devpod up .
```

### 3. Infrastructure Provisioning (Terraform)

Provision the cloud resources on Azure:
```bash
cd Azure-infrastructure-provisioning-terraform
terraform init
terraform plan
terraform apply
```

### 4. GitOps Bootstrap (Flux CD)

Bootstrap Flux onto your Kubernetes cluster to begin continuous reconciliation:
```bash
flux bootstrap github \
  --owner=<github-user-or-org> \
  --repository=HOMELAB \
  --branch=main \
  --path=clusters/production \
  --personal
```

---

## 🔄 GitOps Workflow

1. **Commit & Push**: Push changes to the `main` branch (or via PR).
2. **Reconciliation**: Flux CD detects changes in Git repository manifests.
3. **State Enforcement**: Flux reconciles any drift between the Git repository and the live cluster state.

---

## 📋 Status & Roadmap

- [x] Initial repository structure and devcontainer setup
- [ ] Azure infrastructure provisioning via Terraform
- [ ] Kubernetes cluster bootstrapping with Flux CD
- [ ] Core infrastructure services setup (cert-manager, ingress-nginx)
- [ ] Observability & monitoring stack deployment
- [ ] Application deployment pipeline
