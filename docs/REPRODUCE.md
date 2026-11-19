# Reproducing OpenTherm

OpenTherm compares models only after fixing the data, split, labels, physical channels, and metric code. Full-training runs use 100 epochs for every baseline, while architecture-specific optimization recipes remain architecture-specific and are recorded in [`exp/exp_basic.py`](../exp/exp_basic.py) and the Therm-FM YAML files.

## Before running

1. Create the [frozen environment](INSTALL.md).
2. Place the [datasets and checkpoints](DATASETS.md) at the repository root.
3. Run `python script/smoke_test.py`.

## Evaluation tracks

| Track | Train support | Test support | Target labels used at test time |
|---|---|---|---:|
| S1 · Fixed design | original source-task protocols | same fixed physical designs | source-defined |
| S2 · Layout | Cases 1–10, placement and orientation | same package families | 0 |
| S3 · Material | S2 + filler conductivity | same package families | 0 |
| S4 · Cooling | S3 + ambient temperature and convection | same package families | 0 |
| S5 zero-shot | frozen S4 model | unseen packages, Cases 11–15 | 0 |
| S5 few-shot | frozen S4 model + adaptation pool | held-out samples from Cases 11–15 | K per package |

S2–S4 are independently generated rather than sample-wise paired perturbations. Their comparison measures a progressive change in distributional difficulty, not a strict causal ablation of one variable.

The commands below currently cover S2–S5. The [S1–S5 result record](RESULTS.md) is available and the S1 datasets are released, while S1's one-command evaluator is on the way.

## One model

```bash
# Evaluate a released checkpoint
python run.py --model UFNO --data level2 --task test

# Train with the registered benchmark recipe, then evaluate
python run.py --model UFNO --data level2 --task train

# Cross-package OOD: automatically use the S4 checkpoint
python run.py --model UFNO --data level5 --task test

# Adapt with 10 labeled samples per unseen package
python run.py --model UFNO --data level5 --task finetune --shots 10
```

Accepted model names are:

```text
FNO  UFNO  SAUFNO  UNet  DeepONet  ThermFM-T  ThermFM-B  ThermFM-L
```

## All baselines

```bash
# S2–S5 evaluation and summary
bash script/test_all.sh

# Registered S2/S3/S4 training recipes
bash script/train_all.sh

# S5 K-shot curves
bash script/finetune_all.sh
```

Model-specific wrappers also accept a scope token or shot list:

```bash
bash script/UFNO/train.sh level2
bash script/UFNO/test.sh level4 level5
bash script/UFNO/finetune.sh 10 50 100 250 500
```

Run Therm-FM on one GPU. For example:

```bash
python run.py \
  --model ThermFM-T \
  --data level4 \
  --task train \
  --gpus 0
```

The released Therm-FM recipe is deliberately single-GPU: training uses AdamW with learning rate `1.5e-4`, embedding/recovery and time-embedding learning rates `1.5e-3`, batch size 40, cosine scheduling with zero warmup, and validation-best checkpoint selection. Data-parallel training on four GPUs multiplies the effective global batch size and does not reproduce the published results. S5 adaptation uses `1.5e-5`, one tenth of the Therm-FM training rate.

## Selected reference results

These paper values record the strongest method family in each track (MAE in kelvin):

| Track | Best method | MAE ↓ | Second-best method | MAE ↓ | Interpretation |
|---|---|---:|---|---:|---|
| S1 | **Therm-FM** | **0.004–0.049** | U-FNO / SAU-FNO (task-dependent) | <u>0.011–0.144 / 0.013–0.162</u>¹ | task-group source results; not pooled |
| S2 | **Therm-FM** | **0.248** | U-FNO | <u>0.326</u> | layout variation is learnable within package families |
| S3 | **U-FNO** | **0.493** | Therm-FM | <u>0.517</u> | adding filler-conductivity variation causes moderate degradation |
| S4 | **Therm-FM** | **0.585** | U-FNO | <u>0.724</u> | joint layout, material, and cooling variation remains tractable |
| S5 zero-shot | **Therm-FM** | **8.00** | U-Net | <u>17.37</u> | unseen packages raise the best MAE 13.7× |
| S5 10-shot | **Therm-FM** | **1.71** | U-Net | <u>1.80</u>² | 50 target labels total recover much of the gap |

¹ The S1 runner-up varies by task between U-FNO and SAU-FNO; their MAE ranges are listed in that order. ² Measured on the fixed 2,500-sample adaptation holdout; S5 zero-shot uses all 5,000 samples. Therm-FM-T/B/L count as one method family, represented by their best score on each track. Best results are bold; second-best results are underlined.

S1 preserves eleven source protocols; see [RESULTS.md](RESULTS.md) for the source-suite summary and expanded S2–S5 results. Small last-digit differences in S2–S5 can arise from GPU kernels and execution environments. A valid reproduction should preserve the split, model recipe, normalization mode, and metric implementation before attributing differences to a method.

## Metrics

All baselines call [`utils/metrics.py`](../utils/metrics.py). Do not replace it with an “equivalent” local implementation when reporting OpenTherm numbers.

| Reported metric | Meaning |
|---|---|
| RMSE ↓ | per-sample field RMSE, then averaged |
| MAE ↓ | per-sample field MAE, then averaged |
| R² ↑ | pooled over every test cell |
| MaxAE ↓ | worst-cell absolute error in a field, then averaged |
| Peak-temperature error ↓ | absolute difference between predicted and true maximum temperatures |
| Top-50 MAE ↓ | MAE at the 50 hottest ground-truth cells |

MaxAE and peak-temperature error are different: the largest pointwise error may occur away from the true or predicted hotspot.

Summarize saved results with:

```bash
python utils/summarize.py
python utils/summarize.py level2 level5
```

Pass `--output path/to/result.json` to `run.py` when integrating with another experiment manager.

## Reproduction boundaries

- Every released baseline uses temperature-field labels; this is a supervised benchmark, not a PINN-training comparison.
- S5 zero-shot evaluation does not update model weights or normalization statistics.
- Therm-FM starts from Poseidon for training; evaluating a released checkpoint does not require Poseidon.
- `--epochs`, `--batch_size`, `--lr`, `--per_channel_norm 0`, or multi-GPU Therm-FM training create a new experimental recipe and should not be presented as the released reproduction.
- Report dataset generation, conversion, and reference-solver time separately from model training/inference time.

## Output locations

```text
checkpoints/{data}_{model}/...                 trained models
checkpoints/fewshot/{data}_{model}_k{K}.json  adaptation curves
results/...                                   evaluation JSON/plots when requested
```

Few-shot JSON files include the frozen baseline, per-epoch curve, and final metrics. This makes the adaptation result auditable without rerunning every epoch.
