# ArgoCD GitOps Demo

This repository demonstrates GitOps with ArgoCD and CloudBees Unify orchestration.

## Architecture

**GitOps Pull Model:**
- CloudBees Unify updates desired state in Git (pushes to Git)
- ArgoCD pulls changes from Git and applies to cluster
- Unify triggers ArgoCD refresh to eliminate reconciliation delay
- Unify polls ArgoCD status and captures timing evidence

## Repository Structure

```
argocd-demo/
├── charts/
│   └── demo-app/          # Helm chart for simple nginx app
│       ├── Chart.yaml
│       ├── values.yaml    # Chart defaults
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── configmap.yaml  # Custom index.html
├── apps/
│   └── demo-app/          # Values stacking for GitOps
│       ├── values-base.yaml   # Stable defaults
│       ├── values-dev.yaml    # Environment-specific config
│       └── values-image.yaml  # Image tag (updated by Unify)
```

## Values Stacking Pattern

ArgoCD Application uses multiple `valueFiles` in order:
1. `apps/demo-app/values-base.yaml` - Base configuration
2. `apps/demo-app/values-dev.yaml` - Environment overrides
3. `apps/demo-app/values-image.yaml` - Image tag override (Unify updates this)

This separation allows:
- Stable config in base
- Environment-specific config per env
- **Image tag changes isolated to one file** (clean Git diffs)

## Demo Flow

1. **Initial State:**
   - App running with nginx:1.25.0
   - ArgoCD shows Synced status

2. **CloudBees Unify Workflow Triggers:**
   - Updates `apps/demo-app/values-image.yaml` to nginx:1.25.1
   - Commits to branch and opens PR
   - Waits for PR merge

3. **Post-Merge:**
   - Workflow triggers ArgoCD refresh (no reconciliation delay)
   - Polls ArgoCD until status = Synced
   - Captures timing evidence:
     - Git commit time
     - Refresh trigger time
     - Sync completed time
     - Total lag (commit → synced)

4. **Result:**
   - App now running nginx:1.25.1
   - Evidence shows GitOps pull model with Unify orchestration
   - **Key message:** Unify doesn't push to cluster, only updates Git

## ArgoCD Application Setup

The ArgoCD Application should be configured with:
- **Source:** This Git repository
- **Path:** `charts/demo-app`
- **Values Files:**
  - `apps/demo-app/values-base.yaml`
  - `apps/demo-app/values-dev.yaml`
  - `apps/demo-app/values-image.yaml`
- **Destination:** `argocd-demo` namespace
- **Auto-Sync:** Enabled (with prune + selfHeal optional)

## Accessing the Demo App

```bash
# Port-forward to access the app
kubectl port-forward -n argocd-demo svc/demo-app-dev 8080:80

# Open browser to http://localhost:8080
```

The web page shows:
- Current version
- Current image tag
- Deployment method (ArgoCD)
- Orchestrator (CloudBees Unify)

## CloudBees Unify Workflow

See `.cloudbees/workflows/gitops-update.yaml` for the workflow that:
1. Updates image tag in Git
2. Creates PR
3. Triggers ArgoCD refresh
4. Polls until Synced
5. Captures evidence

## Key Benefits Demonstrated

1. **GitOps Compliance:** All changes via Git (audit trail)
2. **Pull Model Preserved:** ArgoCD pulls, Unify doesn't push to cluster
3. **Zero Reconciliation Delay:** Refresh eliminates waiting
4. **Orchestration Evidence:** Full timing/status capture
5. **Clean Separation:** Image updates isolated from config

## Related Links

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [CloudBees Unify](https://docs.cloudbees.com/docs/cloudbees-platform/)
