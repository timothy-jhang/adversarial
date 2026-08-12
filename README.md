# Near-Perfect High-Frequency Adversarial Detection Is an Upscaling Artifact

Reproduction code and notebooks for:

> J.-R. Lee and K. Jhang, **"Near-Perfect High-Frequency Adversarial Detection Is an Upscaling Artifact: A Four-Axis Protocol for Diagnosing Frequency Detectors."**

## Overview

Several published pixel/frequency adversarial-example detectors report near-perfect accuracy
(AUROC > 0.99). This work shows that this near-perfect performance is largely an **artifact of the
benchmark pipeline**: standard benchmarks upscale low-resolution images (e.g., CIFAR-10 32×32 → 224)
*before* generating the attack. Bicubic upscaling empties the image's own high-frequency band, and the
perturbation added at the target resolution becomes the only signal left in that band — so a
high-frequency detector separates clean from adversarial almost perfectly for reasons that have nothing
to do with adversarialness.

To diagnose this, we introduce a **four-axis evaluation protocol** that stress-tests a detector along
four axes that prevailing (clean-only, AUROC-only, upscaled, non-adaptive) evaluation hides:

1. **Hard negatives** — benign perturbations (matched noise, JPEG, blur) that occupy the same
   high-frequency band, testing whether the detector measures adversarialness or merely high frequency.
2. **Native resolution** — evaluation without the upscaling step.
3. **Adaptive attacks** — a detector-aware attacker (C&W-style objective with BPDA for non-differentiable
   defenses).
4. **Operational TPR at a fixed FPR** — deployment-realistic thresholds set on benign inputs.

The repository reproduces every table and figure in the paper.

## Repository layout

```
notebooks/   Jupyter notebooks, one (or a few) per paper table/figure  (see mapping below)
results/     Cached numerical outputs (JSON/CSV/PKL) produced by the notebooks
requirements.txt
```

## Setup

```bash
python -m venv venv && source venv/bin/activate      # Python 3.10+ recommended
pip install -r requirements.txt
```

A CUDA-capable GPU is recommended (experiments were run on a single NVIDIA A100 / RTX A5000 / RTX 3090).
The notebooks fall back to CPU but will be slow.

## Data and checkpoints

The notebooks operate on cached attack sets and fine-tuned classifiers rather than regenerating
everything from scratch. Each notebook locates its inputs automatically with a recursive `find(...)`
helper, so place the following anywhere under the working directory:

- **Cached attack sets** named `mixed_dataset.pkl` — a list of `(image, label, attack)` tuples, where
  `image` is a `[0,255]` tensor and `attack` is `'clean'` or an attack name (`FGSM`/`PGD`/`C&W`/`AutoAttack`).
  The canonical caches are:
  - `cifar10_detector_results/mixed_dataset.pkl` — CIFAR-10, upscaled ×7 (32→224)
  - `imagenet_val_detector_results_eps8/mixed_dataset.pkl` — ImageNet validation, native 224, ε = 8/255
- **Fine-tuned backbones** for the low-resolution sets: `resnet50_cifar10_finetuned.pt`,
  `resnet50_cifar100_finetuned.pt`, `resnet50_svhn_finetuned.pt`, `resnet50_tinyimagenet_finetuned.pt`.
  The ImageNet backbone uses public torchvision weights (`ResNet50_Weights.IMAGENET1K_V2`;
  `ViT-B/16 IMAGENET1K_V1` for the transformer experiments) and needs no checkpoint.

Fine-tuning recipe (for regenerating checkpoints): SGD (momentum 0.9, weight decay 1e-4), discriminative
learning rates (1e-4 backbone / 1e-3 head), cosine annealing, batch size 128, 15 epochs (10 for SVHN),
best-validation checkpoint; inputs bicubically upscaled to 224. All classifiers are standard,
non-robust models.

## Notebook → paper artifact map

Notebooks are self-contained "drop-ins": open and **Run All**. Table numbers follow the paper; names below
are descriptive so they remain valid across paper versions.

| Notebook | Reproduces |
| --- | --- |
| `centerpiece_upvr_o2_dropin.ipynb` | Centerpiece four-axis stress test (main results table) + O2 logit-based control |
| `scan_hard_negative_v2_fixed.ipynb` | Hard-negative evaluation, Protocol A / Protocol B + hard-negative-collapse figure |
| `scan_table4_ci_autoattack.ipynb` | Operational TPR @ {1,5,10}% FPR (incl. AutoAttack) + operational-collapse figure |
| `scan_controlled_resolution_v2_dropin.ipynb` | Controlled-resolution sweep (HF-Energy) + resolution figure |
| `reconcile_table1_table5_dropin.ipynb` | Consistency check between the centerpiece and controlled-resolution runs |
| `table6_unified_dropin.ipynb` | Adaptive-attack table (ASR / L2 / detection AUROC) + adaptive figure |
| `scan_adaptive_v4_batched.ipynb`, `revision_adaptive_selfcontained.ipynb` | Adaptive-attack variants (batched / self-contained) |
| `scan_perfeature_ci_ablation.ipynb` | Per-feature Overall AUROC with 95% CI + complementarity heatmap |
| `scan_ci_cifar100_svhn_v2.ipynb` | Per-feature rows for CIFAR-100 and SVHN |
| `scan_ci_ablation.ipynb` | Leave-one-feature-out ablation and σ sensitivity |
| `scan_ensemble_noise_aware_ci.ipynb` | Noise-aware aggregation vs. SOTA (Mahalanobis/LID) with paired-bootstrap CIs + aggregation figure |
| `tost_equivalence_dropin.ipynb`, `tost_logistic_dropin.ipynb` | TOST statistical-equivalence tests for the aggregators |
| `scan_vit_eval_v2_direct.ipynb`, `revision_vit_selfcontained.ipynb` | Architecture generalization (ViT-B/16) |
| `recent_detectors_protocol.ipynb` | Recent frequency detectors (HFAmp / DCTspec / DFTspec) under the four-axis protocol |
| `epsilon_sweep_protocol.ipynb` | Perturbation-budget (ε) sweep, ε ∈ {1,2,4,8,16}/255 |
| `fig3_score_export_dropin.ipynb` | Score-distribution figure (standardized scores: clean / benign-noise / adversarial) |
| `revision_experiments.ipynb` | Miscellaneous revision experiments |

## Results

The `results/` directory holds cached outputs so the tables/figures can be regenerated without rerunning
the (slower) feature-extraction and attack stages:

- `centerpiece_results.json`, `centerpiece_table0.csv` — centerpiece four-axis numbers
- `controlled_resolution_v2.json` — controlled-resolution sweep
- `hard_negative_features*.pkl`, `hard_negative_summary_v2.pkl` — cached hard-negative features/summaries
- (the recent-detector and ε-sweep notebooks write `recent_detectors_results.json` and
  `epsilon_sweep_results.json` when run)

## Citation

```bibtex
@article{lee2026upscaling,
  title   = {Near-Perfect High-Frequency Adversarial Detection Is an Upscaling Artifact:
             A Four-Axis Protocol for Diagnosing Frequency Detectors},
  author  = {Lee, Jong-Ryul and Jhang, Kyoungson},
  journal = {Manuscript under review},
  year    = {2026}
}
```

## License

Released for research reproducibility. If no separate `LICENSE` file is present, the code is provided
as-is for academic use; please contact the authors for other uses.
