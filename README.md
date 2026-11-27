# Todo desired state

Argo CD reads `main` of this repository. Application code, Dockerfiles and the release workflow live in [KubernetesSubmissions](https://github.com/ehye/KubernetesSubmissions).

| Application | Namespace | Kustomize path | Promotion |
|---|---|---|---|
| todo-app-staging | staging | overlays/staging | Source main build |
| todo-app-production | production | overlays/production | Source tag, using that commit’s recorded build |
| todo-app | project | todo_app/overlays/k3d | Existing local exercise, imported images |

Edit shared resources under `todo_app/base`, `todo_backend/manifests` and `broadcaster`, and environment configuration under `todo_app/overlays`. The workflow updates image digests in `overlays/<env>/kustomization.yaml`, keeps immutable staging build records under `builds/` and records the source commit in `promotions/`. A config-only commit does not rebuild images.

On k3d, install Argo CD, Argo Rollouts, Envoy Gateway (`local-envoy`), NATS, Prometheus and local-path storage. Provision `todo-postgres-secret` in each namespace; production also needs `broadcaster-discord` and `todo-backup-gcs-secret`. Configure registry pull access for private GHCR packages. Never commit credentials. Register each Application when its prerequisites are available:

```sh
kubectl --context=k3d-k3d-test apply -f argocd/todo-app.yaml
kubectl --context=k3d-k3d-test apply -f argocd/staging.yaml
kubectl --context=k3d-k3d-test apply -f argocd/production.yaml
kubectl --context=k3d-k3d-test -n argocd get applications
```
