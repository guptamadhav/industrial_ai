# Kiln Drying ML Model — Evaluation Report

Predicting lumber kiln drying outcomes (final moisture content, drying time, energy) for **Spruce-Pine-Fir (SPF)** lumber in Drayton Valley, AB.

- **Dataset:** 1,000 synthetic runs (physics-based, USDA FPL-GTR-57)
- **Train/Test split:** 80% / 20% (stratified on NLGA compliance to preserve class distribution)

### Quick Start

| Resource | Path |
|----------|------|
| Notebook | [`Kiln_Drying.ipynb`](Kiln_Drying.ipynb) |
| Dataset | [`data_schedules/kiln_drying_dataset.csv`](data_schedules/kiln_drying_dataset.csv) |
| Generator | [`data_schedules/generate_kiln_dataset.py`](data_schedules/generate_kiln_dataset.py) |

---

## Model Architecture

| Parameter | Value |
|-----------|-------|
| Algorithm | Gradient Boosting (`sklearn`) |
| Regressor | `GradientBoostingRegressor` |
| Classifier | `GradientBoostingClassifier` |
| n_estimators | 200 |
| max_depth | 4 |
| learning_rate | 0.08 |
| subsample | 0.8 |
| min_samples_leaf | 5 |

---

## Input Features (8 total — all observable before drying starts)

| Feature | Description |
|---------|-------------|
| species_enc | Wood species (Spruce / Pine / Fir) |
| board_enc | Board size (2×4, 2×6, 2×8, 2×10, 4×4) |
| thickness_mm | Board thickness (mm) — drives L² in Fick's Law |
| density_kg_m3 | Species basic density (kg/m³) |
| fiber_saturation_pct | Fiber saturation point (%) — drying slows below this |
| initial_MC_pct | Green moisture content at batch load (%) |
| schedule_enc | Kiln schedule (T6-D3, T8-D4, T10-D5) |
| airflow_m_s | Kiln fan airspeed (m/s) |

---

## Prediction Targets (4 models)

| Target | Task | Use case |
|--------|------|----------|
| final_MC_pct | Regression | Primary target — dry grade compliance |
| total_drying_time_hr | Regression | Scheduling / throughput |
| total_energy_kWh | Regression | Cost optimization |
| nlga_compliant | Classification | Pass/fail probability |

---

## Test Set Results (n = 200 runs)

### Final MC (%)

| Metric | Value |
|--------|-------|
| R² | 0.9930 |
| MAE | 0.83 % |
| RMSE | 1.50 % |

![Final MC Prediction](plots/pred_MC.png)

---

### Drying Time (hrs)

| Metric | Value |
|--------|-------|
| R² | 0.9930 |
| MAE | 2.43 hrs |
| RMSE | 5.16 hrs |

![Drying Time Prediction](plots/pred_time.png)

---

### Energy (kWh)

| Metric | Value |
|--------|-------|
| R² | 0.9851 |
| MAE | 18.78 kWh |
| RMSE | 41.36 kWh |

![Energy Prediction](plots/pred_energy.png)

---

### NLGA Compliance (classification)

| Metric | Value |
|--------|-------|
| AUC-ROC | 0.6720 |

![NLGA Compliance Confusion Matrix](plots/confusion_matrix.png)

---

### Model Diagnostics — Residuals (Final MC)

![Residual Plot — Final MC Prediction](plots/residuals_MC.png)

---

## Feature Importance — Final MC (ranked)

| Rank | Feature | Importance |
|------|---------|------------|
| 1 | Thickness (mm) | 0.4944 |
| 2 | Board Size | 0.2290 |
| 3 | Initial MC (%) | 0.1693 |
| 4 | Kiln Schedule | 0.0735 |
| 5 | Airflow (m/s) | 0.0128 |
| 6 | Density (kg/m³) | 0.0118 |
| 7 | Fiber Saturation (%) | 0.0077 |
| 8 | Species | 0.0015 |

---
Synthetic Dataset created using Fick's Law Drying physics + Siau EMC + Arrhenius diffusion, contact me @madhav75gupta@gmail.com for the script
---
## References

- **Physics:** USDA [FPL-GTR-57](https://www.fpl.fs.usda.gov/documnts/fplgtr/fplgtr57.pdf) (kiln schedules), Siau (1984), Fick's Law
- **Methodology:** Rahimi et al. (2023) — *Polymers* 15(4):792, UBC/FPInnovations
- **Grading:** NLGA Standard Grading Rules for Canadian Lumber
