# Destination-port sensitivity on CICIDS2017

A small Random Forest sensitivity experiment inspired by J.-B. Altidor and C. Talhi, “Enhancing Port Scan and DDoS Attack Detection using Genetic and Machine Learning Algorithms,” CIoT 2024. DOI: [10.1109/CIoT63799.2024.10757005](https://doi.org/10.1109/CIoT63799.2024.10757005).

This experiment tests whether models with selected small feature subsets respond differently to destination-port changes. It does **not** reproduce the paper's genetic algorithm or its final feature subset.

## Data and method

Obtain the two Friday-afternoon MachineLearningCSV files from [CICIDS2017 (UNB)](https://www.unb.ca/cic/datasets/ids-2017.html):

- `Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv`
- `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv`

After removing missing/nonfinite and duplicate rows, 435,336 flows remain. Removing 10 constant and 8 identical predictor columns leaves 60 of the original 78 predictors. Sample 40,000 benign, 20,000 PortScan and 20,000 DDoS flows; split 70/30 with stratification. Random Forest uses 100 trees, seed 42 and otherwise default parameters.

In the test set only, replace destination ports 80, 443 and 53 with uniform random integers from 1024 to 65535, using RNG seed 0. This changes 14,280 of 24,000 main test flows, including benign flows. All other features and labels stay fixed.

The small subset contains the 12 highest-ranked training-model features plus destination port, which originally ranked 21st. Feature importance is impurity based, not GA based.

## Recorded results

| Experiment | Accuracy (%) | Macro F1 (%) |
|---|---:|---:|
| A: all 60 features | 99.95 | 99.95 |
| B: same model, test ports changed | 99.89 | 99.89 |
| C: retrained without port (59 features) | 99.95 | 99.95 |
| D1: 12 highest-ranked features + port | 99.96 | 99.96 |
| D2: same small model, test ports changed | 90.83 | 89.73 |
| D3: retrained on 12 features without port | 99.95 | 99.95 |

In D2, **2,194/6,000 DDoS flows (36.57%) are predicted benign**; none are predicted PortScan. Across seeds 42, 1 and 7, D2 accuracy ranges from 90.68% to 90.92%, and DDoS misclassification from 36.1% to 37.2%. These seeds jointly change sampling, splitting, model randomness and subset selection; the port RNG seed stays fixed.

For ten random subsets containing 12 non-port features plus port, perturbed accuracy ranges from 74.95% to 90.85%; 36.6–100.0% of DDoS flows are predicted as a class other than DDoS. This statistic is not specifically a benign prediction rate. Supplementary CSVs transcribe the saved notebook tables; the original code displays them but does not export them.

The full model is relatively insensitive to this perturbation in this run; the tested small port-containing subsets are more sensitive. Removing port preserves similar rounded original-test accuracy. This does not establish a universal relationship between feature count and robustness.

## Limitations

- One source dataset, two Friday-afternoon files and one classifier; random flow splits do not measure transfer to new days or networks.
- Constant/duplicate column filtering uses the combined data before splitting. A strict follow-up should fit preprocessing on training data only.
- Exact row deduplication precedes column filtering; reduced projections can still have identical or near-identical predictors across splits.
- Port edits are a tabular perturbation, not regenerated traffic or demonstrated real-world evasion.
- Every tested 13-feature subset intentionally includes port. The paper's GA-selected subset is not known from this experiment.
- Three seeds reuse overlapping source data, and ten subsets are evaluated on one test split. These are sensitivity checks, not independent validation datasets.
- Package versions from the original execution were not recorded; requirements list dependencies without claiming an exact environment.

## Run locally

```bash
python -m pip install -r requirements.txt
python -m notebook
```

Put the two raw CSVs beside `port_shortcut_experiment.ipynb` (or in `data/`), open the notebook and run all cells. The main summary is written to `results.csv`. The notebook includes saved outputs. Raw CSVs and the paper PDF are not included in this repository.
