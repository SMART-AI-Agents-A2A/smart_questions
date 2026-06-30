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
- Harvest timing, transport, and natural coffee bean drying

## File

`db.json` — the full dataset, in English.

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


## Prompt Engineering: Input/Output Format
 
Each record in this dataset follows a consistent input → output pattern, designed to be used as fine-tuning or few-shot data for an LLM-based agronomic assistant. The pattern below explains *why* the format looks the way it does, using item `1` as a worked example.
 
### The underlying task
 
The implicit system prompt behind every entry is something like:
 
> *You are an agronomic decision-support assistant for Fazenda NSAAB, a coffee plantation. You receive a question from a farm manager or agronomist about a specific field condition (irrigation, spraying, flowering, harvest, etc.). You have access to live sensor data (InfluxDB) and weather forecast data (OpenWeather). Answer with a concrete, actionable recommendation grounded in the actual data values, not a generic statement. Justify your recommendation with both the underlying agronomic principle and the literature it is based on.*
 
This is why every `answer` is data-grounded (specific numbers, not vague advice), conditional (tells the user what to watch for, not just what to do right now), and traceable (cites where the data came from).
 
### Input (`question`)
 
```
"Given that the coffee plants at Fazenda NSAAB are in the flowering or
pre-flowering period, do today's weather conditions favor pollination
and flower set, taking into account wind speed, wind direction, gusts,
air temperature, relative humidity, and the chance of rain in the
coming hours?"
```
 
Three things are intentionally embedded in the question itself, by design:
 
1. **Context anchor** — "Fazenda NSAAB" and the phenological stage ("flowering or pre-flowering") ground the question in a specific farm and a specific moment in the crop cycle, rather than a generic "is it good weather" question.
2. **Explicit variable list** — the question names the exact variables the answer must address (wind speed, direction, gusts, temperature, humidity, rain chance). This is a prompting technique: it constrains the model's answer space and makes the expected output auditable — a grader (human or automated) can check whether each named variable was actually used in the answer.
3. **Decision framing** — the question is phrased as a yes/no-leaning judgment call ("do conditions favor..."), forcing the model to commit to a position rather than hedge indefinitely.
### Output (`answer`)
 
```
"Today's conditions are partially favorable for pollination and flower
set. The recommendation is to limit activity to observing the bloom and
avoid any operation that could cause mechanical stress to the plants
during gust periods. Average wind speed is 8 km/h, with a predominant
easterly direction, gusts of 17 km/h, air temperature of 26.8 °C,
relative humidity of 72%, and a low chance of rain in the next 3 hours.
[...] Data source: InfluxDB and OpenWeather. Measurement time: 09:00."
```
 
The answer follows a fixed internal structure, repeated across all 90 items:
 
1. **Verdict first** — "partially favorable" — a direct answer to the question, stated in the first sentence. This matters for downstream use (e.g. a dashboard or chatbot needs a quick headline before the detail).
2. **Concrete action** — what to do (or not do) right now, stated before the supporting data. This keeps the response usable even if the user only reads the first two sentences.
3. **Supporting data dump** — every variable named in the question is restated with its actual value (8 km/h, 17 km/h gusts, 26.8 °C, 72%, low rain chance). This is what makes the answer auditable: a reviewer can verify the verdict against the numbers without needing external data.
4. **Conditional close** — a forward-looking instruction ("keep hourly monitoring of wind and rain before any management activity"), since field conditions change and a single snapshot reading shouldn't be treated as a standing order.
5. **Provenance footer** — "Data source: InfluxDB and OpenWeather. Measurement time: 09:00." This is a deliberate transparency pattern: every claim in the answer can be traced back to which system produced it and when, which is important for an agronomic tool where a wrong recommendation has real crop-damage consequences.
### Output (`technical_rationale_for_recommendation`)
 
```
"The recommendation prioritizes water stability and stress reduction
during the reproductive phase. During flowering, pre-flowering, fruit
set, and newly opened flowers, the crop responds sensitively to abrupt
swings in rainfall, wind, temperature, and soil water availability.
For this reason, even when current values appear acceptable, the
response should not authorize management activities unconditionally;
it should indicate caution, observation, and reassessment based on
wind, rain, and soil moisture data."
```
 
This field exists separately from `answer` because it answers a different question: not *"what should I do"* but *"why does the model reason this way for this category of scenario."* It is one of only 10 reusable rationale templates across the dataset (each tied to a recurring decision pattern — flowering/reproductive stage, drip irrigation, spray application, heat stress, etc.) — see the note below.
 
### Output (`technical_basis_for_recommendation`)
 
```
"Based on Meireles et al. (Coffee Plant Phenology), who describe the
phenological stages of the coffee plant and the importance of
agrometeorological monitoring; on Torres et al. (Coffea arabica L.),
who point out the sensitivity of phenological stages to temperature
and precipitation; and on Cararo and Souza Dias (Irrigation in Coffee
Plantations), who relate water requirements to climate, soil, plant,
and management."
```
 
This field grounds the rationale in named agronomic literature, rather than letting the model assert a principle without backing. It is the citation layer of the answer — separated from the rationale so that the "why" (general decision logic) and the "where this comes from" (sources) can be inspected independently.
 
### Output (`complete_answer`)
 
A straight concatenation of `answer` + `technical_rationale_for_recommendation` + `technical_basis_for_recommendation`, provided so that a single field contains everything needed to display or evaluate the full response without having to reassemble it from the other three fields at inference/serving time.
 
### Why this 5-field structure, in summary
 
| Field | Answers the question | Purpose |
|---|---|---|
| `question` | What is being asked? | Defines scope and the variables the answer must use |
| `answer` | What should be done, and based on what data? | The actionable, auditable recommendation |
| `technical_rationale_for_recommendation` | Why this kind of decision logic? | Generalizable reasoning, reusable across similar scenarios |
| `technical_basis_for_recommendation` | Where does this reasoning come from? | Literature grounding / citation |
| `complete_answer` | All of the above, together | Convenience field for direct display or single-field evaluation |
 
This separation of concerns (data-grounded answer vs. general rationale vs. literature citation) is what makes the dataset suitable for training or evaluating a model that needs to be both useful in the moment and explainable after the fact.
 