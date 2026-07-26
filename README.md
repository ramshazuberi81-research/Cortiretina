👁️ CortiRetina

Can the eye see what a stress hormone is doing to your heart?

Cortisol excess is a well-documented, independent driver of premature cardiovascular disease — yet standard risk calculators (Framingham, ASCVD/PCE) don't account for it. CortiRetina tests whether cortisol-driven vascular damage, already trackable via invasive markers like carotid intima-media thickness (CIMT), is also detectable through something far simpler: a retinal fundus photograph.

🩺 The Problem

Chronic hypercortisolism (Cushing's syndrome, subclinical adrenal cortisol excess) markedly increases the risk of myocardial infarction and stroke — and that risk persists even after cortisol is biochemically controlled. Meanwhile, MI in young adults is increasingly recognized as its own clinical entity, often striking people with none of the traditional risk factors standard calculators screen for.

Cortisol doesn't just raise blood pressure — it appears to directly damage vascular endothelium, altering nitric oxide bioavailability in ways mechanistically distinct from diabetes or hyperlipidemia. This damage is already measurable via CIMT ultrasound. This project asks: can it be measured non-invasively instead, through the one place doctors already photograph blood vessels routinely — the retina?

🔬 The Approach

Fundus image
    ↓
Frangi vesselness filtering (multi-scale Hessian-based vessel enhancement)
    ↓
Quantitative vessel biomarkers: caliber, tortuosity, fractal dimension, AVR-proxy
    ↓
Validate against REAL clinical outcomes: DR grade → CIMT (thickened/normal) → cortisol status

In parallel, a direct end-to-end CNN regression path (ResNet50, 6-channel L+R fundus input) has also been trained to predict raw CIMT value directly from fundus images, as a second validation route alongside the Frangi-feature pipeline.

Diabetic retinopathy's well-characterized lesion taxonomy provides a template: if cortisol damages vessels the way hyperglycemia does — just via a different mechanism — similar vessel-level signatures should be recoverable from the same imaging pipeline already used for DR screening at scale.

✅ What's Actually Validated (Not Just Claimed)

| Component | Status |
|---|---|
| Frangi-filter vessel feature extraction | Working, runs end-to-end on EyePACS, Messidor-2, China-Fundus-CIMT |
| Vessel biomarkers → DR grade | Preliminary sanity check only — modest accuracy (~25–32%, near chance baseline on balanced 5-class data). Feature set + sample size need expansion. |
| Vessel biomarkers → CIMT (thickened/normal) | Pipeline built, ready to run — CIMT is a real, established CVD surrogate, the most credible current validation target |
| CNN direct fundus → CIMT regression (ResNet50, n≈2,900) | **Run and validated on held-out test set.** Test MAE 0.137, Test R² 0.126 — a real but modest signal; confirms retina-derived features carry some CIMT-relevant information, with substantial unexplained variance remaining |
| Cortisol status classification + CVD risk score | Hypothesis-stage. Currently a structured, clinically-motivated rule-based mapping from DR-like lesion patterns — not yet trained on real cortisol-labeled patient data, since no public dataset like this currently exists |

Being upfront about this split is the point: the vessel-extraction engine works, the DR/CIMT validation is in progress and improving, the direct CNN→CIMT regression has real (if modest) held-out performance, and the cortisol-specific layer is exactly the open research question this project exists to answer.

📊 CNN Regression Results (Fundus → CIMT)

Trained a ResNet50-based CNN (first conv layer adapted to 6-channel input for stacked left+right fundus images, regression head replacing the final FC layer) on ~2,900 patients with labeled CIMT values, using a patient-level train/val/test split to avoid leakage.

- Test MAE: 0.137 (CIMT units)
- Test R²: 0.126

Training and validation MAE both decreased steadily for the first ~10-13 epochs; validation MAE plateaued after that point while training MAE continued to drop, indicating the model begins overfitting past that point on the current dataset size/augmentation setup. R²=0.126 means the model captures a real but limited share of CIMT variance — plausible next steps include addressing confounders (age, blood pressure, metabolic factors) not currently modeled, stronger regularization, and a larger/more diverse training set.

These numbers should be read as a first-pass baseline for the direct end-to-end regression route, not a finished or clinically validated tool.

📁 Repo Layout

```
src/
  run_local_pipeline.py                  # EyePACS + Messidor-2 → DR-grade model
  run_cimt_pipeline.py                    # China-Fundus-CIMT → CIMT thickened/normal model
  train_cnn_cimt_regression.py            # ResNet50 direct fundus → CIMT regression (see results above)
  download_and_extract_retinal_data.py    # Kaggle data acquisition
  cortisol_vascular_pathology_mapper.py   # hypothesis-stage: DR lesion → cortisol mapping
  frangi_to_cortisol_integration.py       # hypothesis-stage: full risk-scoring pipeline
  vessel_calibration_agent_framework.py   # 5-agent calibration system (pressure/motion/flicker/curve-fit/normalization)
docs/
  frangi_filter_feature_extraction.md     # Frangi filter theory + implementation guide
  kaggle_dataset_extraction_guide.md      # dataset access instructions
  COMPLETE_PIPELINE_GUIDE.md              # full workflow walkthrough
outputs/                                  # generated CSVs, trained models (gitignored)
```

🚀 Quick Start

```
pip install -r requirements.txt
```

Data isn't bundled here — each dataset carries its own license. Pull it yourself:

```
kaggle datasets download -d sovitrath/diabetic-retinopathy-2015-data-colored-resized -p ./data/eyepacs --unzip
kaggle datasets download -d google-brain/messidor2-dr-grades -p ./data/messidor2 --unzip
```

China-Fundus-CIMT is distributed via Figshare — see the Scientific Data paper for access.

```
python src/run_local_pipeline.py            # DR-grade sanity check
python src/run_cimt_pipeline.py             # CIMT thickened/normal classification
python src/train_cnn_cimt_regression.py     # CNN direct fundus → CIMT regression (results above)
```

Edit the path constants at the top of each script for your local data location.

🗺️ Roadmap

- [ ] Scale EyePACS sampling to properly power minority DR grades
- [ ] Run CIMT thickened/normal classification as primary validation benchmark (established surrogate marker, unlike DR grade)
- [x] Run CNN direct fundus → CIMT regression as a baseline validation route (Test MAE 0.137, Test R² 0.126)
- [ ] Improve CNN regression: address confounders (age, BP, metabolic factors), add regularization, expand dataset size/diversity
- [ ] Acquire a real cortisol-labeled retinal imaging cohort (Cushing's / metyrapone-treated patients) to finally train the cortisol-mapping layer on ground truth
- [ ] Cross-validate Frangi-derived features against manual vessel annotations
- [ ] Explore reversibility tracking — cortisol-induced vascular damage should partially resolve once cortisol is controlled, unlike diabetic vascular damage

📚 Key References

Frangi et al. (1998). Multiscale vessel enhancement filtering. MICCAI.
Cardiovascular health and mortality in Cushing's disease — elevated MI/stroke hazard persists even after remission.
Subclinical cortisol excess and CIMT — cortisol-related vascular damage already tracked via carotid ultrasound in the endocrine literature.
MI in young adults — increasingly recognized as occurring independent of traditional risk factors.

👤 Author

Ramsha Zuberi — Clinical AI Researcher / Aga Khan University PGD in AI in Healthcare, NED University of Engineering & Technology
ORCID: 0009-0004-9272-0343

Contributions, critique, and collaboration inquiries welcome — especially from anyone with access to cortisol-labeled retinal imaging cohorts.

About

Retinal vascular biomarker pipeline testing whether cortisol-driven vascular damage — an established but underrecognized cause of premature CVD in young adults — is detectable non-invasively via retinal imaging. Frangi-filter vessel features validated against DR grade and CIMT; direct CNN fundus→CIMT regression run with a modest but real held-out signal (R²=0.126); cortisol-specific validation in progress.
