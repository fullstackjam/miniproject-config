# GitOps Configuration

This repository contains the Kubernetes manifests and GitOps configuration for the Demo Application.

## Overview

It is designed to be monitored by **ArgoCD** to automatically sync the application state to the Kubernetes cluster.

- **Pattern**: App Code + Config Repo
- **Tooling**: ArgoCD, Argo Rollouts, Istio

## Repository Structure

```
miniproject-config/
└── base/
    ├── namespace.yaml          # Namespace definition
    ├── service-stable.yaml     # Stable Service (v1)
    ├── service-canary.yaml     # Canary Service (v2)
    ├── destinationrule.yaml    # Istio DestinationRule
    ├── virtualservice.yaml     # Istio VirtualService
    └── rollout-v2.yaml         # Argo Rollout definition
```

## Deployment Workflow

1. **CI Trigger**: When code is pushed to `miniproject`, the CI pipeline builds a new image.
2. **Config Update**: The CI pipeline automatically commits the new image tag to `base/rollout-v2.yaml` in this repository.
3. **Sync**: ArgoCD detects the change and syncs the manifests to the cluster.
4. **Rollout**: Argo Rollouts executes the progressive delivery strategy (Canary).

## Manual Management

To manually trigger a deployment (e.g., rollback or testing):

```bash
# Edit the image tag
vim base/rollout-v2.yaml

# Commit and push
git commit -am "Update image to v2"
git push
```

## Verification

To verify the complete flow:

1. **Make a code change** in `miniproject` and push.
2. **Watch CI**: Check GitHub Actions in `miniproject` repo.
3. **Check Config**: Verify `miniproject-config` has a new commit with the updated image tag.
4. **Watch Rollout**:
   ```bash
   kubectl argo rollouts get rollout demo -n demo --watch
   ```

## Troubleshooting

### CI fails with "Could not update config repo"
- Check `GH_PAT` secret in `miniproject` repo settings.
- Ensure PAT has `repo` scope.

### ArgoCD shows "OutOfSync"
- Verify ArgoCD can access this public repository.
- Try manual sync: `kubectl patch application demo-app -n argocd --type merge -p '{"operation":{"sync":{}}}'`

### Rollout not progressing
- Check rollout status: `kubectl describe rollout demo -n demo`
- Check pods: `kubectl get pods -n demo`
