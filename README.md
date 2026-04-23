# EV V2G Technical Potential Dashboard

Public project overview for an interactive EV dashboard exploring
vehicle-to-grid (V2G) technical potential through machine learning,
optimisation, and visualisation.

The dashboard asks a practical modelling question:

> If a large EV fleet were available at different times of day, how much battery
> energy might be technically available for electricity-system support after
> preserving mobility needs?

This repository is a documentation and project-overview companion for the
deployed dashboard. It intentionally does not include the full implementation at
this stage.

## Live dashboard

Live dashboard: [https://uk-ev-dashboard-v2g.vercel.app/](https://uk-ev-dashboard-v2g.vercel.app/)

## At a Glance

- **Project type:** public technical dashboard and modelling overview.
- **Core idea:** estimate V2G technical potential from travel behaviour,
  availability assumptions, optimisation outputs, and interactive visualisation.
- **Serving model:** precomputed optimisation results are stored in a database
  and served through a read-only dashboard API.
- **Time handling:** selected UI timestamps are mapped to comparable
  reference-calendar buckets rather than treated as literal live readings.
- **Important caveat:** this is a model-based exploration, not a live control,
  dispatch, or commercial optimisation platform.

## What V2G Means

Vehicle-to-grid (V2G) refers to EVs interacting with the electricity system as
flexible batteries. That can include exporting electricity back to the grid,
supporting a home or local load, or adjusting charging behaviour to reduce
stress on the system.

In this project, V2G is treated as an estimation problem. The dashboard explores
technical potential under modelled assumptions; it does not send instructions to
vehicles, chargers, homes, or electricity markets.

## What the Dashboard Shows

The dashboard serves model-based half-hour estimates of EV flexibility. It is
designed to help inspect:

- Estimated EV availability at home.
- Battery energy above a retained mobility reserve.
- Scaled technical potential for a target EV population.
- Daily and weekly patterns in available flexible energy.
- Indicative household-equivalent comparisons for interpreting large energy
  values.

The values are exploratory estimates, not measured live fleet capacity.

## How to Read It

The dashboard combines machine-learning classification, availability assumptions,
optimisation outputs, and aggregation. Read the results as estimates of what a
similar period could look like under the modelled assumptions.

Two mechanics matter most:

- **Precomputed optimisation:** optimisation runs are completed beforehand,
  validated, aggregated into dashboard-ready rows, and uploaded to a database.
  The app does not solve a new optimisation problem for each user interaction.
- **Reference-calendar lookup:** selected dates and times in the UI are mapped
  into stored reference-calendar buckets so current or selected timestamps can be
  interpreted through the precomputed result set.

## Methodology

### Pipeline

```text
travel data
  -> Home vs Other classification
  -> EV availability assumptions
  -> demand, PV, tariff, and battery inputs
  -> optimisation
  -> half-hour aggregation
  -> database export
  -> interactive dashboard estimate
```

The pipeline uses a machine-learning classification model to infer when EVs are
likely to be at home, combines that with representative demand, PV, tariff, and
battery inputs, runs an optimisation model, then aggregates the outputs into
dashboard-facing metrics. The classification model was trained on cleaned
historical trip data and assessed on a held-out test split.

### Reference Calendar

The dashboard does not replay a literal historical timestamp. A selected or
current timestamp is translated into a comparable reference-calendar bucket:

1. The UI receives a selected local date and time.
2. The backend converts it into a half-hour bucket.
3. The selected date is mapped into the stored reference calendar.
4. If the exact date is unavailable, the lookup chooses the closest suitable
   reference date using comparable calendar structure where possible.
5. The dashboard returns the stored optimisation result for the scenario,
   reference date, and half-hour bucket.

The current serving approach uses a stitched August-to-July reference calendar.
This keeps the stored optimisation outputs organised as a representative annual
window while allowing the UI to interpret selected dates through a consistent
lookup layer.

The mapping preserves practical temporal structure where applicable, including
time of day, weekday/weekend context, seasonal context, UK bank-holiday-style
regimes, and Christmas/New Year periods supported by the reference data.

### Precomputed Results

The optimisation layer is not executed live in the browser or on each API
request. Scenario inputs are prepared offline, optimisation runs are performed
beforehand, outputs are aggregated into half-hour rows, and the dashboard
queries those stored rows through a read-only API.

This design keeps the public app responsive and makes displayed values traceable
to a fixed set of modelled assumptions.

## Data Sources

The project combines public and reference datasets, including:

| Source | Role |
| --- | --- |
| [National Travel Survey](https://www.gov.uk/government/collections/national-travel-survey-statistics) | Travel behaviour and mobility context |
| [My Electric Avenue](https://www.eatechnology.com/projects/my-electric-avenue/) | EV usage context |
| [London household smart-meter electricity data](https://data.london.gov.uk/dataset/smartmeter-energy-use-data-in-london-households) | Household demand profiles |
| [London photovoltaic generation data](https://data.london.gov.uk/dataset/photovoltaic--pv--solar-panel-energy-generation-data) | PV generation profiles |
| [Octopus Agile tariff context](https://octopus.energy/agile/) | Half-hourly tariff interpretation |

These sources are aligned into model-ready inputs before optimisation. No single
dataset is treated as a perfect representation of every EV, household, region,
or tariff situation.

## Assumptions and Limits

The dashboard is intentionally assumption-driven. Key interpretation points:

- Availability is based on simplified Home vs Other machine-learning
  classification, not real-time observation of every vehicle.
- Results use representative sampled EV profiles before being scaled to a target
  EV population.
- Illustrative results use a representative EV battery capacity of **37.5 kWh**,
  near the lower end of a commonly seen EV battery range rather than a claim
  about every vehicle. [Reference](https://octopusev.com/ev-hub/electric-car-batteries-explained)
- A mobility reserve is retained so the model does not treat the entire battery
  as exportable.
- Household-equivalent comparisons are indicative only and use an explicit
  typical-home daily electricity-use assumption of about **10.7 kWh/day**.
  [Reference 1](https://octopusev.com/ev-hub/what-is-v2g) [Reference 2](https://www.ofgem.gov.uk/average-gas-and-electricity-use-explained)
- Tariff, PV, demand, participation, and scaling assumptions affect the results
  and should not be treated as universal constants.

Good uses of the dashboard include comparing relative patterns, inspecting how
assumptions shape available flexibility, and understanding order-of-magnitude
technical potential. It should not be used as a live fleet measurement, a
dispatch signal, or a precise forecast.

## Technical Stack

| Area | Tools and role |
| --- | --- |
| Python / data | Python, Pandas, NumPy, validation pipelines |
| Machine learning | scikit-learn, LightGBM, MLflow experiment tracking |
| Optimisation | Pyomo, Gurobi Optimizer 12|
| Database | Supabase (PostgreSQL), SQLModel, Pydantic, psycopg |
| Backend | FastAPI, Uvicorn |
| Frontend | React, TypeScript |
| Observability | Logfire|

The deployed dashboard uses the database as the serving layer for model outputs.
The API resolves the requested scenario and reference-calendar bucket, then
returns the matching precomputed aggregate.

## Repository Scope

This repo is a public documentation and project-overview companion for the
deployed dashboard.

It includes:

- High-level project explanation.
- Methodology and reference-calendar rationale.
- Dataset, assumption, and stack summaries.
- Public-facing interpretation notes.

It does not include:

- The full source implementation.
- Deployment configuration or secrets.
- Private environment variables or database credentials.
- Raw internal notes or unpublished implementation details.

That boundary is intentional: the repo is meant to make the project legible and
credible while keeping the implementation private for now.

## Solver and Usage Note

Optimisation runs for this project were carried out using **Gurobi Optimizer**
under a **non-commercial academic licence**. This public project is shared for
informational and technical demonstration purposes, and is not presented as a
commercial optimisation service or commercial Gurobi-powered product.

## Links

- Live dashboard: [https://uk-ev-dashboard-v2g.vercel.app/](https://uk-ev-dashboard-v2g.vercel.app/)
- GitHub: [DonovanAD](https://github.com/DonovanAD)
- LinkedIn: [Donovan Aguilar](https://www.linkedin.com/in/donovanad/)
