# KPM-1: UK Local Election Projections

**159 English councils. 65,000 synthetic personas. 12,224 reasoning traces. Pre-registered predictions.**

KPM-1 (Kronaxis Projection Model 1) projects the outcomes of the 7 May 2026 UK local council elections using synthetic persona panels grounded in Census 2021 demographics and DYNAMICS-8 psychometric personality profiles.

No real humans are surveyed. The projection is generated entirely from synthetic personas, corrected against published election data, referendum results, and demographic statistics.

---

## Results (V3, 14 April 2026)

V3 expands coverage from 136 to 159 councils (136 all-out + 23 thirds elections) with 12,224 causal reasoning traces showing why each persona voted the way they did.

### National Vote Share

| Party | Projected Vote Share |
|---|---|
| Reform UK | 28.3% |
| Conservative | 23.4% |
| Labour | 20.1% |
| Liberal Democrat | 18.2% |
| Green | 7.5% |

### Projected Council Control (159 Councils)

| Party | Current Control | Projected Control | Change |
|---|---|---|---|
| Reform UK | 0 | 72 | +72 |
| Liberal Democrat | 15 | 28 | +13 |
| Conservative | 42 | 30 | -12 |
| Labour | 98 | 18 | -80 |
| No Overall Control | 4 | 11 | +7 |

### Projected Seats Won (Councillors)

| Party | Projected Seats | Net Change | Share |
|---|---|---|---|
| Reform UK | 2,043 | **+2,043** | 40.6% |
| Conservative | 1,191 | **-210** | 23.7% |
| Liberal Democrat | 952 | **+599** | 18.9% |
| Labour | 839 | **-2,441** | 16.7% |
| Green | 9 | **+9** | 0.2% |
| **Total** | **5,034** | | |

### London vs Rest of England

| Region | Councils | Reform UK | Conservative | Labour | Lib Dem |
|---|---|---|---|---|---|
| London | 32 boroughs | 5 | 9 | 12 | 6 |
| Rest of England | 104 councils | 62 | 18 | 4 | 20 |

---

## Blog Posts

| Post | Description |
|------|-------------|
| [V2 Full Results (9 April 2026)](https://kronaxis.co.uk/blog/election-results-v2-april-2026) | Complete 136-council projections with national and regional breakdowns |
| [Full Methodology Explained](https://kronaxis.co.uk/blog/election-projections-2026) | Plain-English walkthrough of every step in the projection pipeline |
| [Earlier 20-Council Projections](https://kronaxis.co.uk/blog/predicting-may-7-elections) | Initial projections for the first 20 councils (before full 136-council run) |
| [Technical Paper](https://kronaxis.co.uk/blog/election-prediction-paper) | Detailed technical writeup covering model architecture and correction layers |
| [By-Election Validation](https://kronaxis.co.uk/blog/election-prediction-by-elections) | Validation against March 2026 by-election results (10 wards) |

---

## Results at Every Level

KPM-1 produces projections at four levels of granularity:

- **National:** vote share per party, total seat counts, net seat changes across all 136 councils
- **Regional:** London (32 boroughs) vs Rest of England (104 councils) breakdown of council control
- **Council-level:** 136 individual council projections, each with projected winner, winning margin, confidence tier, and full party vote shares. See [`predictions/may7_2026_projections.json`](predictions/may7_2026_projections.json).
- **Ward-level:** 2,965 ward projections with seat allocation, swing analysis, and local demographic weighting. Ward-level data is available on the [kronaxis.co.uk results pages](https://kronaxis.co.uk/blog/election-results-v2-april-2026) and on request.

---

## Pre-registration

Predictions were published and cryptographically hashed (SHA-256) on 9 April 2026, before polls open on 7 May 2026. The hash proves the predictions were not modified after seeing results.

```
sha256:e873495f33e0cef89b32138e5b1d7cdb6dbaec6a81b602291db832e2601bde2d
```

To verify: hash `predictions/may7_2026_projections.json` with SHA-256 and compare.

| File | Description |
|------|-------------|
| `predictions/may7_2026_projections.json` | Full 136-council projections with per-council vote shares |
| `predictions/pre_registration_hash.txt` | SHA-256 hash of the projections file |

---

## Upcoming Reruns

| Date | Purpose |
|---|---|
| **1 May 2026** | Rerun with latest polling data, candidate lists, and news context |
| **6 May 2026** | Final rerun (eve of election), captures last-minute shifts |
| **8 May 2026** | Post-election analysis: actual vs predicted, published same day |

Each rerun produces a new hash. All versions are archived.

---

## Methodology

See [METHODOLOGY.md](METHODOLOGY.md) for the full plain-English explanation of every step.

**Summary:**

- 65,000 synthetic personas with full demographics, voting history, and DYNAMICS-8 personality profiles
- Each persona deliberates individually: sincere preference, tactical considerations, final vote with reasoning
- A proprietary language model fine-tuned on 5,000 examples of UK local voting behaviour handles the deliberation
- 14 correction layers applied after aggregation (incumbency, protest voting, Brexit-Reform correlation, ethnicity-Gaza effect, candidate ceilings from Democracy Club data, and more)
- Ward-level disaggregation across 2,965 wards

## External Data Sources

| Dataset | Coverage | Source |
|---------|----------|--------|
| Census 2021 demographics | 650 constituencies | ONS |
| Election history 2018-2025 | 38/136 councils | Wikipedia |
| Candidate counts | 136 councils, 4,624 candidates | Democracy Club |
| Brexit referendum Leave% | 136 councils | Electoral Commission |
| Census 2021 ethnicity | 140 councils | ONS |
| IMD deprivation index | 136 councils | ONS |
| National polling averages | 5 parties | Published polls (April 2026) |
| Ward boundaries | 2,965 wards | Democracy Club |

## Validation

| Metric | Result |
|--------|--------|
| In-sample backtest (38 councils, 2022-2024) | 76% winner accuracy |
| Leave-one-out cross-validation | 71% winner accuracy |
| Sensitivity analysis (6 scenarios) | 135/136 councils stable |
| UNS baseline comparison | Model beats naive baseline by 14 points |

## Success Criteria

Published before the election:
- **Confident tier:** 80%+ winner accuracy
- **National vote share:** Within 4pp per party vs BBC projection
- **Overall direction:** 65%+ council winners correct

## Election Night

Results vs projections will be updated live from 11pm on 7 May 2026.

## Post-Election

Analysis will be published within hours, leading with misses. If the model fails, we will say so directly and explain why.

## How to Cite

```
Duke, J. (2026). Kronaxis KPM-1: Synthetic persona election projections for
136 English local councils, May 2026. Kronaxis.
https://github.com/Kronaxis/kpm1-election-projections
```

## Licence

Data and predictions: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may use, share, and adapt with attribution.

Prediction model code: proprietary (Kronaxis Ltd). The DYNAMICS-8 personality framework specification is published under CC BY 4.0.

## About

Built by [Kronaxis](https://kronaxis.co.uk). KPM-1 uses the DYNAMICS-8 personality framework and a proprietary fine-tuned language model.

This is a scientific experiment with a publicly stated hypothesis, not a marketing exercise.
