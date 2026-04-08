# KPM-1 Election Projection Strategy

**Model:** Kronaxis Projection Model 1 (KPM-1)
**Target:** 136 English local council elections, 7 May 2026
**Last updated:** 8 April 2026

---

## 1. What we're doing

We project which party will win the highest vote share in each of the 136 councils being contested on 7 May 2026. We do this by constructing demographically representative panels of 200 synthetic personas per council, running each through a structured voting deliberation, then applying statistical corrections grounded in external data.

No real humans are surveyed. The projection is generated entirely from synthetic personas with census-matched demographics and psychometric personality profiles, corrected against published election data, polling averages, and referendum results.

**This is an experimental method.** It has never been tested at this scale against a real election. The methodology is published before the election so that the results can be evaluated against a stated process, not retrofitted to the outcome.

---

## 2. The personas

65,000 synthetic personas, each representing a fictional British resident with:

- **Demographics:** age, gender, ethnicity, region, town, education, occupation, income, housing type, household composition. Distributions match Census 2021 at regional level.
- **Personality:** eight DYNAMICS-8 dimensions (Discipline, Yielding, Novelty, Acuity, Mercuriality, Impulsivity, Candour, Sociability), scored 0-1. Assigned via demographic correlation models derived from published psychometric literature.
- **Political profile:** party affiliation, engagement level (1-5), key issues, voting history (2019 GE, 2024 GE). Party affiliation distributions are age, region, and education-weighted to approximate known voter alignment patterns.
- **Constituency:** mapped to one of 650 UK parliamentary constituencies.

**Known limitation:** Census 2021 demographics are five years stale. Population shifts since 2021 (university town growth, inner-city gentrification, retirement migration) are not captured. This introduces systematic error in councils with rapid demographic change.

**Known limitation:** DYNAMICS-8 personality scores are generated, not measured. They have been validated against published UK survey distributions (BSA, ONS Wellbeing, GfK Consumer Confidence) but have not been validated against individual-level psychometric assessments of real voters.

---

## 3. Panel construction

For each council, a panel of 200 personas is built:

1. **Constituency matching.** The council is mapped to overlapping parliamentary constituencies. Personas from those constituencies are selected first.
2. **Regional expansion.** If fewer than 400 direct matches exist, the pool expands to the full region.
3. **Demographic weighting.** Each persona is scored by council fit (45%), turnout probability (20%), age (15%), homeownership (10%), and education (10%). Age weighting: 65+ at 3x, 50-64 at 1.5x, reflecting actual local election turnout patterns (source: Electoral Commission turnout reports). Homeowners at 1.5x. Graduates at 1.3x in university towns.
4. **Swing voter oversample.** Personas with high personality volatility (Mercuriality >0.6) and low party loyalty (engagement ≤2) weighted 1.5x. Rationale: marginal outcomes are determined by swing voters, not loyal partisans.
5. **Exclusion.** Personas whose last-voted party is not standing in this council are removed (prevents Scottish National Party voters appearing in English council panels).
6. **Sampling.** 200 personas drawn by weighted random selection with a fixed seed per council for full reproducibility.

**Known limitation:** 200 personas gives a standard error of approximately ±4pp per party. Margins below ~8pp are within noise range, which is why predictions with margins below 8pp are classified as "Toss-up" rather than firm projections.

---

## 4. The voting stimulus

Each persona receives a structured prompt containing their full profile, the council context, and a three-round deliberation protocol:

**Context provided:**
- Council name, area character description, incumbent party
- Parties standing (from Democracy Club)
- Previous election results in this area (where available)
- Current political context: "It is May 2026. Reform UK won 677 council seats in 2025. Labour's national approval has collapsed. The cost of living crisis continues."

**Three-round deliberation:**
1. **Sincere preference:** Which party best represents your values and interests?
2. **Tactical adjustment:** Given who can realistically win here, would you vote differently?
3. **Final vote:** Considering the political mood and your satisfaction with government, what is your final vote?

**Additional outputs:**
- Likelihood to vote (1-10 scale)
- Confidence in choice (1-5)
- One-sentence reasoning

