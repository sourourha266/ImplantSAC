# ImplantSAC
 
An AI system that classifies dental implant cases as Straightforward, Advanced, or Complex, following the ITI SAC framework used in clinical implant planning.
 
This repository documents the system's design, methodology, and results. It does not include source code or patient data, since both involve institutional and IP considerations tied to the project's academic and clinical partners.
 
## Overview
 
Implant case difficulty is normally assessed by a clinician using the ITI SAC classification, which scores a case across a set of clinical factors and assigns it one of three risk levels: Straightforward, Advanced, or Complex. ImplantSAC automates this assessment by combining four modalities that a clinician would normally review separately:
 
- CBCT imaging (bone quality and volume factors)
- Panoramic X-rays (lesion and structural detection)
- Intraoral photographs (soft tissue and prosthetic space factors)
- Tabular clinical history (18 ITI SAC factors)
Each modality is scored independently, then combined using a worst-case aggregation rule: the final SAC classification for a case is never lower risk than the highest risk factor identified across any single modality. This mirrors clinical practice, where one serious complicating factor is enough to escalate a case, regardless of how straightforward the rest of the presentation looks.
 
## Why this matters
 
Implant treatment planning errors are costly and largely preventable with better upfront risk stratification. Standardizing the SAC assessment with a multimodal AI pipeline gives less experienced clinicians a consistent second read on case difficulty, and gives experienced clinicians a faster way to document their reasoning.
 
## System architecture
 
The system is organized as four independent pipelines, each producing per-factor risk scores, followed by a cross-modality aggregation step.
 
| Pipeline | Input | Core models |
|---|---|---|
| CBCT | 3D cone beam CT scans | Router across 7 CNN/transformer backbones (DenseNet121, YOLOv8, Detectron2, TorchXRayVision, ConvNeXt-V2, EfficientNetV2-S, DINOv2 ViT-S/14) |
| Panoramic | 2D panoramic X-rays | YOLOv8s detector + 3-head ResNet50 classifier (DentalNet3Head) |
| Intraoral | Clinical photographs | Merged-class YOLOv8s tooth detector + ResNet50 severity classifier |
| Tabular | 18-factor clinical history | TabNet, majority-colour vote aggregation |
 
See `docs/architecture.md` for a detailed breakdown of each pipeline.
 
## Key results
 
| Metric | Value |
|---|---|
| Panoramic tooth/lesion detector AUC_ROC | 92.73% |
| Panoramic classifier calibration (ECE), before / after Platt scaling | 13.70% to 1.41% |
| Intraoral tooth detector mAP50 | 94.55% |
| Intraoral tooth detector recall | 89.67% |
| CBCT ensemble accuracy (with reg-A override) | ~66 to 67%, up from ~61% baseline |
| Tabular pipeline worst-case accuracy | 95.1% |
| Tabular pipeline safety property | No Complex case is ever downgraded to Straightforward |
 
Full breakdown by factor and model is in `docs/results.md`.
 
## Clinical framework
 
The system evaluates cases against the 18 ITI SAC factors, spanning bone quality and volume, anatomical risk, esthetic risk, and patient factors. Factor definitions and how each maps to a modality are documented in `docs/methodology.md`.
 
## Project status
 
The thesis was defended on July 8, 2026. The project is currently in post-defense development, with a UI validation phase underway to test the end-to-end pipeline against curated test cases.
 
## Team and attribution
 
- **Author:** Sourour Hammoud, M.Eng. Computer & Communications Engineering, Lebanese University, Faculty of Engineering (Tripoli/ULFG1)
- **Supervisor:** Dr. Samer El Zant
- **Collaboration:** ZAKA
- **Annotation protocol validation:** Dr. Stephanie Mrad Hawi, DDS, MSc, PhD candidate, of Dr. Joe Basil Clinic
## Data and code availability
 
Source code, trained model weights, and the underlying datasets (CBCT, panoramic, intraoral, and clinical history data) are not included in this repository due to patient privacy and institutional IP considerations. Access requests can be directed to the project supervisor.
 
## Contents of this repository
 
```
ImplantSAC/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── results.md
│   └── methodology.md
├── poster/
└── presentation/
```
