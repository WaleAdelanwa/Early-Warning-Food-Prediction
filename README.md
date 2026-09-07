# Early-Warning Food Prediction

## Food-Supply Shortage Early-Warning Evidence Across 43 African Countries

This repository contains the supporting analytical materials for the **Talence Research Food-Supply Shortage Early-Warning Project**.

The project examines whether information available for a country and commodity in year *t* can help identify a **material decline in commodity-level food-supply calories per person in year *t + 1***. It combines historical analysis, time-ordered model development, validation, forward screening, and country-level evidence reporting across **43 African countries** and **eight selected staple commodities**.

The repository is published to make the analytical workflow **inspectable, reproducible and open to independent review**. It should be read alongside the project’s methodology and analytical reports, which remain the primary sources for interpretation, methodological decisions, limitations and reporting boundaries.

> **Important:** This project does **not** predict famine, household hunger, food affordability or every dimension of food security. A model warning is a screening signal for further investigation, not proof that a shortage has occurred or will occur.

---

## Project scope

| Component | Scope |
|---|---|
| Geographic coverage | 43 African countries represented in the FAOSTAT Food Balance Sheet release used for the project |
| Food-system scope | 8 selected staple commodities |
| Source-data period | 2010–2023 |
| Historical outcome period | 2011–2023 |
| Forward screening | 2024, using information available through 2023 |
| Unit of analysis | Country–commodity–year |
| Primary data source | FAOSTAT Food Balance Sheets |
| Geographic reference | United Nations M49 statistical geography |

### Selected commodities

The analysis covers:

- Maize and products
- Wheat and products
- Cassava and products
- Rice and products
- Sorghum and products
- Yams
- Groundnuts
- Millet and products

The commodities were selected using a combination of recent calorie importance, country relevance, production and calorie-supply coverage, and their role as continental or regional staples. No single continent-wide average determined inclusion.

---

## What the model predicts

For a given country and one of the selected commodities in year *t*, the model estimates the risk that food-supply calorie availability for that commodity will fall materially in year *t + 1*.

A historical shortage event is defined as a next-year decline that satisfies all of the following conditions:

1. current-year food-supply calories are at least **5 kcal/person/day**;
2. next-year calorie availability falls by at least **15%**; and
3. the absolute decline is at least **5 kcal/person/day**.

The final leakage-safe historical dataset contains **3,248 eligible annual transitions** and **334 observed shortage events**.

---

## Modelling approach

The modelling workflow was deliberately time-ordered so that later information could not influence earlier model development.

- **Training predictor years:** 2010–2018
- **Validation predictor years:** 2019–2020
- **Locked test predictor years:** 2021–2022
- **Outcome years:** always the following year

Logistic regression and random forest models were compared using repeated expanding-window validation. The selected random forest showed the strongest average temporal PR-AUC across the earlier out-of-time checks.

The final warning threshold was fixed at **0.13** before the locked test was opened.

### Locked-test performance

On 498 untouched test observations:

| Measure | Result |
|---|---:|
| Actual shortages | 51 |
| Warnings issued | 159 |
| Shortages detected | 33 |
| Shortages missed | 18 |
| Recall | 64.71% |
| Precision | 20.75% |
| ROC-AUC | 0.7597 |
| PR-AUC | 0.3071 |
| Brier score | 0.0823 |

The model is therefore intended as an **early-warning screening and prioritisation tool**, not an automated decision system.

---

## 2024 forward screening

After the locked evaluation was completed, an operational copy of the selected model was refitted using all labelled historical observations through 2023 outcomes.

The 2023 feature structure was reconstructed using the same definitions and ordering used during model development. Across **418,992 comparable historical feature values**, the reconstruction produced **zero mismatches**.

Of the 344 possible country–commodity observations in 2023:

- **249** were eligible and scored;
- **66** were below the 5 kcal eligibility boundary; and
- **29** had missing current calorie availability.

The unchanged operational method produced **77 formal warnings**, including **12 high warnings**.

These are **prospective model-generated estimates for 2024**, not observed 2024 outcomes.

---

## Repository structure

The repository is organised so that the computational workflow follows the same logical sequence as the methodology report.

```text
Early-Warning-Food-Prediction/
│
├── README.md
├── LICENSE
├── CITATION.cff
│
├── data/
│   ├── README.md
│   ├── processed/
│   └── outputs/
│
├── notebooks/
│   ├── 01_data_scope_and_validation.ipynb
│   ├── 02_commodity_selection.ipynb
│   ├── 03_analytical_panel.ipynb
│   ├── 04_shortage_definition.ipynb
│   ├── 05_feature_engineering.ipynb
│   ├── 06_model_development.ipynb
│   ├── 07_locked_test_evaluation.ipynb
│   ├── 08_operational_model.ipynb
│   ├── 09_2024_forward_scoring.ipynb
│   └── 10_ranking_and_reporting.ipynb
│
├── src/
│   └── reusable analytical functions
│
├── figures/
│
└── documentation/
    ├── data_dictionary.md
    └── methodology_notes.md
```

