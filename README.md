
# 🚀 ArgoCD GitOps with ApplicationSet

## 🌐 Overview

This repository manages Kubernetes applications using **ArgoCD** with **ApplicationSet** and **Kustomize**. It supports multiple environments (`dev`, `staging`, `production`, `test`, `main`) and dynamically discovers applications via Git paths.

---

## 📁 Folder Structure

```bash
.
├── apps.yaml          # ApplicationSet for all apps by environment
├── root.yaml          # Bootstrap ApplicationSet (e.g., base apps)
└── kustomization.yaml # Kustomize entry point
```

---

## 🔧 Prerequisites

✅ You must have:

- A Kubernetes cluster (`minikube`, `kind`, `EKS`, etc.)
- `kubectl` installed and configured
- Your Git repo ready (like `https://github.com/huygiale-fi/test-argocd.git`)
- [Optional] A domain for ArgoCD ingress

---

## 📥 Step 1: Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
```

---

## 🔐 Step 2: Access ArgoCD UI

### Option 1: Port-forward (Local testing)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open: https://localhost:8080
```

### Option 2: Ingress (Production)

Create an Ingress resource using NGINX or another controller (optional).

---

## 🔑 Step 3: Login to ArgoCD

Default user: `admin`

Get password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

Login via CLI:

```bash
argocd login localhost:8080 --username admin --password <the-password> --insecure
```

---

## 📦 Step 4: Structure Your Git Repo

Your repo should look like:

```bash
test-argocd/
├── apps/
│   ├── app-name/
│      └── dev/
│          └── frontend/
│              └── config.json
│          └── backend/
│              └── config.json
├── bootstrap/
│   └── applications/
│       ├── root.yaml/
│       ├── apps.yaml
│       ├── kustomization.yaml
│   └── somethings/
│       └── base/
│           └── config.json
```

---

## ✨ Step 5: Understand the Files

### ✅ root.yaml

Bootstraps the initial set of core applications from the `bootstrap/` directory.

### ✅ apps.yaml

Creates dynamic apps per environment (dev, staging, production, test, main) using `ApplicationSet`.

Each `config.json` might look like:

```json
{
  "appName": "my-app-environment",
  "destNamespace": "my-app-environment"
}
```

---

## 🚀 Step 6: Deploy ArgoCD Applications

Apply using `kustomize`:

```bash
cd bootstrap/applications
kubectl apply -k .
```

This reads:

- `kustomization.yaml`
- `root.yaml`
- `apps.yaml`

---

## 🔄 Step 7: Automate Sync & Create Namespaces

Your `ApplicationSet` templates include:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

---

## 👀 Step 8: Watch in ArgoCD UI

Open your ArgoCD dashboard. You will see:

- `bootstrap` application
- Environment-specific apps (`dev`, `prod`, etc.)
- All managed from Git

---

## ✅ Step 9: Add New Apps

Push a new `config.json` into the appropriate path:

```json
apps/<project-name>/dev/frontend/config.json
apps/<project-name>/dev/backend/config.json

**Example**
apps/blogger/dev/blogger-fe/config.json
apps/blogger/dev/blogger-be/config.json
```

ArgoCD detects and creates the app automatically.

---

## 🔐 Step 10: Secure Your Setup

- Change admin password
- Enable RBAC
- [Optional] Enable HTTPS (e.g., with cert-manager)
- Use secret management (Sealed Secrets, SOPS, etc.)

---

## 📚 References

- [ArgoCD Official Docs](https://argo-cd.readthedocs.io/)
- [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/)
