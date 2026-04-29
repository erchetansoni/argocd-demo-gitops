# environments/

One folder per branch in the source repo. Each folder is the *source of truth* for what gets deployed to namespace `<branch>` in the cluster.

## Owned by CI — do not edit

These folders are owned by [`.github/workflows/deploy.yml`](https://github.com/erchetansoni/argocd-demo/blob/main/.github/workflows/deploy.yml) in the source repo. The workflow wipes a folder before re-rendering on every push, so any hand-edits will be lost on the next sync.

## Per-env structure

```
environments/<branch>/
├── app1/
│   ├── kustomization.yaml      # helmGlobals.chartHome -> ../../../_source/apps
│   └── values.yaml             # per-env Helm values for app1
├── app2/
│   └── kustomization.yaml      # resources -> ../../../_source/apps/app2 + ingress patch
└── app3/
    └── kustomization.yaml      # resources -> ../../../_source/apps/app3 + ingress patch
```

## How they map to ArgoCD Applications

The [matrix ApplicationSet](../applicationset.yaml) generates one Application per (env × app):

| Folder | Application | Namespace |
|---|---|---|
| `environments/main/app1` | `main-app1` | `main` |
| `environments/main/app2` | `main-app2` | `main` |
| `environments/main/app3` | `main-app3` | `main` |
| `environments/qa/app1` | `qa-app1` | `qa` |
| `environments/qa/app2` | `qa-app2` | `qa` |
| `environments/qa/app3` | `qa-app3` | `qa` |

## Lifecycle

| Source-repo event | Effect here |
|---|---|
| `git push origin <branch>` | `environments/<branch>/` written/refreshed by CI |
| `git push --delete origin <branch>` | `environments/<branch>/` removed by CI ([cleanup.yml](https://github.com/erchetansoni/argocd-demo/blob/main/.github/workflows/cleanup.yml)) |

After either event, ArgoCD's ApplicationSet polls this repo and creates/deletes Applications to match.