The exact repository contents may evolve as the project is documented and packaged for public use.

---

## Notebook guide

### `01_data_scope_and_validation.ipynb`
Defines the raw FAOSTAT scope, checks dimensions and flags, identifies African countries using M49 codes, and documents source coverage.

### `02_commodity_selection.ipynb`
Removes aggregate food categories and evaluates candidate commodities using calorie relevance, country relevance and data coverage.

### `03_analytical_panel.ipynb`
Builds the complete country–commodity–year panel, separates population from commodity rows, reconciles units, and preserves missingness states.

### `04_shortage_definition.ipynb`
Tests candidate shortage definitions and constructs the final leakage-safe historical outcome.

### `05_feature_engineering.ipynb`
Creates current, lagged, change, rolling, missingness, quality, ratio and per-person predictors using only information available at the prediction date.

### `06_model_development.ipynb`
Implements training-only preprocessing, expanding-window validation, logistic-regression and random-forest comparison, and threshold selection.

### `07_locked_test_evaluation.ipynb`
Evaluates the fixed model and threshold once on the untouched 2021–2022 predictor-year test period.

### `08_operational_model.ipynb`
Refits the selected operational model on all labelled historical observations after the official evaluation is complete.

### `09_2024_forward_scoring.ipynb`
Reconstructs the 2023 predictor structure, applies scoring eligibility rules, produces 2024 model probabilities and assigns evidence-review levels.

### `10_ranking_and_reporting.ipynb`
Builds the historical, recent and forward evidence views used in the continental and country reports while keeping observed and prospective evidence separate.

---

## Data and reproducibility

The project uses **FAOSTAT Food Balance Sheet** data and **United Nations M49** statistical geography.

Where source-data licensing or redistribution conditions limit republication of raw files, this repository will provide the information needed to identify and retrieve the relevant source data, together with the processing code required to reconstruct the analytical datasets.

Processed analytical files included in this repository should not be interpreted as substitutes for the original FAOSTAT source or as FAO publications.

### Source-data principles used in the project

- FAO estimated (`E`), imputed (`I`) and external-source (`X`) flags are retained and distinguished.
- Recorded zero values are not treated as missing.
- Legitimate negative accounting values are retained.
- Source-present missing values and source-absent combinations are kept as separate states.
- Population is treated as a separate country-year variable rather than a food commodity.
- FAO aggregate food groups are excluded from the final commodity selection to avoid double counting.

---

## Interpretation boundaries

The analysis measures **national apparent food availability** from Food Balance Sheets. It does not directly measure:

- household consumption;
- food affordability;
- subnational distribution;
- humanitarian severity;
- famine conditions;
- monthly or seasonal shocks; or
- whether every household had physical or economic access to food.

The model also produces false warnings and misses some historical shortage events. A no-warning result should therefore be interpreted as **lower estimated model risk**, not as evidence of safety.

Model probabilities should be considered alongside weather, crop conditions, market prices, conflict, trade disruption, policy changes and local expert knowledge before substantive decisions are made.

---

## Research outputs

This repository supports the wider Talence Research publication series:

1. **Continental Analytical Report**  
   *Eight Staple Food Systems Across 43 African Countries*

2. **Research Methodology Report**  
   *Development and Validation of a Food-Supply Shortage Early-Warning Model Across 43 African Countries*

3. **Country Evidence Profiles**  
   A consistent country-level evidence series covering the countries represented in the analytical scope.

Project page: **https://talenceresearch.com/africa-food-supply-shortage-early-warning/**

---

## Citation

If you use the repository as a whole, a suggested citation is:

> Adelanwa, A. (2026). *Early-Warning-Food-Prediction: Supporting data and analytical notebooks for the Food-Supply Shortage Early-Warning Project*. Talence Research. GitHub. https://github.com/WaleAdelanwa/Early-Warning-Food-Prediction

Readers citing a particular report should use the title, version and publication date shown on that publication.

A machine-readable `CITATION.cff` file can also be provided in this repository.

---

## Licence

The licence for original Talence Research code, documentation and derived analytical materials will be stated in the repository `LICENSE` file.

Third-party source data remain subject to the terms and conditions of their original providers. Inclusion of source references or derived analytical outputs does not transfer ownership of third-party data to Talence Research.

---

## Project status

This repository is being populated as part of the public release of the research project. Files may be added or reorganised as reproducibility documentation is completed.

Material changes to the analytical method, data scope or published outputs should be documented through repository version history and corresponding report-version updates.

---

## Contact

**Talence Research**  
https://talenceresearch.com/

For questions about the methodology, analytical materials, country evidence, or research collaboration, please use the contact details provided on the Talence Research website.

---

## Disclaimer

This is an independent Talence Research project. It uses FAOSTAT Food Balance Sheet data but is **not an FAO publication** and does not imply endorsement by the Food and Agriculture Organization of the United Nations or the United Nations.
