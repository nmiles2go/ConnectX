# ConnectX - AI-Driven Fiber & Market Intelligence Platform

### Project Idea
ConnectX is an AI-driven decision-support platform built using IBM WatsonX.ai and Watson Orchestrate. It helps telecom providers and planners identify, evaluate, and prioritize DSL-to-Fiber conversion projects nationwide, enabling data-driven rollout planning for next-generation broadband infrastructure.

---

### Project Overview
Despite nationwide broadband initiatives, around 15–20% (~10 million) U.S. households, primarily in rural and semi-urban areas — still rely on DSL.
DSL suffers from low speeds, high latency, and signal degradation, while Fiber-to-the-Home (FTTH) offers speeds exceeding 1 Gbps, ultra-low latency, and high reliability, essential for remote work, telemedicine, 4K/8K streaming, and IoT connectivity.

However, fiber deployment requires significant upfront investment, so operators face key questions:

<ul>1. Which regions offer the highest adoption potential if fiber is introduced?</ul>

<ul>2. What will be the capital cost per premise?</ul>

<ul>3. How long will it take to break even, and what is the expected ROI?</ul>

<ul>4. In what sequence should regions be upgraded to maximize returns?</ul>

ConnectX addresses this by providing an AI-assisted framework that forecasts market adoption, estimates capital costs, and prioritizes deployment phases based on both financial and demographic factors.

---

### Project Architecture
ConnectX is designed as a multi-agent system hosted and orchestrated through IBM watsonx.ai and Watson Orchestrate.

##### Agents Overview
<table> 
  <thead> 
    <tr> 
      <th>Agent</th> 
      <th>Purpose</th> 
      <th>Core Functions</th> 
      <th>Outputs</th> 
    </tr> 
  </thead> 
  <tbody> 
    <tr> 
      <td><b>Marketing Data Analyst Agent</b></td> 
      <td>Predicts expected Fiber Take Rate for each region based on demographics and competition.</td> 
      <td><code>score_take_rate()</code></td> 
      <td><code>take_rate_pct</code></td> 
    </tr> 
    <tr> 
      <td><b>Financial Costing Agent</b></td> 
      <td>Estimates per-premise CAPEX, ROI, and Payback Years using regional and network parameters.</td> 
      <td><code>estimate_capex()</code>, <code>predict_roi()</code></td> 
      <td><code>capex_per_premise_usd</code>, <code>roi_3yr_pct</code>, <code>payback_years</code></td> 
    </tr> 
    <tr> <td><b>GTM Project Lead Agent</b></td> 
      <td>Assigns rollout <b>Priority Rank</b> and <b>Phase Alignment</b> for deployment planning.</td> 
      <td><code>priority_score()</code>, <code>assign_phase()</code></td> <td><code>priority_score</code>, <code>phase</code></td> 
    </tr> 
  </tbody> 
</table>

---

### Project Implementation

As the first step of the project, I used IBM watsonx.ai’s data generation tool to create a synthetic dataset of 500 regions, each representing a potential broadband service area. Using Python, I then refined and standardized the dataset, renaming columns and structuring key demographic, network, and economic attributes such as population, median income, broadband adoption, churn rate, and competitor presence.

<img width="892" height="337" alt="Screenshot 2025-11-01 at 6 15 21 PM" src="https://github.com/user-attachments/assets/276784a1-cf2d-475d-9263-3102555548e5" />

#### Data Models and Inputs:

<table>
  <thead>
    <tr>
      <th>Category</th>
      <th>Example Columns</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Demographic</strong></td>
      <td>
        <code>region_id</code>, <code>state</code>, <code>population</code>, <code>households</code>, 
        <code>median_income_usd</code>, <code>education_index</code>, <code>urban_rural_flag</code>
      </td>
      <td>Defines socio-economic characteristics of each area</td>
    </tr>
    <tr>
      <td><strong>Geospatial</strong></td>
      <td>
        <code>population_density_km2</code>, <code>terrain_slope_deg</code>, 
        <code>road_density_km_per_km2</code>, <code>existing_backhaul_distance_km</code>
      </td>
      <td>Represents network build complexity and cost factors</td>
    </tr>
    <tr>
      <td><strong>DSL Service</strong></td>
      <td>
        <code>dsl_coverage_pct</code>, <code>avg_dsl_speed_mbps</code>, 
        <code>complaint_rate_pct</code>, <code>churn_rate_pct</code>
      </td>
      <td>Reflects service quality and dissatisfaction</td>
    </tr>
    <tr>
      <td><strong>Market Metrics</strong></td>
      <td>
        <code>broadband_adoption_pct</code>, <code>competitor_fiber_presence</code>, 
        <code>avg_monthly_data_gb</code>
      </td>
      <td>Indicates current adoption and competition intensity</td>
    </tr>
  </tbody>
