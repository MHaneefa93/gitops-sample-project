## Install helm

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

## Install ArgoCD using Helm

```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd -n argocd
```
## Deploying with ArgoCD on Minikube

This guide explains how to connect this GitHub repository to ArgoCD running on Minikube, so that Kubernetes manifests are automatically deployed.

1. Access ArgoCD UI

Forward the ArgoCD service port:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open the ArgoCD UI at:

https://localhost:8080

2. Get the ArgoCD Admin Password

Run the following to fetch the initial admin password:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Login with:

Username: admin

Password: <output from above>

3. Connect GitHub Repository to ArgoCD
Option A: Using CLI

First, log in to ArgoCD:

```
argocd login localhost:8080 --username admin --password <PASSWORD> --insecure
```

Add the GitHub repository:

```
argocd repo add https://github.com/<your-username>/<your-repo>.git
```

Option B: Using UI

Go to Settings → Repositories in ArgoCD UI.

Click Connect Repo.

Enter your GitHub repository URL.

If private, provide authentication (SSH key or PAT).

4. Create an Application in ArgoCD

Create a new ArgoCD application pointing to your repo:

```
argocd app create myapp \
  --repo https://github.com/<your-username>/<your-repo>.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

--path: directory in the repo containing Kubernetes manifests (e.g., k8s/, manifests/, or .).

--dest-server: Kubernetes API server (for Minikube, use https://kubernetes.default.svc).

--dest-namespace: namespace where the app will be deployed.

5. Sync the Application

Deploy manifests by syncing:

argocd app sync myapp


✅ Now your application is continuously deployed from GitHub to your Minikube cluster by ArgoCD.
