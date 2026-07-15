# Architecture
 
ImplantSAC is built as four independent modality pipelines feeding into a single aggregation layer. Each pipeline was developed and validated separately before integration.
 
## 1. CBCT pipeline
 
Processes 3D cone beam CT scans to assess five bone-related SAC factors: apical position, buccal bone, fenestration, horizontal bone volume, and vertical bone volume.
 
Seven model architectures were evaluated across a 538-image dataset:
 
- M1: DenseNet121
- M2: YOLOv8
- M3: Detectron2
- M4: TorchXRayVision
- M5: ConvNeXt-V2
- M6: EfficientNetV2-S
- M7: DINOv2 ViT-S/14
Rather than picking one backbone for all factors, a per-factor router selects the best-performing model for each specific factor:
 
- M7 (DINOv2) for apical position, buccal bone, and fenestration
- M5 (ConvNeXt-V2) for horizontal bone volume
- M1 (DenseNet121) for vertical bone volume
A coordinate-curriculum MLP is trained in three phases to localize anatomical landmarks that feed into the factor scoring. An ensemble technique referred to as the reg-A override combines model outputs to lift overall accuracy from approximately 61% to 66 to 67%. Tolerance margins used in scoring are grounded in published CBCT measurement-accuracy literature rather than arbitrary thresholds.
 
## 2. Panoramic pipeline
 
A two-stage system applied to 2D panoramic X-rays:
 
**Stage 1: Detection.** A YOLOv8s detector locates teeth and lesions, achieving mAP50 of 99.5%.
 
**Stage 2: Classification.** DentalNet3Head, a ResNet50 backbone with three prediction heads, handles classification, location, and bounding box refinement simultaneously for each detected finding.
 
Raw classifier confidence scores were poorly calibrated. Platt scaling was applied post-hoc, reducing expected calibration error (ECE) from 13.70% to 1.41%, which makes the model's confidence scores meaningfully usable in the aggregation step rather than just a ranking signal.
 
## 3. Intraoral pipeline
 
Processes clinical photographs to assess three SAC factors:
 
- Factor 10: prosthetic and interocclusal space
- Factor 11: edentulous span
- Factor 13: soft tissue inflammation (gingivitis severity)
A merged-class YOLOv8s tooth detector was trained on 6,265 images, reaching mAP50 of 94.55% and recall of 89.67%.
 
Factor 13 required multiple reframings before it worked reliably. An initial 3-class formulation (mapping to mild, moderate, severe inflammation) failed due to severe class imbalance in the training data. Reframing it as a binary NOT_SEVERE / SEVERE classification resolved this. ResNet50 was selected over EfficientNet-B4 for this factor based on higher recall on the SEVERE class (0.729 vs 0.697), which matters more than raw accuracy since missing a severe case is the costlier error clinically.
 
MGI (Modified Gingival Index) severity labels are sourced exclusively from the Mendeley gingivitis dataset, with raw MGI scores collapsed into SAC-relevant categories via a defined MGI-to-SAC mapping. No image augmentation was used for Factor 13 training. All training examples are real, unaugmented crops, and class balancing was handled through ratio-based sampling instead.
 
## 4. Tabular pipeline
 
Processes structured clinical history across all 18 ITI SAC factors using TabNet, trained on 10,000 synthetic rows generated to reflect realistic factor distributions.
 
Predictions across factors are combined using a majority-colour vote, where each factor is color-coded by its SAC risk level (green, yellow, red) and the case-level classification follows the majority signal, subject to the worst-case override described below.
 
## Cross-modality aggregation
 
All four pipelines feed into a single worst-case aggregation rule: the final case classification is set to the highest risk level identified by any single factor across any single modality. A case is never classified as lower risk than its most complicating individual factor. This is enforced as a hard safety property in the tabular pipeline: no case scored Complex by any factor can be output as Straightforward.
 
## Clinical decision support interface
 
A Streamlit application was built to expose pipeline outputs to clinicians, including per-factor threshold gauges, model-agreement visualizations, and clinical interpretation cards. The interface accepts image and medical information inputs. 
