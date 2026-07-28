# Gym Platform — GitOps Deployment with ArgoCD

This repo demonstrates a GitOps deployment workflow for two microservices from the gym platform project, using ArgoCD on Kubernetes.

## Overview

Two services are deployed as separate ArgoCD Applications, each pointing to its own branch in this repository, to demonstrate two different sync strategies:

| Service | Branch | Sync Policy |
|---|---|---|
| profile-service | `gym-profile-service` | Automated (auto-sync + self-heal + prune) |
| progress-service | `gym-progress-service` | Manual |

The `main` branch holds documentation and the ArgoCD `Application` manifests (`argocd-apps/`).

## Repo Structure

```
├── argocd-apps/
│   ├── profile-service-app.yaml
│   └── progress-service-app.yaml
├── service-a-branch/       (profile-service manifests)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── service-b-branch/       (progress-service manifests)
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

## Why Two Sync Strategies

- **Auto-sync (profile-service):** simulates a stable, well-tested service where changes merged to the branch should roll out automatically. `selfHeal` reverts any manual drift in the cluster; `prune` removes resources no longer defined in git.
- **Manual sync (progress-service):** simulates a service where changes require a human check before rolling out to the cluster (e.g. schema-sensitive changes), so ArgoCD shows it as `OutOfSync` until someone runs `argocd app sync`.

## Installing ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml --server-side
kubectl get pods -n argocd -w
```

Get the initial admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Access the UI:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Deploying the Applications

```bash
kubectl apply -f argocd-apps/profile-service-app.yaml
kubectl apply -f argocd-apps/progress-service-app.yaml
```

profile-service will sync automatically. To sync progress-service manually:
```bash
argocd app sync progress-service
```

## Verifying

```bash
argocd app list
argocd app get profile-service
argocd app get progress-service
```

## Screenshots

_ArgoCD UI showing both services deployed and healthy:_
<img width="1318" height="553" alt="image" src="https://github.com/user-attachments/assets/68d35e9e-90a4-4925-b3f3-90b238c65a09" />
