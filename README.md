# KPM-1: UK Election Projections

**Pre-registered predictions. Closed proprietary model. Public predictions and SHA-256 hash for verification.**

This repository contains pre-registered predictions and SHA-256 hashes for UK elections produced by KPM-1 (Kronaxis Persona Model 1), a synthetic-panel modelling system. The hash for each pre-registered run is committed to this repository **before** any ballot is cast, so anyone can verify after results land that the predictions were not retroactively adjusted.

KPM-1 itself, the 65,000-persona UK dataset, and the calibration pipeline are proprietary to Kronaxis Limited. Predictions and hash exist only for verification — the model is not open source. Commercial enquiries: jason@kronaxis.co.uk.

→ Live predictions browser: <https://kronaxis.co.uk/election-results>
→ Methodology: <https://kronaxis.co.uk/methodology>
→ Validation paper (103 cross-domain benchmarks): <https://kronaxis.co.uk/research>

---

## Pre-registered runs

### 7 May 2026 UK local council elections

The first public election test of KPM-1.

- **Predictions:** [`predictions/may7_2026_projections.json`](predictions/may7_2026_projections.json)
- **SHA-256 hash:** [`predictions/pre_registration_hash.txt`](predictions/pre_registration_hash.txt)
- **CSV export (per-council):** [`predictions/may7_2026_kronaxis_council_projections.csv`](predictions/may7_2026_kronaxis_council_projections.csv)
- **Methodology snapshot:** [`METHODOLOGY.md`](METHODOLOGY.md)
- **Pre-registration committed:** 1 May 2026, before voting opened
- **Post-mortem:** to be added 8 May 2026

### Verifying the hash

To verify the predictions JSON has not been altered since pre-registration:

```bash
curl -L -o predictions.json \
  https://raw.githubusercontent.com/Kronaxis/kpm1-election-projections/main/predictions/may7_2026_projections.json
sha256sum predictions.json
# Compare against the value in predictions/pre_registration_hash.txt
```

If the output of `sha256sum` matches the hash committed to this repository on 1 May 2026, the predictions are unchanged.

---

## Methodology summary

Each council prediction is the output of:

1. A 200-persona panel drawn from constituency-level demographic data (proportional to ward composition)
2. A two-question protocol per persona — favourability rating, then voting intention given turnout
3. Turnout simulation (~30% of personas effectively vote, matching real local-election turnout)
4. Demographic post-stratification to match council population
5. 70% LLM + 30% statistical-baseline ensemble blend
6. V9 calibration (incumbency boost, protest multiplier, brexit-Reform correlation, ethnicity-Gaza scaling)
7. Tactical-voting layer (when Lab+LD+Grn combined > 45% with non-progressive leading)
8. Relational anchor cap — every party's projected share bounded to [2024 GE share + national swing ± elasticity]
9. Bootstrap confidence interval — 1000 resamples for win-probability distribution

Full methodology: <https://kronaxis.co.uk/methodology>

---

## Pre-registration philosophy

Pre-registration is what makes a prediction falsifiable. Without a public hash committed before results land, any model can quietly retract or reframe its predictions in the post-mortem. With a public hash, the predictions are the predictions — there is no ambiguity about what was claimed and no opportunity to retroactively adjust.

Kronaxis publishes:

- **Predictions:** the full JSON, every council, every party, vote shares, confidence tier, win probability
- **Hash:** SHA-256 of the JSON, committed before voting
- **Methodology:** at the level of detail required for academic critique (see <https://kronaxis.co.uk/research>)
- **Known biases:** publicly documented before results, so we cannot retro-explain failures
- **Manual override audit trail:** any council where we override the raw model output is documented in `data/manual_overrides_applied.json` with reason and pre-/post-override values

Kronaxis does NOT publish:

- KPM-1 model weights or LoRA adapters (proprietary)
- The 65,000-persona UK dataset (proprietary)
- The calibration pipeline source code (proprietary)
- Per-persona reasoning traces (available under commercial licence)

---

## Citation

If you use KPM-1 outputs in academic or journalistic work, please cite:

```
Duke, J. (2026). KPM-1: Synthetic-panel modelling for UK political prediction.
Pre-registered May 2026 council predictions. Kronaxis Limited.
https://kronaxis.co.uk/methodology
```

---

## Contact

- Methodology questions / commercial enquiries: jason@kronaxis.co.uk
- Issues / methodological feedback: this repo's Issues tab
- Live predictions browser: <https://kronaxis.co.uk/election-results>

---

*Kronaxis Limited (registered in England, no. 15072850).*
