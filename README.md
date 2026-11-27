# Todo GitOps

Argo CD syncs local k3d from this repository’s `main` branch. Application code and the [promotion workflow](https://github.com/ehye/KubernetesSubmissions/blob/main/.github/workflows/todo-environments.yaml) live in `KubernetesSubmissions`.

| Namespace | Desired state | Image source |
|---|---|---|
| `staging` | [Staging overlay](overlays/staging) | Build from source `main` |
| `production` | [Production overlay](overlays/production) | Tagged commit’s staging digests |
| `project` | [Local exercise](todo_app/overlays/k3d) | Images imported into k3d |

## CI/CD flow

```mermaid
sequenceDiagram
    participant Src as KubernetesSubmissions
    participant CI as GitHub Actions
    participant Reg as GHCR
    participant Git as KubernetesSubmissions-gitops (main)
    participant CD as Argo CD
    participant K as Local k3d

    Src->>CI: Push commit to main
    CI->>Reg: Build and push frontend, backend, broadcaster
    Reg-->>CI: Three image digests
    CI->>Git: Record digests for commit SHA and update staging overlay
    CD->>Git: Read staging overlay
    CD->>K: Sync staging namespace

    Src->>CI: Push release tag
    CI->>Git: Look up build record for tagged commit SHA
    alt Staged build exists
        Git-->>CI: Exact image digests, no rebuild
        CI->>Git: Update production overlay
        CD->>Git: Read production overlay
        CD->>K: Sync production namespace
    else No staged build
        CI-->>Src: Fail promotion without changing production
    end
```

- **[Build records](builds)** answer “Which images did we build from this commit?” Each `<source-commit>.json` lists the exact GHCR digests for the frontend, backend, and broadcaster. When a release is tagged, production reuses those images instead of rebuilding them.
- **[Current promotions](promotions)** answer “Which source commit did CI select for each environment?” `staging.json` and `production.json` each record the commit and GitHub Actions run number. These describe the intended release; check Argo CD for what is actually synced and healthy.

Provision Secrets outside Git, then register the [Argo CD Applications](argocd):

```sh
kubectl --context=k3d-k3d-test apply -f argocd/
kubectl --context=k3d-k3d-test -n argocd get applications
```
