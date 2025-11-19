# Setup Instructions

## Prerequisites

This repository has been initialized locally. Follow these steps to complete the setup.

## Step 1: Create GitHub Repository

1. Go to https://github.com/new
2. Repository name: `miniproject-config`
3. Description: `GitOps configuration for demo application`
4. Visibility: **Public**
5. **Do NOT** initialize with README, .gitignore, or license
6. Click "Create repository"

## Step 2: Push to GitHub

The local repository is ready. Push it:

```bash
cd /Users/fullstackjam/workspace/miniproject-config
git push -u origin main
```

## Step 3: Create Personal Access Token (PAT)

The CI workflow needs a token to update this config repository.

1. Go to https://github.com/settings/tokens/new
2. Note: `GitHub Actions - miniproject CI`
3. Expiration: Choose your preference (recommend 90 days)
4. Scopes: Select **`repo`** (Full control of private repositories)
5. Click "Generate token"
6. **Copy the token** (you won't see it again!)

## Step 4: Add Secret to App Repository

1. Go to https://github.com/fullstackjam/miniproject/settings/secrets/actions
2. Click "New repository secret"
3. Name: `GH_PAT`
4. Value: Paste the PAT from Step 3
5. Click "Add secret"

## Step 5: Update ArgoCD

Apply the updated ArgoCD application:

```bash
kubectl apply -f /Users/fullstackjam/workspace/miniproject/argocd/application.yaml
```

Wait for ArgoCD to sync:

```bash
# Watch sync status
watch kubectl get application demo-app -n argocd
```

## Verification

### Test the complete flow:

1. **Make a code change** in miniproject:
   ```bash
   cd /Users/fullstackjam/workspace/miniproject
   echo "# Test change" >> README.md
   git add README.md
   git commit -m "Test GitOps flow"
   git push
   ```

2. **Watch CI build and push**:
   - Go to https://github.com/fullstackjam/miniproject/actions
   - CI should build image and update config repo

3. **Check config repo updated**:
   - Go to https://github.com/fullstackjam/miniproject-config
   - Should see new commit from github-actions[bot]
   - `base/rollout-v2.yaml` should have new image SHA

4. **Watch ArgoCD sync**:
   ```bash
   kubectl get application demo-app -n argocd --watch
   ```

5. **Watch canary deployment**:
   ```bash
   kubectl argo rollouts get rollout demo -n demo --watch
   ```

6. **Monitor traffic shift**:
   ```bash
   watch 'kubectl get virtualservice demo-vs -n demo -o jsonpath="{.spec.http[1].route}" | jq .'
   ```

## Architecture

```
Developer Push
     ↓
miniproject (App Code)
     ↓
GitHub Actions CI
     ↓ (builds image)
ghcr.io/fullstackjam/demo:<SHA>
     ↓ (updates manifest)
miniproject-config (This Repo)
     ↓ (detects change)
ArgoCD
     ↓ (applies)
Kubernetes + Argo Rollouts
     ↓ (progressive canary)
Production Traffic
```

## Troubleshooting

### CI fails with "Could not update config repo"
- Check GH_PAT secret is set correctly
- Ensure PAT has `repo` scope
- Verify config repo exists and is accessible

### ArgoCD shows "OutOfSync"
- Check repo URL in Application manifest
- Verify config repo is public or ArgoCD has access
- Try manual sync: `kubectl patch application demo-app -n argocd --type merge -p '{"operation":{"sync":{}}}'`

### Rollout not progressing
- Check rollout status: `kubectl describe rollout demo -n demo`
- Check pods: `kubectl get pods -n demo`
- Check events: `kubectl get events -n demo --sort-by='.lastTimestamp'`
