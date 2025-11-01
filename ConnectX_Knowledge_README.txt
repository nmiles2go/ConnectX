CONNECTX – MARKET DEMAND AGENT KNOWLEDGE (UPLOAD AS .TXT)

Purpose
- Canonical schema, default scoring rules, validation/imputation policy, and output contract for the ConnectX Market Demand Agent.
- The end user uploads ONE CSV with data rows; this knowledge file defines the contract and defaults used at runtime.

How to use
1) Upload this TXT file into the Knowledge section.
2) (Optional) Also upload the accompanying CSV files:
   - ConnectX_Schema.csv
   - ConnectX_Aliases.csv
   - ConnectX_DefaultWeights.csv
   - ConnectX_ExampleData.csv
3) In your Orchestrate play’s first Transform step, load these knowledge files to validate the user CSV and apply defaults.

------------------------------------------------------------
SECTION A — SCHEMA (summary; full table in ConnectX_Schema.csv)

- region_id: string (required) — unique region identifier.
- region_name: string — human-readable region name.
- state: string — state or region.
- population: number [10,000–150,000]
- households: number [3,000–60,000]
- population_density_km2: number [50–800]
- median_income_usd: number [30,000–110,000]
- education_index: number [0.4–0.9]
- urban_rural_flag: enum {Urban, Rural}
- dsl_coverage_pct: number [40–95]
- fiber_coverage_pct: number [5–60]
- avg_dsl_speed_mbps: number [5–50]
- avg_fiber_speed_mbps: number [300–1000]
- competitor_fiber_presence: integer [0–4]
- complaint_rate_pct: number [2–20]
- churn_rate_pct: number [2–15]
- broadband_adoption_pct: number [40–95]
- avg_monthly_data_gb: number [100–600]
- terrain_slope_deg: number [0–12]
- road_density_km_per_km2: number [0.5–3.0]
- existing_backhaul_distance_km: number [0.5–10]

Missing or invalid values: impute with column median (numeric) or mode (categorical) and log a warning.

------------------------------------------------------------
SECTION B — COLUMN ALIASES (summary; full table in ConnectX_Aliases.csv)

- region_id: RegionID, regionId, REGION_ID
- population_density_km2: pop_density, PopulationDensity
- broadband_adoption_pct: adoption_rate, broadband_use
- complaint_rate_pct: complaint_rate, dissatisfaction_rate
- competitor_fiber_presence: competitors, competition_index

------------------------------------------------------------
SECTION C — DEFAULT SCORING RULES (full key-values in ConnectX_DefaultWeights.csv)

Normalization
- Min–max per uploaded file: x_norm = (x - min)/(max - min + 1e-9)

Predicted Take-Rate (percent)
- take_rate_predicted_pct = 100 * clip(
    0.35*complaint_rate_pct_norm +
    0.25*median_income_usd_norm +
    0.25*broadband_adoption_pct_norm +
    (1 - 0.15*competitor_fiber_presence_norm), 0, 1 )

Demand Score (0–100)
- usage_proxy_norm = avg_monthly_data_gb_norm if present else broadband_adoption_pct_norm
- demand_score_0_100 = 100 * clip(
    0.40*population_density_km2_norm +
    0.30*take_rate_predicted_pct_norm +
    0.30*usage_proxy_norm, 0, 1 )

Ranking
- rank_by_demand: sort by demand_score_0_100 (desc), 1 = highest

Notes (deterministic, rule-based)
- Short tags like "density↑", "affordability↑", "competition↑" based on median splits.

------------------------------------------------------------
SECTION D — VALIDATION & IMPUTATION POLICY

- Required column: region_id
- For required numeric columns missing entirely, create column and impute with the median of available rows.
- For per-row missing numeric cells, impute with column median; for categorical, impute with mode.
- Cap numeric values to schema min/max bounds after imputation.
- Log all fixes to a warnings array for transparency.

------------------------------------------------------------
SECTION E — OUTPUT CONTRACT

The Market Demand Agent returns:
- CSV columns added: take_rate_predicted_pct, demand_score_0_100, rank_by_demand, notes
- JSON object (optional): 
  { "regions":[{ "region_id", "take_rate_predicted_pct", "demand_score_0_100", "rank_by_demand", "notes" }],
    "assumptions":{ "normalization":"min-max", "weights":{...} }, "warnings":[] }

This output is consumed by the Infra Cost Agent and Strategy Agent.

------------------------------------------------------------
SECTION F — EXAMPLE DATA (see ConnectX_ExampleData.csv for a small sample)

REG-001, Region 1, Indiana, 45000, 18000, 220, 60000, 0.65, Urban, 70, 20, 25, 600, 1, 8, 6, 80, 260, 3.5, 2.2, 4.0
REG-002, Region 2, Ohio,    90000, 36000, 480, 70000, 0.72, Urban, 65, 30, 28, 700, 2, 5, 7, 85, 340, 2.1, 2.8, 3.0

File naming rules reminder: use unique names; keep CSV/TXT under 5 MB.