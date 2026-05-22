# Energy

Energy is a single-page client-side application (no server required) for tracking and visualising household energy consumption over time.
All input data and display configuration live directly in the HTML file — no build step or deployment needed.

## Features

- **Automatic computation** from monthly meter readings: electricity (off-peak / peak hours), domestic hot water, electric heating, wood, sunshine hours, heating degree days (DJU).
- **Five metric types**:
  - `value` — a measured quantity for the period (e.g. DJU, sunshine hours, wood in kg).
  - `index` — a cumulative counter whose period value is derived by subtracting the previous reading (e.g. electricity meter indexes).
  - `calc` — a value derived from other metrics via an arithmetic formula (e.g. `total_elec = total_hc + total_hp`).
  - `reg` — a forecast computed by multiple linear regression fitted on the last complete year (e.g. heating forecast based on DJU and sunshine).
  - `bill` — an occasional bill amount that is automatically spread across previous records up to the previous bill entry.
- **Time-varying constants**: values such as floor area or number of occupants can change over time; they are propagated forward to all subsequent records automatically.
- **Monthly and yearly aggregation**, with `calc` metrics recomputed at the year level to ensure ratio consistency.
- **Charts** (via Chart.js) and **tables** fully driven by configuration — adding a new graph or column requires only a config entry.
- **Cell gradients** in tables to highlight trends at a glance.
- **Automated tests** run on every page load to catch regressions early.

## Entering data

### Records

Data is entered in the `records` JavaScript array at the top of the file. Each entry is an object:

```javascript
{date: "YYYY-MM-DD", total_hc: 31646, total_hp: 32991, ..., dju: 237.9, enso: 179.7}
```

- **`date`** — reading date in `YYYY-MM-DD` format. Records are listed from most recent to oldest.
- Other properties correspond to the keys defined in `config.values`.
- An object **without a `date`** is a constants block (e.g. `{surface: 135, pers: 4}`). Its values are propagated to all older records until redefined.

**Index reset** — if a meter was replaced or reset, add a `<key>_init` property on that record to provide the reference value to subtract from:
```javascript
{date: "2018-03-01", total_hc: 76, total_hc_init: -848+76, ...}
```

**Bills** — for `bill` metrics, add the source property on the corresponding record. The amount is automatically distributed over all preceding records back to the previous bill:
```javascript
{date: "2025-09-01", ..., facture_elec: 1744}
```

### Configuration

The `config` object immediately follows `records` and has three sections: `values`, `graphs` and `tables`.

#### `values` — metric definitions

Each key in `values` maps to a metric. The key must match the property name used in `records`.

| Property | Required | Description |
|---|---|---|
| `name` | yes | Display label used in charts and table headers |
| `unit` | yes | Unit string displayed alongside values (e.g. `"kWh"`, `"€"`, `"%"`) |
| `type` | yes | One of `value`, `index`, `calc`, `reg`, `bill` (see below) |
| `formula` | for `calc` / `reg` | Arithmetic expression or regression formula (see below) |
| `from` | for `bill` | Name of the record property holding the bill amount |
| `color` | no | CSS color string used in charts and legend swatches |
| `dash` | no | `true` to render the series as a dashed line in charts |
| `positive` | for `reg` | `true` to clamp regression output to zero when the result is negative |

**`calc` formula** — a standard arithmetic expression over other metric keys and numeric literals. Supports `+`, `-`, `*`, `/` and parentheses:
```javascript
divers_elec: {name: "Misc.", unit: "kWh", type: "calc", formula: "total_elec - chauf_elec - ecs_hc"}
chauf_surface: {name: "Heating/area", unit: "kWh/m²", type: "calc", formula: "chauf / surface"}
```
Constants from the active constants block (e.g. `surface`, `pers`) are available in formulas by name.

**`reg` formula** — describes the regression model. The left-hand side is the target metric; the right-hand side is a sum of terms `A*variable`, where uppercase letters are the coefficients to fit. A trailing constant term is mandatory:
```javascript
chauf_reg: {name: "Heating forecast", unit: "kWh", type: "reg",
            formula: "chauf=A*dju+B*enso+C", positive: true}
```
Coefficients are computed automatically from the last complete year of data.

#### `graphs` — chart definitions

Each entry in the `graphs` array renders one Chart.js line chart.

| Property | Required | Description |
|---|---|---|
| `title` | yes | Chart heading |
| `unit` | yes | Y-axis label and tooltip unit |
| `metrics` | yes | Ordered array of metric keys to plot as series |
| `timeUnit` | yes | `"month"` or `"year"` — selects the aggregation level |
| `width` | no | CSS width of the chart container (e.g. `"50%"`, `"100%"`) |

```javascript
{title: "Annual consumption", unit: "kWh",
 metrics: ["total", "chauf", "ecs_hc", "divers_elec"],
 width: "50%", timeUnit: "year"}
```

Series colors and dash styles are taken from the corresponding `values` entries, ensuring visual consistency across charts and tables.

#### `tables` — table definitions

Each entry in the `tables` array renders one HTML table.

| Property | Required | Description |
|---|---|---|
| `title` | yes | Table heading |
| `timeUnit` | yes | `"month"` or `"year"` |
| `groups` | yes | Array of column groups (see below) |

Each **group** has a `name` (displayed as a merged header) and a `columns` array. Each column:

| Property | Required | Description |
|---|---|---|
| `name` | yes | Column header label |
| `metric` | yes | Metric key to display |
| `data` | no | Which aggregated value to show: `total` (default), `byDay`, `byMonth`, `byYear` |
| `tooltip` | no | Text shown on header hover |
| `cellTooltip` | no | Name of the metric info field (value + unit) to use for cell tooltip on hover |

```javascript
{title: "Years", timeUnit: "year", groups: [
    {name: "Heating", columns: [
        {name: "kWh",    metric: "chauf",         tooltip: "Total heating energy"},
        {name: "kWh/m²", metric: "chauf_surface"},
        {name: "Wood %", metric: "ratio_bois_total"},
    ]},
    {name: "Hot water", columns: [
        {name: "kWh",   metric: "ecs_hc"},
        {name: "kWh/j", metric: "ecs_hc", data: "byDay"},
    ]},
]}
```

Cells are automatically coloured with a gradient proportional to the column's min/max range.


## TODO
    - Allow to choose the reference year for linear regression?  Requires to rebuild the page and make prepare idempotent
    - Allow to compute wood difference between total and model estimation, requires to evaluate dependencies between variables and to compute them in the right order

