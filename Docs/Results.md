# Results
 
## CBCT pipeline
 
| Metric | Value |
|---|---|
| Dataset size | 538 images |
| Factors assessed | 5 (apical position, buccal bone, fenestration, horizontal volume, vertical volume) |
| Baseline ensemble accuracy | ~61% |
| Accuracy after reg-A override | ~66 to 67% |
 

| Model | Strict (%) | Tol. v1 (%) | Tol. v2 (%)  | ECE (%) |
|---|---|---|---|---|---|
| M1 DenseNet121 | 61.7 | 87.9 | 76.6 | 6.8 |
| M2 YOLOv8 | 38.1 | 79.9 | 70.1  | 33.7 |
| M4 TorchXRayVision | 42.6 | 78.6 | 64.9  | 28.2 |
| M5 ConvNeXtV2 | 54.8 | 86.8 | 80.1  | 8.9 |
| M6 EfficientNetV2-S | 64.1 | 89.4 | 78.3 | 12.8 |
| M7 DINOv2 | 65.8 | 86.4 | 77.5  | 6.6 |

Per-factor best model:
 
| Factor | Best model |
|---|---|
| Apical position | M7, DINOv2 ViT-S/14 |
| Buccal bone | M7, DINOv2 ViT-S/14 |
| Fenestration | M7, DINOv2 ViT-S/14 |
| Horizontal volume | M5, ConvNeXt-V2 |
| Vertical volume | M1, DenseNet121 |
 
Fenestration is scored as a binary presence/absence factor. There is no severity threshold or graded scale for this factor.
 
## Panoramic pipeline
 
 
| Head | Metric | Value |
|---|---|---|
| Detector | mAP50 (teeth and lesions) | 99.5% |
| Head 1: Presence | AUC-ROC | 92.73% |
| | Average Precision | 92.62% |
| | Sensitivity (at τ=0.853) | 76.81% |
| | Specificity | 94.26% |
| | PPV | 90.06% |
| | NPV | 85.73% |
| | F1 | 82.91% |
| Head 2: Location | Overall accuracy | 86.98% |
| | Crown F1 (n=728) | 90% |
| | Middle-root F1 (n=1,130) | 88% |
| | Apex F1 (n=277) | 75% |
| Head 3: Bounding box | Mean IoU | 32.98% |
| | IoU > 25% | 1,196/2,135 |
| | IoU > 50% | 558/2,135 |
| | Volume correlation r | 78.29% |
 
## Intraoral pipeline
 
| Metric | Value |
|---|---|
| Tooth detector mAP50 | 94.55% |
| Tooth detector recall | 89.67% |
| Training images | 6,265 |
| Factor 13 model | ResNet50 |
| Factor 13 SEVERE recall (ResNet50) | 0.729 |
| Factor 13 SEVERE recall (EfficientNet-B4, not selected) | 0.697 |
 
Factor 13 went through two framings during development. A 3-class formulation was abandoned due to class imbalance. The binary NOT_SEVERE / SEVERE formulation that replaced it is the one used in the final pipeline.
 
## Tabular pipeline
 
| Metric | Value |
|---|---|
| Factors covered | 18 (all ITI SAC factors) |
| Training data | 10,000 synthetic rows |
| Worst-case accuracy | 95.1% |
| Safety property | No Complex case is ever downgraded to Straightforward |
 
## Notes on evaluation methodology
 
Model selection throughout the project favored the metric most relevant to clinical cost of error rather than raw accuracy alone. For Factor 13, this meant selecting ResNet50 over EfficientNet-B4 despite similar overall accuracy, because ResNet50 caught more true SEVERE cases. For the tabular pipeline, the worst-case safety property was treated as a hard constraint rather than something to trade off against average accuracy.
 
Backbone comparisons for the intraoral pipeline (three candidate backbones) were run through Optuna-based hyperparameter search rather than manual tuning, to keep the comparison consistent across architectures.
