# k3-config

Kubernetes GitOps-Repository für das Heimserver-Lab (k3s, Single-Node-Cluster).
Verwendet **Kustomize** (Bases & Overlays) und wird über **ArgoCD** synchronisiert.

## Struktur

```text
k3-config/
├── base/                        # Gemeinsame Basis-Manifeste
│   ├── mosquitto/               # Eclipse Mosquitto MQTT Broker
│   ├── frontend/                # Frontend WebUI (Port 80 -> NodePort)
│   ├── dashboard/               # Deployment Dashboard WebUI
│   ├── flasher/                 # Referenz-Manifest für ESP32 OTA Flasher Job
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                     # Dev-Overlay  → Branch: main / Namespace: dev
│   └── prod/                    # Prod-Overlay → Branch: main / Namespace: prod
├── application.yaml             # ArgoCD App: hauswerk-dev  (main -> overlays/dev)
├── application-prod.yaml        # ArgoCD App: hauswerk-prod (main -> overlays/prod)
└── README.md
```

## Branch-Strategie & GitOps Workflow

Das Repository nutzt eine **Single-Branch-Architektur (`main`)** mit Directory-Separation (`overlays/dev` vs. `overlays/prod`):

| Environment | ArgoCD Application | Branch (`targetRevision`) | Overlay         | Namespace | ESP32 Target IP         |
|-------------|--------------------|---------------------------|-----------------|-----------|-------------------------|
| **Dev**     | `hauswerk-dev`     | `main`                    | `overlays/dev`  | `dev`     | `192.168.177.49` / `.50`|
| **Prod**    | `hauswerk-prod`    | `main`                    | `overlays/prod` | `prod`    | `192.168.177.28`        |

**Workflow:**
1. **Frontend / Firmware / Bootloader:** Images werden in CI via Multi-Stage Docker gebaut und nach Zot (`192.168.178.183:5010`) gepusht.
2. **Dev-Deployment:** CI triggert das zentrale `deployments`-Repo (`deploy.yml`) $\rightarrow$ `overlays/dev` wird aktualisiert $\rightarrow$ ArgoCD synct automatisch.
3. **Prod-Deployment:** Promotion über `deployments` (`deploy.yml` / Deployment-Dashboard) nach `overlays/prod`, abgesichert durch das DEV-Promotion-Gate (`git log -S` auf `overlays/dev/kustomization.yaml`) $\rightarrow$ ArgoCD synct Prod-Dienste.

## Services & Workloads

| Service / Workload       | Typ                  | Port(s) / Funktion (PROD / DEV)                     |
|--------------------------|----------------------|-----------------------------------------------------|
| Mosquitto                | NodePort             | 1883 → 31884 / 31885 (TCP), 9001 → 32002 / 32003 (WS)|
| Frontend                 | NodePort             | 80 → 30080 / 30081 (WebUI Dashboard)                |
| Deployment Dashboard     | NodePort             | 80 → 30090 / 30091 (Deployment Control UI)          |
| esp32-flasher            | Container (On-Demand)| Flasht ESP32 Firmware über WireGuard/LAN via `deploy.yml` |
