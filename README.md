# Predictive Maintenance Digital Twin for Electric Vehicles

> **Status:** 🗂️ _Archived_ (cloud resources stopped to avoid charges)  
> **Scope:** Documentation-first repository showcasing the architecture, pipeline design, and ML approach for a cost‑aware EV predictive‑maintenance digital twin.

![Architecture](docs/architecture-diagram.png "Add your diagram here")

## Overview
This project designed a **serverless, scalable** pipeline that simulates EV telemetry and performs **Time‑To‑Failure (TTF)** prediction with real‑time visualization. It integrates **AWS IoT Core**, **IoT SiteWise**, **Lambda**, **EventBridge**, **S3**, **SageMaker**, **IoT TwinMaker**, and **Amazon Managed Grafana** into a cohesive digital‑twin workflow.

### Why this repo exists
- The live Grafana/TwinMaker dashboards have been **shut down** to prevent cloud costs.  
- This repository preserves a **professional record**: problem framing, cloud architecture, ML approach, and how the system worked end‑to‑end.
- When sponsors/resources are available, the stack can be re‑provisioned from this documentation.

## Highlights
- **Data simulation**: EventBridge triggers a Lambda every 15 minutes to stream the next CSV row from S3 into **IoT SiteWise** as structured time series.
- **Prediction loop**: A second Lambda detects a `TriggerPrediction` flag, pulls the latest features, calls a **SageMaker** endpoint (Random Forest) for TTF, and writes `PredictedTTF` and `FailureSoon` back to SiteWise.
- **Visualization**: **IoT TwinMaker** (3D scene) + **Managed Grafana** dashboards for real‑time insight and alerts.

_The end‑to‑end flow and implementation details are documented in_ `docs/DigitalTwin_Report.pdf`.


## Architecture (serverless pattern)
```
S3 (CSV + model) ─→ EventBridge (15m) ─→ Lambda (ingest row)
                                │
                                ├──→ IoT SiteWise (telemetry + TriggerPrediction)
                                └──→ SSM Parameter (row index)

IoT SiteWise (TriggerPrediction) ─→ Lambda (inference) ─→ SageMaker Endpoint (RF)
                                                   │
                                                   └──→ IoT SiteWise (PredictedTTF, FailureSoon)

IoT SiteWise ─→ IoT TwinMaker (3D scene) ─→ Managed Grafana (dashboards)
```
- **Resilient & scalable**: No servers to manage; pay‑as‑you‑go services.
- **Traceable**: SiteWise is the single source of truth for time‑series and prediction flags.
- **Cost‑aware**: Can be paused entirely without data/code loss.

## Machine Learning
- **Model**: Random Forest Regressor (SageMaker‑hosted) predicting TTF; a derived flag `FailureSoon = (TTF < 48h)` supports alerting.
- **Rationale**: RF enabled **fast, lightweight deployment** compared to the higher‑accuracy CNN+BiLSTM prototype that was heavier to serve.
- **Limitations**: Baseline performance is moderate; future work targets sequence models and class imbalance handling.

> See `docs/DigitalTwin_Report.pdf` for training/evaluation notes, deployment scripts, and Lambda code snippets.

## Repository layout
```
ev-digital-twin-predictive-maintenance/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ SECURITY.md
├─ CONTRIBUTING.md
├─ .github/workflows/docs.yml
├─ docs/
│  ├─ DigitalTwin_Report.pdf     # full project report
│  └─ architecture-diagram.png   # add your diagram export here
├─ data/
│  └─ README.md                  # policy: samples only, no private data
├─ notebooks/                    # optional: exploration
├─ infra/                        # optional: IaC later
└─ src/                          # optional: client/server code later
```

## Getting started (local docs)
1. Clone this repository once it’s on GitHub:
   ```bash
   git clone https://github.com/<your-username>/ev-digital-twin-predictive-maintenance.git
   ```
2. Open `docs/DigitalTwin_Report.pdf` to review the system and ML setup.
3. Add screenshots/diagrams into `docs/` (optional).

## Re‑provision later (high level)
- Export IaC (Terraform/CDK/SAM) into `infra/` when budgets allow.
- Rebuild the SageMaker endpoint from training scripts, then set:
  - Lambda(ingest) → SiteWise property/asset IDs
  - Lambda(inference) → SageMaker `ENDPOINT_NAME`
  - EventBridge schedule → 15‑minute cadence (or as needed)

## Status and costs
- All cloud resources are **stopped**. No active endpoints/dashboards are running.
- This repo intentionally contains **no secrets** and **no private datasets**.

## License
MIT — see `LICENSE`.

---

### Author
Hafdoon Muhammed — Computer Engineering (UTP). See the CV and project summary in the repo and on LinkedIn/GitHub profiles.
