# meditriage-gitops

Repositorio GitOps de MediTriage. Contiene **solo manifiestos** (Kustomize); el código y el
Dockerfile viven en el repo app (`ginalef-ui/MediTriage`). Argo CD observa este repo y reconcilia el cluster.

```
base/                 manifiestos comunes (Deployment, Service, ConfigMap, Ingress, HPA)
overlays/
  dev/                namespace meditriage-dev,  1-2 pods,  host dev.api.meditriage.cl
  prod/               namespace meditriage-prod, 3-5 pods,  host api.meditriage.cl
```

## Diferencias entre ambientes

| | dev | prod |
|---|---|---|
| Namespace | `meditriage-dev` | `meditriage-prod` |
| Pods (HPA min-max) | 1 - 2 | 3 - 5 |
| Recursos (req / lim) | 100m,128Mi / 500m,256Mi | 200m,256Mi / 1000m,512Mi |
| `NODE_ENV` | `development` | `production` |
| Host Ingress | `dev.api.meditriage.cl` | `api.meditriage.cl` |
| Secret | `secret.yaml` con valores de ejemplo | fuera de git (Sealed Secrets / ESO) |

## Probar localmente

```bash
kubectl kustomize overlays/dev
kubectl kustomize overlays/prod
kubectl apply -k overlays/dev
```

## Flujo de despliegue

1. El CI del repo app construye y publica `ghcr.io/ginalef-ui/meditriage-backend:<git-sha>`.
2. El job `promote` abre un **PR a este repo** cambiando la imagen del overlay:
   ```bash
   cd overlays/dev
   kustomize edit set image ghcr.io/ginalef-ui/meditriage-backend=ghcr.io/ginalef-ui/meditriage-backend:<git-sha>
   ```
3. Se revisa y se hace merge del PR. Argo CD detecta el commit y sincroniza.
4. Para prod se repite el PR sobre `overlays/prod` (2 revisores, sin push directo a `main`).
5. **Rollback** = `git revert` del commit de promoción.

## Apuntar Argo CD

Una `Application` por ambiente, con `path: overlays/dev` y `path: overlays/prod`,
`repoURL` de este repo y `syncPolicy.automated` (`prune` y `selfHeal`) en dev.
