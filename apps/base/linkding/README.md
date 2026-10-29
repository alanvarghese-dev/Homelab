# 🔖 Linkding Application

[Linkding](https://github.com/sissbruecker/linkding) is a fast, minimalist, self-hosted bookmark manager designed to be clean, responsive, and easy to use.

---

## 📋 Overview & Specification

| Property | Value |
| :--- | :--- |
| **Namespace** | `linkding` |
| **Container Image** | `sissbruecker/linkding:1.31.0` |
| **Internal Port** | `9090` |
| **Service Type** | `ClusterIP` |
| **Current Replicas** | `1` |
| **Persistent Storage** | `1Gi` (`linkding-data-pvc`) |
| **Data Mount Path** | `/etc/linkding/data` |
| **Run As User/Group** | `33:33` (`www-data`) |
| **Privilege Escalation**| `false` |

---

## 📁 Manifests Breakdown

- **[`namespace.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/linkding/namespace.yaml)**: Defines the dedicated `linkding` namespace for workload isolation.
- **[`deployment.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/linkding/deployment.yaml)**: Defines the `linkding` Deployment:
  - Pulls image `sissbruecker/linkding:1.31.0`.
  - Exposes container port `9090`.
  - Configures non-root security context (`runAsUser: 33`, `runAsGroup: 33`, `fsGroup: 33` for `www-data`).
  - Disallows privilege escalation (`allowPrivilegeEscalation: false`).
  - Mounts the `linkding-data` PVC to `etc/linkding/data`.
- **[`service.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/linkding/service.yaml)**: Internal `ClusterIP` service routing traffic on port `9090` to matching `app: linkding` pods.
- **[`storage.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/linkding/storage.yaml)**: PersistentVolumeClaim `linkding-data-pvc` requesting `1Gi` of `ReadWriteOnce` storage from the default storage class (`local-path-provisioner`).
- **[`kustomization.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/linkding/kustomization.yaml)**: Groups all resources together for Kustomize composition.

---

## 🚀 Access & Initial Setup

### 1. Port Forwarding
Because the service is currently configured as `ClusterIP`, port-forward the service to your workstation:

```bash
kubectl port-forward -n linkding svc/linkding 9090:9090
```

Navigate to **[http://localhost:9090](http://localhost:9090)** in your browser.

### 2. Creating the Initial Superuser Account
Linkding does not have a default username and password. You must generate an administrator account via the Django management CLI:

```bash
kubectl exec -it -n linkding deploy/linkding -- python manage.py createsuperuser
```

Follow the prompts to enter:
1. **Username** (e.g., `admin`)
2. **Email** (optional, press Enter to skip)
3. **Password**

After creating the superuser, log in at `http://localhost:9090` with your credentials.

---

## 🧩 Browser Extension & Integrations

Linkding has official and community browser extensions for Chrome and Firefox, as well as bookmarklets:
1. Log in to Linkding.
2. Go to **Settings > Integrations**.
3. Generate an API token and install the Linkding extension for one-click bookmarking from your browser.

---

## ⚠️ High Availability & Database Note

> [!NOTE]
> Linkding stores its data in an embedded **SQLite** database inside the persistent volume at `/etc/linkding/data/db.sqlite3`.
> 
> Because SQLite relies on local filesystem locks and the PVC is `ReadWriteOnce`, the deployment is configured with **`replicas: 1`** to ensure data integrity and prevent SQLite database write lock contention.
> 
> If horizontal scaling or high availability is required in the future, configure Linkding to use an external PostgreSQL database via environment variables (`LD_DB_ENGINE: postgres`, `LD_DB_HOST`, etc.).

---

## 🛠 Useful Commands

```bash
# Check pod status and logs
kubectl get pods -n linkding
kubectl logs -n linkding -l app=linkding -f

# Check storage binding
kubectl get pvc -n linkding

# Restart Deployment
kubectl rollout restart deployment linkding -n linkding
```
