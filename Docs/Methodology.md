# Methodology
 
## The ITI SAC framework
 
SAC stands for Straightforward, Advanced, Complex. It is a clinical classification framework used to categorize the difficulty of an implant case before treatment planning. A case is assessed across a defined set of factors spanning bone quality, bone volume, anatomical risk, esthetic risk, and patient-related considerations. The overall case classification is driven by the single highest-risk factor identified, not an average across factors. ImplantSAC follows this same worst-case logic in its aggregation step.
 
The system evaluates all 18 ITI SAC factors through the tabular pipeline, and re-derives a subset of image-assessable factors directly from CBCT, panoramic, and intraoral data. Where an image-derived score and a reported clinical history score disagree for the same factor, the worst-case rule applies at the case level, so the higher-risk assessment governs the final classification.
 
## Datasets
 
| Pipeline | Dataset | Size / source |
|---|---|---|
| CBCT | Public release of Cui et al. (2022), 50 CBCT volumes | 538 annotated sagittal images |
| Panoramic | Primary set (panoramic_116 and panoramic_1000), Kaggle panoramic-dental, DENTEX, PerioXrays | Four datasets: the primary XML-annotated pair plus three supplementary public sources added to raise the lesion-positive pool |
| Intraoral | interoral_Dataset | 6,265 images |
| Intraoral (Factor 13) | Mendeley gingivitis dataset | Exclusive source of MGI severity labels |
| Tabular | Synthetic clinical history | 10,000 rows, generated to reflect realistic ITI factor distributions |
 
The CBCT measurement protocol was established from example sagittal images pre-annotated by Dr. Stephanie Mrad Hawi, DDS, MSc, PhD candidate, of Dr. Joe Basil Clinic, demonstrating how each of the five bone factors should be measured. Panoramic lesion ground truth was produced separately, by Dr. Hanine Zaidan.
 
## Validation approach
 
Each pipeline was developed and validated independently before integration into the aggregation layer. Development followed an iterative debugging pattern: initial model choices were revisited when a particular framing failed (as with Factor 13's initial 3-class formulation), and metrics were re-evaluated against the clinically relevant failure mode rather than treated as a single leaderboard number.
 
TabNet, the model actually selected for deployment, does not use SHAP; its per-factor attribution is read directly from its own attention masks. This distinction, between SHAP's perturbation-based estimate and TabNet's native, deterministic attribution, is the basis for preferring TabNet: attribution stability and native handling of missing fields, not interpretability alone.
 
## Post-defense validation phase
 
Following the thesis defense, work has continued on UI-level validation. This involves selecting known test cases from the source datasets, such as identifying the highest-confidence lesion case for a specific tooth position in the panoramic dataset, or identifying a fully healthy case using the Factor 13 binary classifier in the gingivitis dataset, to confirm the end-to-end system behaves correctly on cases with known ground truth before any broader testing.
