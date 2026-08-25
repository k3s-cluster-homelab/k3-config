# k3-config

Kubernetes GitOps-Repository für das Heimserver-Lab (k3s, 4 GB RAM Limit).
Verwendet **Kustomize** (Bases & Overlays) und wird über **ArgoCD** synchronisiert.

## Struktur

```
k3-config/
├── base/                        # Gemeinsame Basis-Manifeste
│   ├── mosquitto/               # Eclipse Mosquitto MQTT Broker
│   ├── frontend/                # Frontend WebUI
│   ├── flasher/                 # ESP32 OTA Flasher Job (ArgoCD Sync Hook)
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                     # Dev-Overlay  → Branch: develop
│   └── prod/                    # Prod-Overlay → Branch: main
├── application.yaml             # ArgoCD App: dev  (develop Branch)
├── application-prod.yaml        # ArgoCD App: prod (main Branch)
└── README.md
```

## Branch-Strategie & GitOps Workflow

| Environment | ArgoCD Application   | Branch (`targetRevision`) | Overlay         | Namespace |
|-------------|----------------------|---------------------------|-----------------|-----------|
| **Dev**     | `devops-stack-dev`   | `develop`                 | `overlays/dev`  | `dev`     |
| **Prod**    | `devops-stack-prod`  | `main`                    | `overlays/prod` | `prod`    |

**Workflow:**
1. **Frontend / MQTT / Firmware:** Images werden in CI gebaut und nach Zot (`192.168.178.183:5010`) gepusht.
2. **Dev-Deployment:** Image-Tag in `overlays/dev` anpassen $\rightarrow$ Push/Merge nach `develop` $\rightarrow$ ArgoCD synct automatisch (flasht Dev-ESP32).
3. **Prod-Deployment:** Merge nach `main` $\rightarrow$ ArgoCD synct automatisch (flasht Prod-ESP32).

## Services & Workloads

| Service / Workload       | Typ                 | Port(s) / Funktion              |
|--------------------------|---------------------|---------------------------------|
| Mosquitto                | NodePort            | 1883 → 31883, 9001 → 39001      |
| Frontend                 | ClusterIP           | 80                              |
| esp32-firmware-flasher   | Job (ArgoCD Hook)   | Flasht ESP32 Firmware über LAN  |
