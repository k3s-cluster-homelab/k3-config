# k3-config

Deklaratives GitOps-Konfigurationsrepository (Kustomize + ArgoCD) für den Single-Node K3s-Cluster.

## Struktur

```text
k3-config/
├── base/
│   ├── dashboard/           # Deployment Control UI
│   ├── frontend/            # Hauswasserwerk WebUI (Nginx)
│   ├── mosquitto/           # Eclipse Mosquitto (TCP 1883 + WS 9001)
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                 # DEV-Patches  (main -> Namespace: dev)
│   └── prod/                # PROD-Patches (main -> Namespace: prod)
├── application.yaml         # ArgoCD App: hauswerk-dev
└── application-prod.yaml    # ArgoCD App: hauswerk-prod
```

## Umgebungen & Ports

| Umgebung | ArgoCD App | Branch | Overlay | Namespace | Frontend | Mosquitto TCP / WS | Dashboard |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PROD** | `hauswerk-prod` | `main` | `overlays/prod` | `prod` | `30080` | `31884` / `32002` | `30090` |
| **DEV** | `hauswerk-dev` | `main` | `overlays/dev` | `dev` | `30081` | `31885` / `32003` | `30091` |

Image-Tags in `overlays/` werden ausschließlich über [`deployments`](../deployments) per Commit aktualisiert; ArgoCD synchronisiert anschließend automatisch (`prune` + `selfHeal`). Das ESP32-OTA-Flashing läuft on-demand direkt im Runner über `deployments/.github/workflows/deploy.yml`.
