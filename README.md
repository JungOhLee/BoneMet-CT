# BoneMet-CT

**Expert-level bone metastasis detection on body CT — trained with MRI + PET/CT reference standards.**

[![Paper](https://img.shields.io/badge/Radiology%3A%20AI-10.1148%2Fryai.250283-1f4e79)](https://doi.org/10.1148/ryai.250283)
[![Models](https://img.shields.io/badge/Models-Zenodo%2010.5281%2Fzenodo.14872632-1682b4)](https://zenodo.org/records/14872632)
[![Commentary](https://img.shields.io/badge/Commentary-10.1148%2Fryai.260247-6c757d)](https://doi.org/10.1148/ryai.260247)
[![License](https://img.shields.io/badge/Models-CC%20BY--NC%204.0-lightgrey)](https://creativecommons.org/licenses/by-nc/4.0/)

BoneMet-CT is the open-source release of the deep learning models from our study in *Radiology: Artificial Intelligence* (RSNA, 2026). The models detect bone metastases on routine thoracic and abdominal CT, reach the performance of musculoskeletal radiologists, and run in **under a minute per scan** — so they can sit inside a clinical image viewer as a second-reader "safety net."

> **Paper** — Lee JO, Kim DH, et al. *Bone Metastasis Detection at CT with Deep Learning Models Trained Using Multicenter, Multimodal Reference Standards: Development and Evaluation.* Radiology: Artificial Intelligence 2026;8(3):e250283. https://doi.org/10.1148/ryai.250283
>
> **Model weights** — https://zenodo.org/records/14872632

---

## Why it matters

Bone is one of the most common sites of metastasis — especially in breast, prostate, and lung cancer — and missed lesions can lead to pathologic fracture and spinal cord compression. Yet on a busy oncologic CT read, where the radiologist is also assessing soft tissue, measuring tumor burden, and tracking response, skeletal metastases are a leading source of false-negative errors. A reliable detector that flags suspicious bone lesions could act as a safety net without adding to the radiologist's workload.

The subtle problem most prior CT detectors share: they are trained **and** graded against labels drawn from CT itself. That silently ignores the lesions CT cannot show. We asked a harder question — **what if the ground truth comes from MRI and PET/CT, not CT?**

## The core idea: a multimodal reference standard

For every patient we paired each CT with **whole-spine MRI** and **FDG PET/CT** acquired within 15 days, then labeled each lesion by how it looks *on CT*:

| CT visibility | Definition |
|---|---|
| **Visible** | Clearly identifiable on CT |
| **Indeterminate** | Subtle/equivocal on CT, but confirmed metastatic on MRI or PET |
| **Invisible** | Undetectable on CT, confirmed only on MRI/PET (excluded from training) |

To measure how much the *label definition itself* matters, we trained two otherwise-identical models:

- **Model 1** — trained on **CT-visible lesions only** (the conventional approach)
- **Model 2** — trained on **visible + indeterminate lesions** (the multimodal standard)

## Dataset

- **4 centers** (South Korea), **332 patients**, **502 CT scans**, **4,999 annotated bone metastases**, >12 cancer types (lung + breast ≈ 56%).
- Centers 1–3 → training/validation; **Center 4 → external test set** — held out, the only set containing cancer-without-metastasis negative controls, and with ~⅓ of its scans from scanner models never seen in training.
- Lesions annotated by two MSK radiologists (10 and 14 years' experience) by consensus on CT while referencing MRI and PET/CT; substantial inter-reader agreement on visibility (Cohen κ = 0.72).

## Key findings

Performance on the external test set (Model 2 = recommended):

| Reader / Model | Lesion precision | Lesion recall | Scan-level AUROC |
|---|---|---|---|
| **Model 2** (visible + indeterminate) | **80.1%** | **41.8%** | **0.80** (0.68–0.90) |
| Model 1 (visible only) | 78.8% | 33.9% | 0.76 (0.64–0.87) |
| Musculoskeletal radiologists (n = 3) | 66.5% | 43.8% | 0.87 (0.80–0.93) |
| Radiologists-in-training (n = 3) | 66.6% | 39.4% | 0.87 (0.80–0.93) |

- **Both models beat radiologists on precision** (≈80% vs ≈66%) — substantially fewer false alarms.
- **Model 2 matches radiologists on recall and scan-level AUROC** (P = .47 and .09 vs MSK radiologists), while **Model 1 falls short**. The *only* difference between the two models is the reference standard — adding MRI/PET-confirmed *indeterminate* lesions to training is what closed the gap (recall 41.8% vs 33.9%, P < .001).
- The recall figures look modest (~42%) **by design**. Because the bar is set by MRI/PET — counting lesions CT genuinely struggles to show — even expert radiologists only reached ~44%. This is a more honest benchmark than CT-defined labels, under which earlier studies reported far higher numbers.
- **Generalization** — an additional check on the public **noncontrast** dataset (TCIA Spine-Mets-CT-SEG) showed the models transfer beyond the predominantly contrast-enhanced training data.
- **Honest failure modes** — Model 2's false positives were mostly benign mimics that challenge radiologists too: fractures (40%), bone islands (19%), degenerative change, Schmorl nodes, and vertebroplasty.

## Models & usage

Both models are archived on **Zenodo** ([10.5281/zenodo.14872632](https://zenodo.org/records/14872632), CC BY-NC 4.0):

- **Model 2 — nnU-Net Task 503** (visible + indeterminate) — *recommended for clinical use.*
- **Model 1 — nnU-Net Task 504** (visible only) — provided for reproducibility/ablation.

Inference outline:

1. Install [nnU-Net](https://github.com/MIC-DKFZ/nnUNet).
2. Download the weights from Zenodo and place them in your nnU-Net results directory.
3. Run prediction on axial CT volumes (NIfTI) with the `3d_lowres` configuration.
4. Post-process: drop predictions smaller than a 5 mm sphere, then convert segmentation masks to bounding boxes for detection.

> The exact `nnUNetv2_predict` command depends on your installed nnU-Net version — see the nnU-Net docs. **For research use only; not for clinical use.**

## Editorial commentary

The study is accompanied by an invited commentary in the same issue:

> Khosravi P. *Seeing Beyond CT: Multimodal Data and the Next Frontier of AI for Bone Metastasis Detection.* Radiology: Artificial Intelligence 2026;8(3):e260247. https://doi.org/10.1148/ryai.260247

Khosravi frames the work around a principle at the heart of this project: *"the quality and completeness of the reference standard may be as critical as the sophistication of the model itself."* She highlights the multimodal reference standard as the study's defining contribution — and notes that the lower-but-more-realistic recall "likely reflects a more realistic assessment of the intrinsic difficulty of detecting early or subtle metastatic disease at CT," rather than a weakness of the models.

As Khosravi concludes, *"future progress may depend as much on how we define and curate the truth within our datasets as on the models we use to learn from them."*

## Citation

If you use these models or build on this work, please cite the paper:

```bibtex
@article{lee2026bonemet,
  title   = {Bone Metastasis Detection at CT with Deep Learning Models Trained
             Using Multicenter, Multimodal Reference Standards: Development and Evaluation},
  author  = {Lee, Jung-Oh and Kim, Dong Hyun and Chae, Hee-Dong and Lee, Eugene
             and Kang, Ji Hee and Lee, Ji Hyun and Kim, Hyo Jin and Seo, Jiwoon
             and Chai, Jee Won},
  journal = {Radiology: Artificial Intelligence},
  volume  = {8},
  number  = {3},
  pages   = {e250283},
  year    = {2026},
  doi     = {10.1148/ryai.250283}
}
```

## License

- **Paper** — © The Author(s) 2026, published by RSNA under CC BY 4.0.
- **Model weights** (Zenodo) — CC BY-NC 4.0 (non-commercial).
