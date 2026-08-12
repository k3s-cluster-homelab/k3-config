# k3-config

Kubernetes GitOps-Repository für das Heimserver-Lab (k3s, 4 GB RAM Limit).
Verwendet **Kustomize** (Bases & Overlays) und wird über **ArgoCD** synchronisiert.

## Struktur

```
k3-config/
├── base/                        # Gemeinsame Basis-Manifeste
│   ├── mosquitto/               # Eclipse Mosquitto MQTT Broker
│   ├── frontend/                # Frontend-Platzhalter
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                     # Dev-Overlay  → Branch: develop
│   └── prod/                    # Prod-Overlay → Branch: main
├── application.yaml             # ArgoCD App: dev  (develop Branch)
├── application-prod.yaml        # ArgoCD App: prod (main Branch)
└── README.md
```

## Branch-Strategie

| Environment | ArgoCD Application   | Branch (`targetRevision`) | Overlay         | Namespace |
|-------------|----------------------|---------------------------|-----------------|-----------|
| **Dev**     | `devops-stack-dev`   | `develop`                 | `overlays/dev`  | `dev`     |
| **Prod**    | `devops-stack-prod`  | `main`                    | `overlays/prod` | `prod`    |

**Workflow:**
1. Feature-Branches → Merge nach `develop` → Dev-Umgebung synct automatisch
2. `develop` → Merge nach `main` → Prod-Umgebung synct automatisch

## Unterschiede zwischen Overlays

| Eigenschaft          | Dev                              | Prod                             |
|----------------------|----------------------------------|----------------------------------|
| Image-Tag            | `latest`                         | `stable`                         |
| Frontend Replicas    | 1                                | 2                                |
| Mosquitto Mem Limit  | 64Mi (Base)                      | 128Mi (gepatcht)                 |
| Namespace            | `dev`                            | `prod`                           |

## Voraussetzungen

- k3s-Cluster läuft und ist via `kubectl` erreichbar
- ArgoCD ist im Cluster installiert (`argocd` Namespace)
- Lokale Container-Registry auf `localhost:5010`
- Git-Repo hat Branches `main` und `develop`

## Deployment

1. **`repoURL` anpassen** – In `application.yaml` und `application-prod.yaml` die Platzhalter-URL durch die tatsächliche Git-Remote-URL ersetzen.

2. **Beide Environments deployen:**

   ```bash
   kubectl apply -f application.yaml
   kubectl apply -f application-prod.yaml
   ```

3. **Manuell testen (ohne ArgoCD):**

   ```bash
   # Dev
   kubectl apply -k overlays/dev

   # Prod
   kubectl apply -k overlays/prod
   ```

## Services

| Service    | Typ       | Port(s)                         |
|------------|-----------|---------------------------------|
| Mosquitto  | NodePort  | 1883 → 31883, 9001 → 39001     |
| Frontend   | ClusterIP | 80                              |