**Bias mitigations:**
- Party order randomised per persona (prevents positional bias)
- Worked example uses a neutral party (not the leading party)
- No leading language about any party's prospects
- Persona is told "you may or may not vote the same way as last time" to prevent anchoring on voting history

**Known limitation:** The three-round protocol is novel and has not been validated against a simpler single-question approach. It may add noise rather than signal. We include it because qualitative testing showed it produced more demographically consistent responses, but this has not been rigorously proven.

**Known limitation:** The political context statement may prime personas towards Reform UK by leading with their 2025 success. The same information framed differently ("Labour lost 1,900 seats in 2025") could produce different results.

---

## 5. The language model

A proprietary 27B parameter model (Kronaxis Imprint) with two stacked fine-tuning layers:

- **Persona adapter:** trained on 500,000 reasoning traces to produce persona-consistent responses. Without this, the base model produces homogeneous responses regardless of persona demographics.
- **Political adapter:** trained on 5,000 UK voting decision examples covering all 136 council types, calibrated against published election results. Without this, the model lacks knowledge of post-2025 UK political dynamics.

Temperature is adjusted per council: 0.20 for safe seats (historical margin >25pp), 0.35 default, 0.50 for ultra-marginals (<5pp margin). This reflects genuine uncertainty: we should be less certain about close races.

**Known limitation:** The political adapter was trained on synthetic examples generated by a separate language model, not on real voter survey data. The training data reflects our assumptions about how voters behave, which may contain systematic biases we cannot detect without real-voter validation.

---

## 6. Vote parsing and filtering

Responses are parsed from JSON, with regex fallback for malformed output. Party names are normalised across 40+ aliases.

**Turnout filtering (likelihood-to-vote):**
- Score 1-3: excluded (modelled as non-voters)
- Score 4-6: weighted at 0.5x
- Score 7-10: full weight

Parse failures receive a forced-choice re-query. Persistent failures are excluded from the denominator.

**Known limitation:** Excluding parse failures and low-likelihood personas reduces the effective panel size. In testing, 95-100% of responses parse successfully, but systematic parse failure in certain persona types (low education, low engagement) could bias the sample towards more politically engaged demographics.

---

## 7. The correction pipeline

Raw vote counts are processed through a multi-layer statistical correction pipeline. Each layer addresses a specific known bias or data gap.

**Rationale for corrections:** An uncorrected LLM produces vote shares that systematically differ from real elections in predictable ways: it overpredicts the largest party (central tendency bias), underpredicts small parties (frequency bias), ignores local incumbency effects, and cannot model protest voting dynamics. The corrections address each of these.

### Statistical corrections (Layers 0.5-4)
- **Trimmed mean:** Bootstrap resampling with 5% trim. Standard survey methodology.
- **Satisfaction weighting:** Amplifies protest voters who are dissatisfied with the incumbent. Grounded in the well-documented anti-incumbent bias in UK local elections.
- **Floor and regression to mean:** 2% floor per standing party, shrinkage on extremes. Prevents impossible 0% or 60%+ projections. James-Stein estimation.
- **Incumbency boost:** +6pp for the incumbent party. Based on published research showing local incumbency advantage of 5-8pp in UK elections (Rallings and Thrasher).
- **Protest adjustment:** 1.2x anti-incumbent multiplier for local elections. Local elections have higher protest voting than general elections (source: Electoral Reform Society).
- **Statistical ensemble:** 45% LLM + 55% statistical baseline. The statistical baseline uses national polls adjusted for council demographics. This ensemble prevents the LLM from producing wild outliers while still capturing persona-specific reasoning.

### Calibration corrections (Layers 5-6)
- **Per-party calibration:** Corrections derived from 10 validated by-election predictions. London receives 50% of the national Reform/Labour correction.
- **Calibration cap:** No correction exceeds ±5pp. Prevents cascading overcorrection.

**Known limitation:** Calibration from 10 by-elections is a small sample. The corrections may not generalise to the full 136 councils, particularly non-London metropolitan boroughs and district councils that are underrepresented in the calibration set.

