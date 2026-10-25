# Rerunning the modality-budget experiment in Kaggle

## Files
- `modality_budget_ieee_robustness_v2.ipynb`: updated end-to-end Kaggle notebook.
- `modality_budget_experiment_v2.py`: standalone source written by the notebook's `%%writefile` cell.

## Kaggle setup
1. Import the notebook into Kaggle and attach the BraTS 2021 Task 1 dataset used by the original notebook (`dschettler8845/brats-2021-task1`).
2. Enable a GPU accelerator (the script supports one or two GPUs; the original setup targeted two T4s) and internet for package installation.
3. Run Install, Write Script, Settings, Extract, and Data Check cells.
4. The optional smoke test now writes to a separate `mb_exp_smoke` folder. Its output is only a pipeline check; never use its metrics in the paper.
5. Run/resume the dropout and full-input training cells. They no longer automatically delete the primary output folder. If valid checkpoints are present, `--resume` continues or skips completed epochs; if not, training runs from scratch.
6. Run the evaluation cell with `--hd95`. It reuses each model's `best.pt`; it does not retrain. Evaluation computes Dice plus sampled voxel calibration metrics (ECE and Brier, including class-balanced versions) for all 15 subsets. HD95 uses each NIfTI file's voxel spacing; recognized NIfTI spatial units are converted to millimetres, and unspecified units fall back to voxel units.
7. Run the Analysis cell. It writes paired case-level comparisons (bootstrap confidence intervals and two-sided sign-flip permutation tests with Holm adjustment), per-subset safety metrics, T1ce cost sensitivity from 1 to 10 relative units, and cost-free fewest-sequence sets.
8. Check `mb_exp/analysis/` and download `results_package.zip`. Key outputs include `paired_bootstrap_comparison.csv`, `safety_metrics_dropout.csv`, `safety_metrics_full.csv`, `cost_sensitivity.csv`, `cost_sensitivity_transitions.csv`, and `cost_free_minimal_sets.csv`.

## Multi-seed extension
After the single-seed run works, set `RUN_MULTI_SEED = True` in the optional three-seed cell. It trains or resumes both regimes with seeds 42, 43, and 44 while keeping the case subset and train/validation/test split fixed at split seed 42. This is compute-intensive. Leave it disabled for the first pass; when all six result files exist, the cell creates per-seed summaries and re-runs analysis on per-case metrics averaged across seeds. These are averages of metrics across independently trained models, not an ensemble prediction.

## Publication cautions
- The T1ce cost sweep changes a **relative cost weight**, not measured scan minutes, money, or clinical risk. Replace relative weights with a protocol agreed with a radiologist/technologist before making clinical acquisition-cost claims.
- The rerun overwrites result CSVs with the new metrics. Do not copy any new numbers into the manuscript until the evaluation and analysis cells finish successfully.
- The local validation here is a syntax check and synthetic-analysis test. The BraTS GPU training/evaluation itself must be run in your Kaggle session; the full dataset and MONAI/NIfTI runtime are not available in this local environment.
# modality-budget-full-pipeline
