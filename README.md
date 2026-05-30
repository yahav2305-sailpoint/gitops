# gitops

Production Helm values for **Linker**, consumed by Argo CD.

This repo acts as the single source of truth for the cluster's desired state. Argo CD watches it alongside the [app-helm](https://github.com/yahav2305-sailpoint/app-helm) chart repository; any change merged to `main` here is automatically applied to the cluster.

## How it works

The Argo CD `Application` manifest (in [app-helm/chart-testing/app.yaml](https://github.com/yahav2305-sailpoint/app-helm/blob/main/chart-testing/app.yaml)) references two sources:

1. **The Helm chart** — pulled from the `app-helm` chart repository (GitHub Pages OCI).
1. **This repo** — `values.yaml` is passed to the chart as a values overlay.

Argo CD merges the two and reconciles the cluster state on every sync.

## values.yaml highlights

The production overlay hardens the deployment beyond the chart defaults:

- **3 replicas** for availability.
- **Read-only root filesystem** — prevents writes to the container filesystem at runtime.
- **Non-root, no privilege escalation** — runs as UID 65532, all capabilities dropped.
- **Resource limits** set to prevent noisy-neighbour issues (`50m` CPU / `128Mi` memory).
- **Service account creation disabled** — the app does not need Kubernetes API access.

## Making a change

1. Edit `values.yaml` on a branch.
2. Open a pull request and merge to `main`.
3. Argo CD detects the change and syncs within its polling interval (default: 3 minutes). You can also trigger an immediate sync from the Argo CD UI or CLI:

    ```sh
    argocd app sync app-helm
    ```