### External data corrections (Layers 6.4-6.6)
- **Green national correction:** The model systematically underpredicts Green (3-4% vs 17% in polls). A floor correction partially addresses this, but the root cause (insufficient Green voter representation in training data) remains.
- **UK micro-dynamics:** Hardcoded corrections for known local dynamics that the model cannot learn from national data:
  - Gaza effect (6 councils): Labour ceiling in councils with large Muslim populations where Labour support collapsed over Gaza policy.
  - Green surge (8 councils): Green floor in university cities and progressive urban areas.
  - Conservative wipeout (10 inner London boroughs): Conservative ceiling at 8%.
  - Lib Dem fortress (3 councils): Lib Dem floor in Richmond, Kingston, Sutton.
- **Brexit-Reform correlation:** Leave vote percentage (2016) used to adjust Reform UK share. Every 10pp above 50% Leave adds ~3pp to Reform, dampened by 60% in London. This is the strongest single external predictor of Reform vote share (correlation coefficient approximately 0.7 in 2025 local results).
- **Ethnicity-Gaza scaling:** Proportional Labour loss based on Asian population percentage (Census 2021). Supplements the hardcoded Gaza effect with a continuous variable.
- **Independent floors:** Councils with strong independent/residents' association traditions get a floor based on historical performance.

**Known limitation:** The micro-dynamics corrections are based on the author's assessment of current UK politics, not on a systematic model. They embed editorial judgements (e.g. "Conservative ceiling of 8% in inner London") that may prove wrong.

### Post-processing (Layers 7-8)
- **Regional smoothing:** 80% own model + 20% neighbouring council average. Reduces noise from unrepresentative persona samples.
- **National recalibration:** If aggregate national shares differ from polls by >3pp per party, half the difference is applied as a correction. This is a guard against systematic model bias, not a replacement for the model's output.

**Known limitation:** National recalibration pulls projections towards polls. If our model detects a genuine divergence from polls (e.g. higher Reform in the north than polls suggest), the recalibration will partially suppress that signal. This is a deliberate trade-off: we accept reduced sensitivity in exchange for reduced risk of systematic national-level error.

---

## 8. Confidence classification

- **Confident:** Margin between 1st and 2nd exceeds 15pp
- **Lean:** Margin 8-15pp
- **Toss-up:** Margin under 8pp

Projections where the predicted winner had less than 10% vote share in the previous election are automatically downgraded to Toss-up (implausibility filter).

Bootstrap 90% confidence intervals are computed for each party in each council. Where two parties' confidence intervals overlap, the projection is Toss-up regardless of the point estimate.

---

## 9. External data sources

| Dataset | Coverage | Source | Used for |
|---------|----------|--------|----------|
| Constituency personas | 650 constituencies | Census 2021 | Panel construction |
| Election history 2018-2025 | 38/136 councils | Wikipedia | Previous results |
| 2025 local results | 4 councils | BBC | Most recent anchor |
| Candidate counts | 136 councils | Democracy Club | Party lists |
| Deprivation index | 136 councils | ONS IMD | Calibration |
| Brexit Leave% | 136 councils | Electoral Commission | Reform UK correction |
| Asian population % | 140 councils | Census 2021 | Gaza effect scaling |
| National polls | 5 parties | Published averages | Baseline + recalibration |
| By-election results | 10 wards | Published results | Calibration |

**Known limitation:** Only 38 of 136 councils have historical election data. For the remaining 98, the projection relies on national polls adjusted by Brexit vote, deprivation, and demographic composition, with no council-specific anchoring. These projections are inherently less reliable.

---

## 10. Validation

**Backtest against known results (2022-2024):**
- 38 councils with published outcomes
- Statistical baseline with all corrections (no LLM): **76% winner accuracy** (29/38)

**Miss analysis (9 incorrect):**

| Category | Count | Examples | Root cause |
|----------|-------|----------|------------|
| Label mismatch | 1 | Bury (Labour Co-op vs Labour) | Same party, different label |
| Local independents | 2 | Havering RA, Tower Hamlets Aspire | Unpredictable non-party candidates |
| Outer London marginals | 3 | Bromley, Croydon, Kensington | Conservative slightly ahead of Labour in close races |
| Non-fortress Lib Dem | 1 | Calderdale | Strong local Lib Dem campaign not captured |
| Brexit overcorrection | 2 | Hounslow, Merton | 45-50% Leave London boroughs miscorrected |

