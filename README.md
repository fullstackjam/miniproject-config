# miniproject-config

GitOps configuration repository for the demo application.

## Repository Structure

```
miniproject-config/
└── base/
    ├── namespace.yaml          # Demo namespace with Istio injection
    ├── service-stable.yaml     # Stable service
    ├── service-canary.yaml     # Canary service
    ├── destinationrule.yaml    # Istio subsets configuration
    ├── virtualservice.yaml     # Traffic routing rules
    └── rollout-v2.yaml         # Argo Rollout with canary strategy
```

## How It Works

This repository contains the Kubernetes manifests for the demo application.

### Automated Deployment Flow:

1. **Developer pushes code** to [miniproject](https://github.com/fullstackjam/miniproject)
2. **CI builds Docker image** tagged with commit SHA
3. **CI pushes image** to ghcr.io/fullstackjam/demo
4. **CI updates** `base/rollout-v2.yaml` in this repo with new image tag
5. **ArgoCD detects change** and syncs automatically
6. **Argo Rollouts deploys** with progressive canary (10% → 30% → 50% → 100%)
7. **Istio routes traffic** according to weights set by Argo Rollouts

### Manual Updates

To manually update the deployment:

```bash
# Update image tag
sed -i 's|image: ghcr.io/fullstackjam/demo:.*|image: ghcr.io/fullstackjam/demo:NEW_TAG|g' base/rollout-v2.yaml

# Commit and push
git add base/rollout-v2.yaml
git commit -m "Update to NEW_TAG"
git push
```

ArgoCD will automatically sync the change within 3 minutes.

## Canary Deployment Strategy

The rollout follows this progression:

- **Step 1**: 10% traffic to new version (pause 30s)
- **Step 2**: 30% traffic to new version (pause 30s)
- **Step 3**: 50% traffic to new version (pause 30s)
- **Step 4**: 100% traffic to new version (promotion complete)

## Header-Based Routing

Beta users can always access the latest canary version by adding a header:

```bash
curl -H "x-beta-user: 1" http://demo-service/
```

## ArgoCD Application

This repository is monitored by ArgoCD Application `demo-app` in the `argocd` namespace.

## Related Repositories

- **App Code**: [miniproject](https://github.com/fullstackjam/miniproject)
- **Config**: [miniproject-config](https://github.com/fullstackjam/miniproject-config) (this repo)
