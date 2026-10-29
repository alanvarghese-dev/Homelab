# 🍲 Mealie Application

[Mealie](https://mealie.io/) is a self-hosted recipe manager and meal planner with a clean REST API and a modern web interface. It allows you to scrape recipes automatically from across the web, plan weekly meals, and generate shopping lists.

---

## 📋 Overview & Specification

| Property | Value |
| :--- | :--- |
| **Namespace** | `mealie` |
| **Container Image** | `ghcr.io/mealie-recipes/mealie:v1.3.0` |
| **Internal Port** | `9000` |
| **Service Type** | `ClusterIP` |
| **Current Replicas** | `2` |
| **Persistent Storage** | `500Mi` (`mealie-data-pvc`) |
| **Data Mount Path** | `/app/data` |
| **Run As User/Group** | `33:33` (`www-data`) |
| **Privilege Escalation**| `false` |

---

## 📁 Manifests Breakdown

- **[`namespace.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/mealie/namespace.yaml)**: Creates the isolated `mealie` namespace.
- **[`deployment.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/mealie/deployment.yaml)**: Defines the `mealie` Deployment:
  - Image: `ghcr.io/mealie-recipes/mealie:v1.3.0`.
  - Exposes container port `9000`.
  - Enforces non-root security context (`runAsUser: 33`, `runAsGroup: 33`, `fsGroup: 33` for `www-data`).
  - Blocks privilege escalation (`allowPrivilegeEscalation: false`).
  - Mounts `mealie-data-pvc` to `/app/data`.
- **[`service.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/mealie/service.yaml)**: Internal `ClusterIP` service routing port `9000` to `app: mealie`.
- **[`storage.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/mealie/storage.yaml)**: PersistentVolumeClaim `mealie-data-pvc` requesting `500Mi` of `ReadWriteOnce` storage via `local-path-provisioner`.
- **[`kustomization.yaml`](file:///Users/iti/Documents/My%20Mac%20Docu/Homelab/apps/base/mealie/kustomization.yaml)**: Aggregates the base manifests into a reusable Kustomize package.

---

## 🚀 Access & Initial Setup

### 1. Port Forwarding
Port-forward the `mealie` service to your local machine:

```bash
kubectl port-forward -n mealie svc/mealie 9000:9000
```

Open **[http://localhost:9000](http://localhost:9000)** in your browser.

### 2. Default Login Credentials
Mealie provides a default administrator account upon initialization:

- **Email**: `changeme@example.com`
- **Password**: `MyPassword`

> [!CAUTION]
> **Change Credentials Immediately**: Navigate to **User Profile / Settings** immediately after logging in to change your email and set a secure password!

---

## ✨ Key Features & Capabilities

1. **Automatic Recipe Scraping**: Paste any recipe URL (e.g., from NYT Cooking, BBC Good Food, Allrecipes), and Mealie extracts ingredients, instructions, and imagery using Schema.org metadata.
2. **Interactive Meal Planner**: Plan breakfast, lunch, and dinner on an interactive weekly calendar.
3. **Smart Shopping Lists**: Add recipe ingredients directly to categorized grocery lists that sync across devices.
4. **Browser Bookmarklet**: Use the Mealie bookmarklet to import recipes with a single click.

---

## ⚠️ High Availability & Database Note

> [!IMPORTANT]
> By default, Mealie operates with an embedded **SQLite** database stored in `/app/data/mealie.db`.
> 
> Because SQLite does not support concurrent write operations across multiple processes and the PVC uses `ReadWriteOnce`:
> - Running 2 pods concurrently against the same SQLite database can lead to write locks or sync errors.
> - **Recommendation**: For standard homelab use with SQLite, maintain `replicas: 1`. To run multiple replicas, configure an external PostgreSQL database via environment variables (`DB_ENGINE: postgres`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_SERVER`, `POSTGRES_PORT`, `POSTGRES_DB`).

---

## 🛠 Useful Commands

```bash
# Check pod status and logs
kubectl get pods -n mealie
kubectl logs -n mealie -l app=mealie -f

# Check storage binding
kubectl get pvc -n mealie

# Restart Deployment
kubectl rollout restart deployment mealie -n mealie
```
