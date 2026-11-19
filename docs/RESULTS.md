# Selected OpenTherm results

This page presents a compact subset of the S1–S5 results. It is intended to show the benchmark's main behavior, not reproduce every table from the paper. S1 retains its source-task protocols; S2–S5 use the shared OpenTherm splits, preprocessing, labels, and metrics. All values are in kelvin and are means over three runs.

> **Release status:** S2–S5 data, evaluation code, and [checkpoints](https://drive.google.com/file/d/1dXtFyhiSo68NFZ-HwW6a3gmjdBfaJ_1-/view?usp=sharing) are released, and the S1 datasets are now released as well. S1 results are recorded, while its one-command evaluator is **on the way**.

## At a glance

| Track | Evaluation setting | Best method | MAE ↓ | Second-best method | MAE ↓ |
|---|---|---|---:|---|---:|
| S1 · Fixed design | fixed-design source tasks | **Therm-FM** | **0.004–0.049**¹ | U-FNO / SAU-FNO (task-dependent) | <u>0.011–0.144 / 0.013–0.162</u>² |
| S2 · Layout | placement and orientation, Cases 1–10 | **Therm-FM** | **0.248** | U-FNO | <u>0.326</u> |
| S3 · Material | S2 + filler conductivity | **U-FNO** | **0.493** | Therm-FM | <u>0.517</u> |
| S4 · Cooling | S3 + ambient temperature and convection | **Therm-FM** | **0.585** | U-FNO | <u>0.724</u> |
| S5 zero-shot | five unseen packages, Cases 11–15 | **Therm-FM** | **8.00** | U-Net | <u>17.37</u> |
| S5 10-shot | ten labeled fields per unseen package | **Therm-FM** | **1.71** | U-Net | <u>1.80</u>³ |

¹ S1 contains eleven task-specific source protocols and therefore has no single pooled score; the paper reports MAE ranges by task group. ² The S1 runner-up varies by task between U-FNO and SAU-FNO; their MAE ranges are listed in that order. ³ Measured on the fixed 2,500-sample adaptation holdout; S5 zero-shot uses all 5,000 samples. Therm-FM-T/B/L count as one method family, represented by their best score on each track. Best results are bold; second-best results are underlined.

The best within-family MAE worsens gradually from 0.248 K on S2 to 0.585 K on S4. S5 is qualitatively different: moving to unseen packages raises the best MAE to 8.00 K, 13.7× the best S4 result. This is the benchmark's main distinction between generalization within known package families and transfer to new packages.

## S1: fixed-design snapshot

S1 collects eleven established Alpha EV6 and industrial tasks without relabeling, resizing, or merging them into a new distribution. The lineage follows [ARO](https://github.com/Mia-WMY/ARO), [SAU-FNO](https://doi.org/10.1109/DAC63849.2025.11132988), and [Therm-FM](https://arxiv.org/abs/2605.22663). The table reports task-level MAE ranges at each task's finest available resolution; errors are not pooled across systems or resolutions. Therm-FM-L results are taken from the source study; the remaining rows are benchmark reruns.

| Method | EV6 steady MAE | EV6 transient MAE | Industrial MAE |
|---|---:|---:|---:|
| FNO | 0.063–0.164 | 0.034–0.163 | 0.017–0.018 |
| U-FNO | 0.046–0.126 | 0.011–0.144 | 0.013 |
| SAU-FNO | 0.041–0.162 | 0.013–0.149 | 0.020–0.026 |
| DeepOHeat | 0.745–2.585 | 0.315–1.820 | 0.039–0.044 |
| **Therm-FM-L (629M)** | **0.012–0.049** | **0.004–0.030** | **0.008** |

FNO, U-FNO, and SAU-FNO stay below 0.17 K on every reported S1 task, so fixed-design power/time prediction is already a strong and relatively mature setting. S2–S5 therefore focus on what happens when designs vary within a package family and when the package itself changes. S1 data remain subject to their upstream licenses and citation requirements.

## S2–S4: within-family DSE

Only the best and runner-up method families are shown here; Therm-FM-T/B/L count as one family. S2–S4 are independently generated, rather than sample-wise paired perturbations, so the trend measures increasing distributional difficulty rather than a strict one-variable ablation.

| Track | Best method | MAE | Runner-up | MAE | Best RMSE | Best peak ΔT |
|---|---|---:|---|---:|---:|---:|
| S2 · Layout | Therm-FM | **0.2477** | U-FNO | <u>0.3265</u> | 0.3536 (Therm-FM) | 0.2164 (Therm-FM) |
| S3 · Material | U-FNO | **0.4928** | Therm-FM | <u>0.5165</u> | 0.6700 (U-FNO) | 0.3312 (U-FNO) |
| S4 · Cooling | Therm-FM | **0.5852** | U-FNO | <u>0.7237</u> | 0.8113 (Therm-FM) | 0.3696 (Therm-FM) |

All eight configurations show increasing MAE from S2 to S4, and every model except DeepOHeat keeps S4 MAE below 1 K. Therm-FM-L leads all reported metrics on S2 and S4, whereas U-FNO leads all six metrics on S3, including 0.493 K MAE versus 0.527 K for Therm-FM-L. With about 5.1M parameters, U-FNO stays within 1.26× of the best RMSE on every within-family scope. Rankings also depend on the metric: on S4, Therm-FM-B has lower MAE than SAU-FNO (0.618 vs. 0.739 K), whereas SAU-FNO has lower MaxAE (4.137 vs. 4.260 K).

## S5: cross-package OOD and adaptation

Frozen S4 checkpoints are evaluated on unseen Cases 11–15 without target labels or updated normalization. Few-shot results use `K` labeled samples per unseen package and a separate fixed 2,500-sample holdout.

| Setting | Best method | MAE ↓ | Second-best method | MAE ↓ | Interpretation |
|---|---|---:|---|---:|---|
| S4 within-family reference | Therm-FM | **0.585** | U-FNO | <u>0.724</u> | known package families |
| S5 zero-shot (5,000 samples) | Therm-FM | **8.003** | U-Net | <u>17.374</u> | transfer to unseen packages |
| S5, K=10 | Therm-FM | **1.71** | U-Net | <u>1.80</u> | 50 target labels in total |
| S5, K=50 | U-Net | **1.01** | Therm-FM | <u>1.02</u> | Therm-FM-T and -L within 0.03 K |
| S5, K=100 | Therm-FM | **0.85** | U-FNO | <u>0.86</u> | most of the gap has been recovered |
| S5, K=500 | U-FNO | **0.59** | Therm-FM | <u>0.62</u> | higher-budget target calibration |

From S4 to S5 the best MAE rises 13.7×; for Therm-FM-L alone it rises from 0.585 K to 14.633 K, about 25× its S4 value. The smallest variant, Therm-FM-T, leads all reported S5 zero-shot metrics, reversing the S4 ordering of the Therm-FM variants. Ten labels per package reduce MAE by 76–93% relative to each model's own frozen score on the same holdout, with Therm-FM-T reaching 1.71 K. Adaptation is still reported separately: a model that adapts well after seeing target labels has not solved zero-shot cross-package generalization.

### Different unseen packages favor different models

The paper reports per-package zero-shot errors as RMSE on the full S5 test set (1,000 samples per package).

| Held-out package | Best method | RMSE ↓ | Second-best method | RMSE ↓ |
|---|---|---:|---|---:|
| Case 11 · chiplet count | Therm-FM | **4.48** | DeepOHeat | <u>4.79</u> |
| Case 12 · power density | U-Net | **3.25** | SAU-FNO | <u>4.71</u> |
| Case 13 · size heterogeneity | Therm-FM | **9.03** | DeepOHeat | <u>10.01</u> |
| Case 14 · power concentration | Therm-FM | **7.50** | SAU-FNO | <u>9.88</u> |
| Case 15 · package utilization | Therm-FM | **6.33** | U-Net | <u>17.05</u> |

No model wins every package. Therm-FM-T has the lowest aggregate zero-shot error because its per-package RMSE stays within 6.33–11.17 K, while other models combine narrow strengths with severe package-specific failures. Reporting per-package results is therefore more informative than publishing only one pooled score.

## Takeaways

- Low absolute errors remain attainable as tasks expand from fixed designs to within-family DSE.
- Cross-package transfer is a much greater challenge, and larger models do not consistently generalize better.
- Limited target supervision substantially mitigates the gap, but should complement—not replace—an explicit zero-shot report.

Commands, metric definitions, and reproduction boundaries are documented in [REPRODUCE.md](REPRODUCE.md). The selected S2–S5 result records are covered by the repository's [data license](../LICENSE-DATA).
