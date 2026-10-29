# 📱 Applications Directory (`apps/`)

This directory houses all Kubernetes application workloads deployed to the homelab. Workload definitions follow the declarative **Kustomize Base + Environment Overlay** pattern, continuously reconciled by **Flux CD**.

---

## 🏗 Directory Architecture

```text
apps/
├── README.md                      # This documentation guide
├── base/                          # Reusable, environment-agnostic application definitions
│   ├── linkding/                  # Linkding bookmark manager base
│   │   ├── deployment.yaml        # Deployment spec (2 replicas, image v1.31.0)
│   │   ├── service.yaml           # ClusterIP service on port 9090
│   │   ├── storage.yaml           # 1Gi PersistentVolumeClaim
│   │   ├── namespace.yaml         # Dedicated 'linkding' namespace
│   │   ├── kustomization.yaml     # Kustomize resource list
│   │   └── README.md              # In-depth Linkding documentation
│   └── mealie/                    # Mealie recipe manager base
│       ├── deployment.yaml        # Deployment spec (2 replicas, image v1.3.0)
│       ├── service.yaml           # ClusterIP service on port 9000
│       ├── storage.yaml           # 500Mi PersistentVolumeClaim
│       ├── namespace.yaml         # Dedicated 'mealie' namespace
│       ├── kustomization.yaml     # Kustomize resource list
│       └── README.md              # In-depth Mealie documentation
├── staging/                       # Staging overlays applied to the local cluster
│   ├── linkding/                  # References ../../base/linkding/
│   │   └── kustomization.yaml
│   └── mealie/                    # References ../../base/mealie/
│       └── kustomization.yaml
└── production/                    # Production overlays for cloud cluster (AKS / future)
    └── linkding/
```

---

## 📦 Application Catalog

| Application | Namespace | Base Path | Staging Path | Upstream Image | Service Port | PVC Size |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| **[Linkding](base/linkding/README.md)** | `linkding` | `apps/base/linkding` | `apps/staging/linkding` | `sissbruecker/linkding:1.31.0` | `9090` | `1Gi` |
| **[Mealie](base/mealie/README.md)** | `mealie` | `apps/base/mealie` | `apps/staging/mealie` | `ghcr.io/mealie-recipes/mealie:v1.3.0` | `9000` | `500Mi` |

---

## 🔄 How Flux CD Reconciles Applications

In `clusters/staging/apps.yaml`, Flux CD registers a `Kustomization` resource configured as follows:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 1m0s
  path: ./apps/staging
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
```

### Automatic Subdirectory Discovery
When Flux reconciles `./apps/staging`, the Flux `kustomize-controller` scans the directory tree. Each subdirectory containing a `kustomization.yaml` (such as `apps/staging/linkding` and `apps/staging/mealie`) is built, assembled into a unified manifest stream, and applied to the cluster.

---

## 🚀 How to Add a New Application

Follow these steps to deploy a new application into the homelab:

### Step 1: Create Base Manifests
Create a new directory in `apps/base/<app-name>/`:

```bash
mkdir -p apps/base/my-app
```

Add the standard resource files:
- **`namespace.yaml`**:
  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: my-app
  ```
- **`deployment.yaml`**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-app
    namespace: my-app
    labels:
      app: my-app
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: my-app
    template:
      metadata:
        labels:
          app: my-app
      spec:
        securityContext:
          runAsUser: 1000
          runAsGroup: 1000
          fsGroup: 1000
        containers:
          - name: my-app
            image: my-image:tag
            ports:
              - containerPort: 8080
            securityContext:
              allowPrivilegeEscalation: false
  ```
- **`service.yaml`**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-app
    namespace: my-app
  spec:
    type: ClusterIP
    selector:
      app: my-app
    ports:
      - port: 8080
  ```
- **`storage.yaml`** *(optional)*:
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-app-data-pvc
    namespace: my-app
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  ```
- **`kustomization.yaml`**:
  ```yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization
  resources:
    - namespace.yaml
    - deployment.yaml
    - service.yaml
    # - storage.yaml
  ```

### Step 2: Create Staging Overlay
Create `apps/staging/<app-name>/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base/my-app/
```

*(Optional: Add patches, replica count adjustments, or environment variables specific to staging).*

### Step 3: Validate the Kustomize Build Locally
Test that your Kustomize files render valid YAML before pushing:

```bash
kubectl kustomize apps/staging/my-app
```

### Step 4: Commit and Push
Push the new files to the `main` branch. Flux will detect the new directory in `./apps/staging` and deploy it within 1 minute.

---

## 🔒 Security & Sizing Best Practices

1. **Explicit Namespaces**: Never deploy workloads to the `default` namespace. Every app must own its namespace.
2. **Non-Root Containers**: Always configure `securityContext` (`runAsUser`, `runAsGroup`, `fsGroup`) and set `allowPrivilegeEscalation: false`.
3. **Storage Bounds**: Use modest PVC storage requests (`500Mi` to `2Gi`) suitable for local experimentation backed by `local-path-provisioner`.
4. **Replica Sizing for Embedded Databases**: Applications relying on SQLite files inside a `ReadWriteOnce` volume should run with `replicas: 1` to prevent database locks and race conditions unless connected to an external RDBMS like PostgreSQL.