</table>


As for the next steps of Agent creation, I built the agents using the Agents Lab on the watsonX.ai, 
<li>Step 1 — Marketing Data Analyst Agent (Take-Rate Estimation) : Predicts fiber adoption likelihood using:</li> 

  - Broadband adoption %
  - Median income normalization
  - Competitor fiber presence
  - Complaint rate 
  - Urban vs. rural factor

**Output**: `take_rate_pct`

<li>Step 2 — Financial Analysis (Financial Costing Agent): Predicts CAPEX per premise (USD), Payback year, ROI over 3-year horizon using:</li>

  - Population Density (per square km)
  - Slope of the terrain
  - Existing backahaul distance in km
  - Urban vs. Rural factor
  - Number of Households

**Output**: `capex_per_premise_usd`, `payback_years`, `roi_years_pct`

<li>Step 3 — Strategic Prioritization (Project Manager & Strategy Agent): Predicts priority_score (0–1 scale), phase classification using:</li>

  - Take Rate %
  - ROI over a 3-year horizon
  - Capex per premise (USD)
  - Competitor fiber presence

**Logic**: High priority = low cost, high take rate, fast ROI.

**Output**: `priority_score`, `assigned_phase`

<li> Step 4 — AI Orchestration (Watson Orchestrate): Watson Orchestrate connects all agents into a parallel workflow, based on the query type, the specific agent is called which will execute the query using the required tools such as `capex_per_premise_usd`, `assign_phase`, `predict_take_rate`, etc:</li><br>

  - User prompt → Call for Domain Specific Agents (Take-Rate Agent / Financial Agent / Strategy Agent) → Result-driven Insight

Example Query:
“Based on the provided information, predict the capex, priority score, and roi: population density: 471.95 slope: 5.32 existing backhaul: 1.96 urban or rural: Rural households: 50881 "take_rate_pct": 67.8, competitor: 2”

Example Output on the backend:
``` 
{
  "capex_per_premise_usd": 2235.58,
  "payback_years": 5.64,
  "roi_3yr_pct": -46.5,
  "priority_score": 0.445,
  "explaination": "The CAPEX per premise of $2,236 is reasonable for a rural build, though the Rural premium and moderate slope increase costs slightly.
                  A take rate of 67.8% indicates strong market demand, ensuring long-term revenue stability.
                  With a payback period of 5.6 years, the investment is viable but not ideal for short-term ROI — the 3-year ROI (-46.8%) reflects high upfront costs typical of rural fiber expansion."
}

```

To make the system interactive, I connected ConnectX to Slack using Watsonx Orchestrate’s Channels feature. Through this integration, users can directly interact with the agents, such as querying take rates, ROI, or project priorities, within a Slack workspace. The connection was configured via the Channels section in Watsonx Orchestrate, where Slack was authorized as a communication endpoint for real-time orchestration.

<img width="1920" height="1080" alt="Screenshot 2025-11-01 at 7 33 35 PM" src="https://github.com/user-attachments/assets/f89f90de-4be3-480e-a07e-b8ca182ff4f2" />

---

### Integrations

1. Orchestrated in: IBM Watson Orchestrate
2. Interface: Slack or API endpoint
3. Runtime: Python 3.11 on watsonx.ai (runtime-24.1-py3.11)

---

### Scope and Conclusion

By helping telecom operators efficiently target high-potential areas, ConnectX contributes to:<br>
- Expanding fiber coverage across underserved U.S. communities.
- Accelerating the national transition toward universal high-speed broadband.
- Reducing digital inequality between urban and rural areas.
- Supporting economic growth, education, and telehealth access through better connectivity.

**ConnectX** represents a fusion of analytics, economics, and AI orchestration — transforming how network operators plan fiber upgrades. Simulating both consumer behavior and financial performance ensures every investment decision is data-backed, transparent, and explainable.

---

### Credits

A huge thanks to the IBM Watsonx team for organizing this hackathon and giving Team Agentylytics (Riya Shetty and Rishikesh Jaiswal) the opportunity to explore the Watsonx platform. This experience allowed us to dive deep into the world of Agentic AI systems, understanding how intelligent agents can be designed, orchestrated, and applied to solve real-world challenges using the Watsonx ecosystem.

