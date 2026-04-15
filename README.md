# ArgoCD Practical Example — Todo App

A simple Node.js todo app deployed to Kubernetes via ArgoCD GitOps.

## Project Structure

```
todo-app/        # Node.js app source + Dockerfile
app/             # Kubernetes manifests (ArgoCD watches this)
argocd/          # ArgoCD Application resource
```

---

## Step 1 — Build & Push the Docker Image

```bash
cd todo-app
docker build -t your-dockerhub-username/todo-app:latest .
docker push your-dockerhub-username/todo-app:latest
```

Then update `app/deployment.yaml` with your actual image name.

---

## Step 2 — Start a Local Cluster

Using minikube:
```bash
minikube start
```

Or kind:
```bash
kind create cluster
```

---

## Step 3 — Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=available deployment/argocd-server -n argocd --timeout=120s
```

---

## Step 4 — Access the ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get the admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

Open https://localhost:8080 — login with `admin` + the password above.

---

## Step 5 — Deploy the App via ArgoCD

```bash
kubectl apply -f argocd/application.yaml
```

ArgoCD will detect the `app/` folder in this repo and sync the deployment, service, and ingress to your cluster.

---

## Step 6 — Access the Todo App

```bash
kubectl port-forward svc/todo-app 3000:80
```

Open http://localhost:3000 in your browser.

---

## Step 7 — Try the GitOps Loop

1. Edit `app/deployment.yaml` (e.g. change `replicas: 2` to `replicas: 3`)
2. Commit and push to `main`
3. Watch ArgoCD auto-sync within ~3 minutes (or click Sync in the UI)

---

## Key Concepts Covered

- GitOps: Git is the single source of truth
- ArgoCD `Application` CRD
- Automated sync with `selfHeal` and `prune`
- Containerising a Node.js app with Docker
- Kubernetes Deployment, Service, and Ingress