**Critical caveat:** This backtest is partially in-sample. The micro-dynamics corrections (Gaza effect, Lib Dem fortress) were designed with knowledge of these councils' histories. The backtest validates that the corrections work in the data they were built for, not that they generalise to unseen councils. True out-of-sample accuracy is unknown until 7 May.

**UNS baseline comparison:**
- Uniform National Swing (the simplest possible model) predicts Reform winning 98/136 councils.
- KPM-1 predicts Reform 45, Labour 37, Lib Dem 24, Con 21, Green 9.
- The UNS result is clearly implausible, confirming that council-level correction adds value over naive swing.

---

## 11. Pre-registration

Before polls open:
1. Full 136-council projections saved as JSON
2. SHA-256 hash computed from deterministic JSON serialisation
3. Hash published on GitHub and Twitter before 7am on 7 May
4. This is cryptographic proof the projections were not modified after seeing results

**The methodology document (this document) is also published before the election.** This means the corrections, assumptions, and known limitations are on record before the outcome is known.

---

## 12. Election night

**20 earliest-declaring councils** identified with expected declaration times (Sunderland 23:30, London boroughs midnight-2am, metropolitan boroughs 2am-4am).

**8 bellwether councils** with specific interpretation thresholds:
- Sunderland: Reform margin indicates national Reform strength
- Wandsworth: Labour hold indicates London base intact
- Birmingham: Labour share indicates Gaza effect magnitude
- Hackney: Green share indicates urban Green surge reality
- Barnsley: Reform margin indicates Red Wall completion
- Tower Hamlets: Aspire hold indicates ethnic bloc voting durability

**Live tracker** records projections vs actuals as results come in, with running accuracy count.

**Pre-written analysis templates** for rapid publication: exceptional result (80%+), decent result (65-79%), poor result (below 65%). Each includes miss analysis framework.

---

## 13. Success criteria

Published before the election:
- **Confident tier:** 80%+ winner accuracy
- **National vote share:** Within 4pp per party vs BBC national projection
- **Overall direction:** 65%+ council winners correct across all tiers

These thresholds are set before the election and will not be changed after results.

---

## 14. Known risks and failure modes

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Tactical voting invisible to persona model | HIGH | 5-10pp error in marginals | Ensemble with statistical baseline |
| Late campaign events shift vote | MODERATE | 3-5pp national swing | Run projections as late as possible (3-4 May) |
| Green systematically underpredicted | HIGH | Miss Green wins in 3-5 councils | National Green correction + surge councils |
| Brexit-Reform correlation decayed since 2016 | MODERATE | Reform over/underpredicted by 3-5pp | Dampened coefficient, capped at ±8pp |
| No historical data for 98/136 councils | HIGH | Weaker projections outside London/metros | National recalibration + regional smoothing |
| Calibration from 10 by-elections too small | HIGH | Corrections may not generalise | Cap at ±5pp, ensemble with baseline |
| Census 2021 demographics stale | MODERATE | Panel composition error | Regional-level matching (less sensitive to local shifts) |
| Personas don't behave like real voters | UNKNOWN | Systematic bias in all projections | **This is the fundamental untested assumption** |

---

## 15. What this proves (if it works)

That synthetic personas grounded in census demographics and psychometric personality profiles can project real-world electoral outcomes at population scale, without surveying a single real voter. This would validate the underlying technology for commercial applications in market research, policy testing, and brand strategy, where the same persona infrastructure is used to simulate consumer and citizen behaviour.

---

## 16. What this proves (if it doesn't work)

That the model has systematic biases we will identify and publish within hours. The pre-registration, transparent methodology, and stated success criteria mean we cannot hide from a poor result. The analysis will distinguish between:
- **Fixable errors:** calibration parameters that need adjustment, missing data sources, correction layers that overcorrected
- **Fundamental limitations:** if synthetic personas simply do not capture real voting behaviour, no amount of correction can fix that, and we will say so directly

---

*Document version: 8 April 2026. Published before the election for accountability.*
