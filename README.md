# k3-config

Kubernetes GitOps-Repository für das Heimserver-Lab (k3s, 4 GB RAM Limit).
Verwendet **Kustomize** (Bases & Overlays) und wird über **ArgoCD** synchronisiert.

## Struktur

```
k3-config/
├── base/                        # Gemeinsame Basis-Manifeste
│   ├── mosquitto/               # Eclipse Mosquitto MQTT Broker
│   ├── frontend/                # Frontend WebUI (Port 80 -> NodePort)
│   ├── dashboard/               # Deployment Dashboard WebUI
│   ├── flasher/                 # ESP32 OTA Flasher Job (On-Demand / Deployment)
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                     # Dev-Overlay  → Branch: develop / Namespace: dev
│   └── prod/                    # Prod-Overlay → Branch: main    / Namespace: prod
├── application.yaml             # ArgoCD App: dev  (develop Branch)
├── application-prod.yaml        # ArgoCD App: prod (main Branch)
└── README.md
```

## Branch-Strategie & GitOps Workflow

| Environment | ArgoCD Application   | Branch (`targetRevision`) | Overlay         | Namespace | ESP32 Target IP |
|-------------|----------------------|---------------------------|-----------------|-----------|-----------------|
| **Dev**     | `devops-stack-dev`   | `develop`                 | `overlays/dev`  | `dev`     | `192.168.177.50`|
| **Prod**    | `devops-stack-prod`  | `main`                    | `overlays/prod` | `prod`    | `192.168.177.28`|

**Workflow:**
1. **Frontend / MQTT / Firmware:** Images werden in CI via Multi-Stage Docker gebaut und nach Zot (`192.168.178.183:5010`) gepusht.
2. **Dev-Deployment:** CI triggert das zentrale `deployments`-Repo $\rightarrow$ `overlays/dev` wird aktualisiert $\rightarrow$ ArgoCD synct automatisch (flasht Dev-ESP32 `192.168.177.50`).
3. **Prod-Deployment:** PR/Merge nach `main` in `k3-config` $\rightarrow$ ArgoCD synct Prod-Dienste (flasht Prod-ESP32 `192.168.177.28`).

## Services & Workloads

| Service / Workload       | Typ                 | Port(s) / Funktion                                |
|--------------------------|---------------------|---------------------------------------------------|
| Mosquitto                | NodePort            | 1883 → 31884/31885, 9001 → 32001/32003 (WS)      |
| Frontend                 | ClusterIP / NodePort| 80 → 30080/30081 (WebUI Dashboard)                |
| Deployment Dashboard     | ClusterIP / NodePort| 80 → 30090/30091 (Deployment Control UI)          |
| esp32-firmware-flasher   | Job (On-Demand)     | Flasht ESP32 Firmware über lokales LAN            |
