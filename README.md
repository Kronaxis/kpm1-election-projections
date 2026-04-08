# KPM-1: UK Local Election Projections

**136 English councils. 65,000 synthetic personas. Pre-registered predictions.**

KPM-1 (Kronaxis Projection Model 1) projects the outcomes of the 7 May 2026 UK local council elections using synthetic persona panels grounded in Census 2021 demographics and DYNAMICS-8 psychometric personality profiles.

No real humans are surveyed. The projection is generated entirely from synthetic personas, corrected against published election data, referendum results, and demographic statistics.

## Methodology

See [METHODOLOGY.md](METHODOLOGY.md) for the full plain-English explanation of every step.

## Pre-registration

Predictions will be published and cryptographically hashed (SHA-256) before polls open on 7 May 2026. The hash proves the predictions were not modified after seeing results.

| File | Description |
|------|-------------|
| `predictions/may7_2026_projections.json` | Full 136-council projections |
| `predictions/pre_registration_hash.txt` | SHA-256 hash of the projections file |
| `data/bayesian_priors.json` | Statistical priors per council (no LLM) |
| `data/uns_baseline.json` | Uniform National Swing baseline (dumb model to beat) |

## External Data Sources

| Dataset | Coverage | Source |
|---------|----------|--------|
| Census 2021 demographics | 650 constituencies | ONS |
| Election history 2018-2025 | 38/136 councils | Wikipedia |
| Candidate counts | 136 councils | Democracy Club |
| Brexit referendum Leave% | 136 councils | Electoral Commission |
| Census 2021 ethnicity | 140 councils | ONS |
| IMD deprivation index | 136 councils | ONS |
| National polling averages | 5 parties | Published polls |

## Validation

| Metric | Result |
|--------|--------|
| In-sample backtest (38 councils, 2022-2024) | 76% winner accuracy |
| Leave-one-out cross-validation | 71% winner accuracy |
| Sensitivity analysis (6 scenarios) | 135/136 councils stable |
| UNS baseline comparison | Our model is substantially more plausible |

## Success Criteria

Published before the election:
- **Confident tier:** 80%+ winner accuracy
- **National vote share:** Within 4pp per party vs BBC projection
- **Overall direction:** 65%+ council winners correct

## Election Night

Results vs projections will be updated live from 11pm on 7 May 2026.

## Post-Election

Analysis will be published within hours, leading with misses. If the model fails, we will say so directly and explain why.

## Licence

Data and methodology: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may use, share, and adapt with attribution.

Prediction model code: proprietary (Kronaxis Ltd).

## About

Built by [Kronaxis](https://kronaxis.co.uk). KPM-1 uses the DYNAMICS-8 personality framework and a proprietary fine-tuned language model.

This is a scientific experiment with a publicly stated hypothesis, not a marketing exercise.
