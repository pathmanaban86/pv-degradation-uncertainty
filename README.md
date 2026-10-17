# pv-degradation-uncertainty

Code for the article *Uncertainty issues in the assessment of photovoltaic degradation rates from longitudinal
indoor flash measurements* (submitted to *Measurement*). <!-- TODO: add DOI after acceptance -->

The code evaluates the uncertainty of module degradation rates from the Sandia PV-IV-EL dataset following the
GUM and its Supplement 1 (Monte Carlo method). It separates three scenarios — A (as released), B (corrected for
the simulator calibration current) and C (upper bound for test-contact resistance) — attributes the variance to
its sources, and tests whether an EL series-resistance metric tracks the series-resistance growth seen in the
I–V curves.

## Repository layout

```
src/
  01_el_pipeline.py         EL images -> cell features, first-vs-last visit pairs, decision report
  02_figures_tables.py      I-V extraction, uncertainty budget, Monte Carlo, Figs 2-13, Tables 1-7
  03_sensitivity.py         robustness checks (Table 7, Secs 3.2, 4.5, 4.6, 4.8, 4.9)
  04_fig1_workflow.py       Fig. 1 as an editable draw.io file + PDF/PNG preview
  05_compose_fig3_fig13.py  optional: re-composes Figs 3 and 13 with shorter panel titles
notebooks/run_in_colab.ipynb  runs steps 01-04 in Google Colab
figures/                    Fig. 1 (.drawio, editable in diagrams.net) and its preview
results/                    derived results as published (small CSV/JSON files, no raw data)
data/README.md              how to obtain the dataset
```

## Data

Download `AnonDB.csv`, `IV.zip`, `RefIV.zip` and `EL.zip` from OEDI submission 8378
(https://data.openei.org/submissions/8378) into `data/` or any folder. See `data/README.md`.
The raw data are not included in this repository.

## Installation

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

## Running

All scripts read the data folder from the `PV_BASE` environment variable and write their outputs into it.

```bash
export PV_BASE=$PWD/data
python src/01_el_pipeline.py                 # EL processing (longest step); try --limit 10 --qc-every 1 first
python src/02_figures_tables.py              # figures and tables; PV_NMC=4000 by default
python src/03_sensitivity.py                 # needs the outputs of 01 and 02
python src/04_fig1_workflow.py figures       # Fig. 1
```

In Google Colab, open `notebooks/run_in_colab.ipynb`, set the Drive folder and the repository URL, and run all
cells. Without `PV_BASE`, the scripts look for a Drive folder matching `PV_DRIVE_GLOB`
(default `/content/drive/MyDrive/Research/Solar*`).

`02_figures_tables.py` also runs without `el_out/el_visit_pairs.csv`; manuscript Figs 12 and 13 are then skipped.

## Reproducibility

- The Monte Carlo generator is seeded (`numpy.random.default_rng(20261012)`, N = 4000 trials).
- Running `02` and `03` on the released data reproduces the files in `results/` exactly
  (all numeric differences 0 for Tables T1–T7, `sens.json` and `per_model_sens.csv`).
- Models 32-2 and 38-4 are excluded because their reference I–V curves are anomalous (power gain together with an
  R_s drop relative to the reference).

## Correspondence between output files and the manuscript

| Script output | Manuscript |
| --- | --- |
| `figures/Fig1_test_procedure_analysis_chain.drawio` | Fig. 1 |
| `Fig01_dataset` … `Fig12_EL_case_studies` | Figs 2–13 (script number + 1) |
| `Fig00_workflow` | earlier draft of Fig. 1, replaced by the draw.io version |
| `T1_dataset` | Table 1 |
| `T6_EL_audit` | Table 2 |
| `T2_budget` | Table 3 |
| `T5_extraction` | Table 4 |
| `T3_rates` | Table 5 |
| `T4_shares` | Table 6 |
| `sens.json` (`pooled_sens`) | Table 7 |
| `T7_contact_correction_validation`, `per_model_sens.csv` | Secs 4.6 and 4.9 |

Table 8 (comparison with the literature) has no code.

## Key quantities

| Symbol | Meaning | Value / setting |
| --- | --- | --- |
| c | calibration-current ratio `RefA / RefRefA` | per measurement |
| β | calibration exponent in P* = P · c^(−β) | 0.195, u(β) = 0.059 |
| f | fraction of the R_s increase attributed to test contact (scenario C) | 1 (upper bound) |
| R_d | degradation rate, least squares through the origin | %/yr |
| s_r | EL series-resistance metric; detection limit 2σ_Δ | σ_Δ = 0.017 |

## Citation

See `CITATION.cff`. Please also cite the dataset (OEDI submission 8378).

## License

MIT (see `LICENSE`).
