# Food-Supply Shortage Early-Warning Evidence Across 43 African Countries

This repository contains the data-processing workflow, analytical evidence, trained model and reporting outputs developed for the Talence Research project on food-supply shortage early warning across 43 African countries.

The project examines whether the calorie availability of a selected commodity in a country may fall materially in the following year. It combines historical analysis with a one-year-ahead machine-learning screening model. The purpose is to support earlier investigation and evidence-based discussion, not to declare that a shortage has occurred.

## Project at a glance

| Component | Coverage |
| --- | --- |
| Source | FAOSTAT Food Balance Sheets |
| Source-data period | 2010 to 2023 |
| Countries | 43 African countries with usable records in the selected release |
| Commodities | 8 selected staple-food systems |
| Complete analytical panel | 4,816 country-commodity-year observations |
| Observed outcome period | 2011 to 2023 |
| Eligible historical transitions | 3,248 |
| Observed shortage events | 334 |
| Historical event rate | 10.28% |
| Operational predictors | 129 |
| Forward screening | Possible 2024 outcomes using 2023 evidence |
| Eligible 2024 scoring observations | 249 of 344 |
| Formal warnings | 77 |
| High warnings | 12 |

## Selected commodities

| FAOSTAT item code | Commodity |
| --- | --- |
| 2511 | Wheat and products |
| 2514 | Maize and products |
| 2517 | Millet and products |
| 2518 | Sorghum and products |
| 2532 | Cassava and products |
| 2535 | Yams |
| 2552 | Groundnuts |
| 2807 | Rice and products |

The commodities were selected using several considerations together: recent population-weighted calorie importance, the number of countries in which each commodity was materially represented, production and calorie-supply coverage, and its role as a continental or regional staple. No single continent-wide average determined the selection.

## What the model predicts

The model estimates the probability that a commodity's food-supply calories per person per day will experience a substantial decline in the following year.

A next-year shortage event was recorded only when all three conditions were satisfied:

1. The starting-year supply was at least 5 kilocalories per person per day.
2. The following year's supply fell by at least 15%.
3. The absolute reduction was at least 5 kilocalories per person per day.

The minimum starting value prevents very small calorie values from producing misleading percentage declines. For example, a fall from 40 to 30 qualifies because it is a 25% decline and an absolute reduction of 10 kilocalories. A fall from 4 to 0 does not qualify because the starting value was below the minimum required for this target definition.

This is a deliberately narrow outcome. It is not a prediction of famine, household hunger, food affordability, malnutrition or every dimension of food security.

## Data preparation

The original FAOSTAT Food Balance Sheet data were filtered to the controlled African country scope and the eight selected commodities. Population records were separated from commodity records, converted from thousands of persons to persons, and merged by country and year.

A complete panel was then constructed for every expected combination of:

```text
43 countries x 8 commodities x 14 years = 4,816 observations
```

Creating the complete grid made it possible to distinguish three different conditions:

- **Recorded:** the expected FAOSTAT row existed and contained a value.
- **Source-present missing:** the expected row existed, but its value was blank.
- **Source-absent:** the country-commodity-year combination belonged in the analytical grid, but no corresponding source row was present.

Recorded zeros, missing values and negative accounting entries were not treated as interchangeable. Missing values were handled through training-only preprocessing, while source-presence indicators and relevant data-quality information were retained for interpretation and review.

## Feature construction

The model-ready dataset contains 129 predictors covering:

- current production, trade, supply, utilisation and nutrition measures;
- previous-year values;
- one-year absolute and percentage changes;
- three-year averages and variability for sufficiently covered measurements;
- population level and growth;
- supply shares, including import, production, export, food, loss and stock-variation shares;
- production, imports and exports per person;
- missingness and source-presence indicators;
- country, commodity and production-status categories.

All predictors came from the current or an earlier year. No future outcome information was used to construct a predictor row.

## Time-ordered model development

The analysis preserved the direction of time instead of randomly mixing observations from different years.

| Partition | Predictor years | Outcome years | Rows | Events | Purpose |
| --- | --- | --- | ---: | ---: | --- |
| Training | 2010 to 2018 | 2011 to 2019 | 2,250 | 231 | Fit candidate relationships |
| Validation | 2019 to 2020 | 2020 to 2021 | 500 | 52 | Compare models and warning thresholds |
| Locked test | 2021 to 2022 | 2022 to 2023 | 498 | 51 | Conduct one final evaluation |

Four expanding-window checks were also completed within the development period. Each model was trained using all available earlier years and evaluated on the immediately following year. Logistic regression and random forest models were compared, including ordinary and class-balanced variants.

The random forest was selected because it produced the strongest average Precision-Recall Area Under the Curve across the expanding-window checks. Its settings and the formal-warning threshold were fixed before the locked test was evaluated.

## Locked-test results

At the fixed probability threshold of 0.13, the selected model produced the following results on 498 untouched test observations:

| Measure | Result |
| --- | ---: |
| Observed shortages | 51 |
| Shortages detected | 33 |
| Shortages missed | 18 |
| False warnings | 126 |
| Recall | 64.71% |
| Precision | 20.75% |
| ROC-AUC | 0.7597 |
| PR-AUC | 0.3071 |
| No-skill PR-AUC reference | 0.1024 |

The model was designed as an early-warning screening tool. Its threshold gives more importance to detecting potential shortages than to avoiding every false warning. A warning should therefore trigger further review, not be treated as proof that a shortage will occur.

## 2024 forward screening

The verified feature-building rules were applied to all 344 country-commodity observations for 2023. Reconstruction was checked against historical model-ready evidence across 418,992 comparable feature values, with no mismatches found.

Of the 344 observations:

- 249 met the eligibility conditions and received a 2024 probability;
- 66 were not scored because 2023 calorie availability was below the 5-kilocalorie starting minimum;
- 29 were not scored because 2023 calorie availability was missing.

The unchanged model produced:

- 93 very-low observations below 5%;
- 79 watch observations from 5% to below 13%;
- 65 warning observations from 13% to below 30%;
- 12 high-warning observations at 30% or above.

These are model-generated screening estimates for possible 2024 outcomes. They are not observed 2024 shortage events.

## Evidence support and country comparisons

Forward estimates were reviewed against historical data coverage, missing predictors, values outside usual training ranges and the amount of country-commodity history available. Observations were classified as being within usual historical support, requiring heightened evidence review, or having limited evidence.

Country evidence is reported through separate measures rather than one combined score:

- **Historical burden:** longer-term observed shortage experience.
- **Recent burden:** observed experience during 2019 to 2023.
- **Forward screening priority:** average evidence-qualified model probability for 2024.
- **Forward peak risk:** the highest individual commodity probability within a country.

Historical and recent rankings use conservative Wilson lower confidence bounds to reduce the chance that a country is placed highly simply because it had fewer eligible observations. The separate measures should be read together because historical experience and prospective model signals answer different questions.

## Repository contents

The main project materials are stored under `food_security_predictor/`.

| Location | Contents |
| --- | --- |
| `data/raw/faostat_food_balance/` | Original FAOSTAT Food Balance Sheet files used by the project |
| `data/processed/africa_first/` | Cleaned panels, feature tables, specifications and analytical summaries |
| `models/africa_first/` | Saved operational model, evaluation records and model-comparison outputs |
| Numbered project notebooks | Data preparation, exploration, modelling, forward scoring and reporting-evidence construction |

The large source CSV is managed through Git Large File Storage. After cloning the repository, run `git lfs pull` to retrieve its full contents.

## Using the repository

Clone the repository and retrieve the Git LFS files:

```bash
git clone https://github.com/WaleAdelanwa/Early-Warning-Food-Prediction.git
cd Early-Warning-Food-Prediction
git lfs pull
```

Open the project notebooks and run them in their numbered order. The notebooks document the successive stages from source-data preparation through model development, locked testing, 2024 forward scoring and construction of reporting evidence.

Because library versions can affect results, users seeking exact computational reproduction should use the package versions recorded in the project environment or dependency files where provided. File paths may need to be adjusted when the project is run outside its original directory.

## Reports and country profiles

The complete public report series, including the continental report, integrated methodology and country profiles, is available from the Talence Research project page:

**[Read the Food-Supply Shortage Early-Warning project](https://talenceresearch.com/africa-food-supply-shortage-early-warning/)**

## Data source

The project uses Food Balance Sheet data published through the Food and Agriculture Organization of the United Nations' FAOSTAT platform:

**[FAOSTAT Food Balances](https://www.fao.org/faostat/en/#data/FBS)**

FAOSTAT remains the authoritative source for the underlying data. The cleaning, selection, feature engineering, modelling, interpretation and reporting decisions in this repository are the responsibility of the project author and publisher.

## Responsible use and limitations

- The probabilities are screening estimates, not confirmed future events.
- The model identifies patterns in national commodity-level calorie availability, not household-level food access.
- Results do not measure affordability, distribution within countries, conflict exposure, acute malnutrition or famine conditions.
- Missing or unusual inputs can reduce the strength of evidence behind an estimate.
- Historical relationships may not persist when food systems, reporting practices or external conditions change.
- Rankings are analytical comparison tools and should not be interpreted as statements that one country is generally better or worse than another.
- Decisions should combine these results with current local evidence, market information, climate conditions, conflict monitoring and expert assessment.

## Suggested citation

Adelanwa, A. (2026). *Food-Supply Shortage Early-Warning Evidence Across 43 African Countries*. Talence Research. https://talenceresearch.com/africa-food-supply-shortage-early-warning/

## Author and publisher

**Author:** Adewale Adelanwa  
**Publisher:** Talence Research  
**Publication date:** September 2026  
**Contact:** info@talenceresearch.com  
**Website:** https://talenceresearch.com/

## Copyright

Copyright © 2026 Talence Nigeria Limited. All rights reserved.

The repository is published to support transparency, scrutiny and reproducibility. Unless a separate licence states otherwise, publication of these materials does not waive copyright or grant unrestricted permission for commercial reuse.
