# Fazenda NSAAB — Coffee Agronomic Q&A Dataset

## Overview

This dataset contains **90 question-and-answer pairs** focused on agronomic decision-making for a coffee plantation (Fazenda NSAAB). Each entry simulates a real-world query an agronomist or farm manager might ask, combined with a data-driven recommendation based on weather and soil sensor readings.

The questions cover topics such as:

- Drip irrigation scheduling and water deficit/excess risk
- Flowering, pollination, and fruit set conditions
- Wind, gusts, and spray-drift risk for agricultural applications
- Heat stress, solar radiation, and evapotranspiration demand
- Atmospheric pressure and short-term weather instability
- Fungal disease risk from humidity and leaf wetness
- Soil compaction and field trafficability after rain
- Harvest timing, transport, and natural bean drying

## File

`db_90_questoes_en.json` — the full dataset, in English.

## Data Structure

Each item in the JSON array represents one Q&A record with the following fields:

| Field | Description |
|---|---|
| `number` | Sequential ID (1–90) |
| `question` | The agronomic question |
| `answer` | The recommendation, including the specific sensor/forecast values used and the agronomic rationale |
| `technical_rationale_for_recommendation` | Why this type of recommendation is made (general principle behind the decision logic) |
| `technical_basis_for_recommendation` | Literature/sources the recommendation logic is grounded in |
| `complete_answer` | Concatenation of `answer` + `technical_rationale_for_recommendation` + `technical_basis_for_recommendation`, for convenience |
