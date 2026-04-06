# Evaluation of Forest Conservation Programs in the Amazonia Rainforest

**Duke MIDS Capstone Project** | In collaboration with **Conservation International (CI)**

**Team:** Vivek Sahukar & Zhenhua Wang  
**CI Partners:** Dr. Rachel Golden Kroner & Dr. Sebastien Costedoat  
**Faculty Lead:** Dr. Alex Pfaff (Duke Sanford School of Public Policy)  
**Capstone Director:** Dr. Gregory Herschlag | **Project Managers:** Dr. Heather Huntington & Dr. Ryan Huang  
**Date:** April 2020

---

## Project Overview

Brazil holds 60% of the Amazonia rainforest — the world's largest tropical forest — yet annual deforestation reached 7,900 km² in 2018, the highest rate since 2008. The Brazilian government introduced **Bolsa Verde (BV)**, a Payment for Ecosystem Services (PES) program running from 2011–2018, as a conditional cash transfer to extremely poor households in exchange for maintaining ≥80% forest cover.

**Conservation International's goal** is to achieve zero net deforestation in Amazonia. To support this, they needed rigorous evaluation of where and when Bolsa Verde actually works — not just whether it works on average.

This project answers three questions for CI:
1. Are Protected Areas (PA) and PES successful in reducing deforestation overall?
2. Which geographic and environmental variables drive forest cover loss?
3. **When is Bolsa Verde most effective within a Protected Area?**

---

## Key Finding

> **Bolsa Verde reduces forest loss by 18.8% in regions closer to cities and roads (low elevation, low slope). It is ineffective in remote, high-elevation regions — where deforestation pressure is already low and people enroll in BV to receive payments without actually changing behavior.**

This finding directly informs CI's targeting strategy: conservation payments are most cost-effective when deployed in high-threat, accessible regions — not in naturally protected remote areas.

---

## Methods

The analysis uses a **causal inference** framework — a randomized study was not possible since the policy was already implemented, so we used:

### 1. Propensity Score Matching (PSM)
- Fitted logistic regression on all geographic covariates (slope, elevation, distance to road, city access, precipitation, temperature) with treatment variable = presence of Bolsa Verde
- Matched each treated region to the most similar untreated region (k=1 nearest neighbor, without replacement)
- Verified balance using balance tables before and after matching

### 2. K-Means Clustering
- Clustered the entire Amazonia study region into 4 geographically distinct sub-regions based on environmental covariates
- This captured spatial heterogeneity: Cluster 1 (close to cities/roads, low elevation), Cluster 2 (remote, far from roads), Cluster 3 (high elevation/slope), Cluster 4 (low precipitation, moderate access)
- PSM was then run separately within each cluster to avoid confounding across heterogeneous regions

### 3. Logistic & Poisson Regression
- Logistic regression used for binary forest loss outcome (any loss vs. no loss)
- Poisson regression used to estimate the magnitude of Bolsa Verde's effect per cluster
- Interaction terms between treatment (BV) and cluster included to quantify heterogeneous treatment effects

---

## Data Sources

| Variable | Source |
|---|---|
| Forest cover loss (outcome) | Hansen/UMD/Google/USGS/NASA Landsat analysis |
| Bolsa Verde presence (treatment) | Conservation Governance Atlas |
| Elevation | NASA Shuttle Radar Topographic Mission |
| Slope | Calculated from elevation using ArcGIS |
| Distance to road | Global Roads Open Access Dataset (SEDAC) |
| City accessibility | JRC European Commission travel time dataset |
| Temperature & precipitation | The Climate Data Guide |

**Scale:** Original 30×30m pixels aggregated to 900×900m for computational feasibility. Each aggregated pixel represents the count of 30m plots with forest loss out of 900 total.

---

## Repository Structure

```
DukeMIDSCapstone/
│
├── vs_code_data/
│   └── vs-ci-final.ipynb              # Full Python analysis pipeline (PSM + regression)
│
├── CI Final White Paper 4-15-20.pdf   # Full technical white paper
├── MIDS Capstone Final PPT 4-15-20.pdf  # Final presentation slides
└── README.md
```

**Note on codebase:** Analysis was conducted in a shared remote Jupyter environment (Duke research servers) to comply with CI's data governance requirements — geospatial conservation data could not be transferred outside the secure server. The data files cannot be released publicly as the work was done under a protected data agreement with Duke University and Conservation International. The notebook was exported for documentation purposes. Zhenhua Wang contributed the R-based clustering and balance checking pipeline; Vivek Sahukar contributed the Python-based PSM and regression pipeline documented in `vs-ci-final.ipynb`.

---

## Code: `vs-ci-final.ipynb`

The notebook implements the complete Python analysis pipeline:

| Section | What it does |
|---|---|
| Data loading | Reads 4 datasets: Brazil forest cover (100k sample), PES indicators, PA indicators, cluster assignments |
| Data merging | Inner joins all 4 datasets on spatial ID (`FID`) |
| Feature engineering | Calculates forest cover loss (`fcl = thre2011 - thre2017`) and converts to binary loss variable |
| Balance checking | Computes mean covariate differences between PES/non-PES groups with t-test p-values |
| PSM (full dataset) | Uses `pymatch` library; fits 100 logistic models on balanced samples; k=1 matching without replacement |
| Logistic regression | `statsmodels` Logit with interaction terms for PES × PA; weighted by match quality |
| PSM (PA subset) | Repeats matching pipeline restricted to Protected Areas only |
| Logistic regression (PA) | Second Logit model isolating the PES effect within PA regions |

**Key libraries:** `pymatch`, `statsmodels`, `sklearn`, `scipy.stats`, `pandas`, `numpy`, `seaborn`, `matplotlib`

---

## Results Summary

- **Overall:** Both PA and PES reduce forest cover loss. All geographic covariates are statistically significant (p < 0.05).
- **Direction:** Forest loss decreases with higher elevation, greater distance to roads, and greater city access time — confirming that remote areas face less deforestation pressure naturally.
- **Cluster 1** (low elevation, close to roads/cities): BV reduces forest loss by **18.8%** — highest impact
- **Cluster 3** (high elevation/slope): BV reduces forest loss by only **2.1%**
- **Clusters 2 & 4:** BV is not effective — deforestation pressure is already low without intervention

---

## Policy Implications for Conservation International

The findings suggest CI and the Brazilian government should **concentrate Bolsa Verde enrollment in high-threat, accessible regions** (Cluster 1 profile) rather than spreading it uniformly. In remote, high-elevation areas, program spending does not translate into additional conservation — communities enroll to receive cash transfers in areas where forest clearing is unprofitable regardless.

---

## Full Reports

- [White Paper (PDF)](CI%20Final%20White%20Paper%204-15-20.pdf) — Full methodology, results, and discussion
- [Final Presentation (PDF)](MIDS%20Capstone%20Final%20PPT%204-15-20.pdf) — Visual summary for CI stakeholders
