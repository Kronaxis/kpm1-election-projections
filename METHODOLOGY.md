# How KPM-1 Works: Plain English

**Last updated:** 8 April 2026

---

## The one-sentence version

We built 65,000 fake British people from census data, gave each one a personality, asked 200 of them per council how they'd vote, then corrected the results using real election data, referendum results, and demographic statistics.

---

## Step by step

### Step 1: Build the fake people

We start with the 2021 Census. We know how many 55-year-old white women with a degree live in Birmingham, how many 22-year-old Pakistani men without qualifications live in Bradford, and so on for every combination of age, gender, ethnicity, education, income, and housing across 650 constituencies.

We generate 65,000 fictional people whose demographics, in aggregate, match the real population. Each one gets a name, a town, a job, a salary, a housing situation, and a household.

Each person also gets a personality profile across eight dimensions (our DYNAMICS-8 framework): how disciplined they are, how agreeable, how open to new ideas, how analytical, how emotionally volatile, how impulsive, how blunt, and how sociable. These are scored 0-1 and assigned based on demographic correlations from published psychology research.

Each person gets a political profile: which party they lean towards, how engaged they are (1-5), what issues matter to them, and who they voted for in 2019 and 2024.

### Step 2: Pick 200 people for each council

For each of the 136 councils being contested, we pick 200 people whose demographics match that area. We don't pick randomly. We weight the selection:

- **Older people count more** (3x for over-65s) because they actually show up to vote in local elections. An 18-year-old has a 15% chance of voting. A 70-year-old has a 55% chance.
- **Homeowners count more** (1.5x) because they vote at higher rates than renters.
- **Graduates count more in university towns** (1.3x) because they drive the Lib Dem and Green vote in places like Cambridge and Oxford.
- **Swing voters count more** (1.5x) because they determine who wins marginals. A lifelong Labour voter in a safe seat tells you nothing. A volatile centrist in a marginal tells you everything.

We also remove anyone whose last-voted party isn't standing in this council. No point asking a Scottish National Party voter about a Birmingham election.

### Step 3: Ask them how they'd vote

Each person gets a prompt that says: here's who you are, here's where you live, here's what happened last time, here's what's happening in the country right now. How would you vote?

They answer in three rounds:

1. **Heart:** Which party best represents your values? (Ignore who can win.)
2. **Head:** Given who can actually win here, would you vote differently? (Tactical voting.)
3. **Final answer:** Considering the national mood and how angry you are with the government, what's your actual vote?

They also rate their likelihood of voting on a scale of 1-10. If they say 1-3, we throw them out (they wouldn't actually vote). If they say 4-6, we count their vote at half weight.

The party order is shuffled randomly for each person so the AI doesn't just pick the first one listed.

### Step 4: Count the votes

We now have 200 votes per council. But raw vote counts from fake people are noisy and biased. The AI tends to over-predict the biggest party, under-predict small parties, and ignore local dynamics. So we correct the results.

### Step 5: Apply 14 corrections

This is where the engineering matters. Each correction fixes a specific known bias:

**Statistical cleanup:**
- Remove the top and bottom 5% most extreme responses (outlier trimming)
- Set a 2% floor so no standing party gets 0%
- Pull back any party predicted above 45% or below 3% (prevents absurd results)

**Local election dynamics:**
- Add 6 percentage points to the incumbent party (people tend to re-elect who's in charge locally)
- Then subtract some of that back as a protest vote penalty (people use local elections to punish the national government)

**Grounding in reality:**
- Blend our persona results (45%) with a simple statistical model based on national polls (55%). This prevents the AI from producing wild outliers.
- Apply calibration corrections from 10 previous elections where we know the real answer
- Cap all corrections at 5 percentage points so no single adjustment dominates

**External data corrections:**
- **Green boost:** The AI systematically underpredicts the Green Party. We apply a national floor correction.
- **Gaza effect:** In 6 councils with large Muslim communities (Birmingham, Bradford, Tower Hamlets, Newham, Blackburn, Luton), Labour support has collapsed. We cap Labour and redirect votes to independents and Greens.
- **Green surge:** In 8 university cities (Bristol, Cambridge, Oxford, Norwich, Hackney, Lewisham, Lambeth, Exeter), we set a Green floor of 15%.
- **Conservative wipeout:** In 10 inner London boroughs, Conservatives are down to single digits. We cap them at 8%.
- **Lib Dem fortresses:** In Richmond, Kingston, and Sutton, the Lib Dems always win regardless of national trends. We set a Lib Dem floor of 35-40%.
- **Brexit-Reform link:** Councils that voted heavily to Leave the EU in 2016 have much higher Reform UK support in 2026. For every 10 points above 50% Leave, we add about 3 points to Reform. This is dampened in London (Leave voters in London are demographically different from Leave voters in Sunderland). This is the single strongest external predictor we use.
- **Muslim population scaling:** We use the actual Asian population percentage from the Census to scale the Gaza effect proportionally, not just for 6 hardcoded councils.
- **Independent floors:** In councils with strong residents' associations or independent traditions (Havering, for example), we set a floor.
- **Bankrupt councils:** Four councils (Birmingham, Thurrock, Nottingham, Woking) have effectively declared bankruptcy. Voters there are much angrier with the incumbent. We double the protest vote adjustment.

**Smoothing:**
- Blend each council's result with its neighbours (80% own result, 20% average of adjacent councils). This catches cases where one council gets an unrepresentative panel.
- Check the national aggregate. If our total across all 136 councils says Reform is at 32% but polls say 25%, we apply half the difference back to every council. This prevents systematic national-level bias.

### Step 6: Classify confidence

Each projection gets a label:
- **Confident:** The winner is ahead by more than 15 points. Hard to see how they lose.
- **Lean:** Ahead by 8-15 points. Probably right but could be wrong.
- **Toss-up:** Ahead by less than 8 points. Genuinely uncertain.

If a party jumps from under 10% last time to winning, we automatically downgrade to Toss-up. That kind of swing is possible but suspicious.

### Step 7: Calculate win probabilities

Instead of just saying "Reform wins Barnsley," we say "Reform has a 70% probability of winning Barnsley." We do this by resampling our 200 votes 1,000 times (pulling random subsets) and counting how often each party comes out on top. This captures the genuine uncertainty from having only 200 people.

### Step 8: Cross-check with a Bayesian prior

Before we even ask fake people, we build a "prior expectation" for each council using only hard data: previous election results, national polling swing, Brexit vote, deprivation index, ethnic composition. This gives us a best guess with no AI involved.

We compare the AI's answer against this prior. Where they agree, we're confident. Where they disagree, one of them is wrong and we flag it for review.

### Step 9: Sensitivity test

We run the entire model six times with different assumptions: Reform stronger, Reform weaker, Green surging, strong anti-incumbent mood, status quo, and our baseline. If a council gives the same winner in all six runs, the projection is robust. If it flips, it's genuinely uncertain. Out of 136 councils, 135 are stable. Only Croydon changes winner depending on assumptions.

---

## What we tested it against

We ran the model (without the AI, just the statistical corrections) against 38 councils where we already know the real result from 2022-2024.

- **In-sample accuracy:** 76% (29/38 correct winners)
- **Out-of-sample accuracy** (leave-one-out, the honest number): 71% (27/38)

The 5-point gap means our corrections are slightly over-fitted to the councils we built them for. We're honest about this.

The main misses: local independents we can't predict (Havering, Tower Hamlets), outer London marginals where Labour and Conservative are within 3 points, and one council where the same party has two different labels (Labour vs Labour Co-op).

For comparison, the simplest possible model (just apply national polling swing to every council uniformly) predicts Reform winning 98 out of 136 councils. That's obviously wrong. Our model predicts Reform 45, Labour 37, Lib Dem 24, Conservative 21, Green 9. That's plausible.

---

## What data we use

| Data | What it tells us | Where it's from |
|------|-----------------|-----------------|
| Census 2021 | Who lives where | ONS |
| Previous election results | What happened last time | Wikipedia, BBC |
| National polls | The current national mood | Published polling averages |
| Brexit referendum 2016 | Which areas lean Reform | Electoral Commission |
| Deprivation index | Which areas are struggling | ONS |
| Ethnic composition | Where the Gaza effect applies | Census 2021 |
| Candidate counts | Which parties are actually standing | Democracy Club |
| By-election results | Calibration corrections | Published results |

---

## What could go wrong

| Risk | How likely | What we do about it |
|------|-----------|-------------------|
| Fake people don't vote like real people | Unknown | This is the fundamental untested assumption |
| Tactical voting (people coordinating) | High | Blend with statistical model |
| Late campaign event shifts everything | Moderate | Run predictions as late as possible |
| Green Party still underpredicted | High | National correction + surge councils |
| Brexit-Reform link has decayed since 2016 | Moderate | Dampened coefficient, capped |
| No historical data for 98/136 councils | High | National polls + demographics + Brexit data |
| Our corrections are tuned to London councils | High | LOO cross-validation measures the overfitting |
| A popular independent candidate we can't predict | High | Nothing we can do. Acknowledged. |

---

## The numbers

- **65,000** synthetic personas
- **136** councils projected
- **200** personas per council
- **14** correction layers
- **6** external datasets
- **4,031** lines of code
- **76%** in-sample accuracy (backtest)
- **71%** out-of-sample accuracy (cross-validation)
- **135/136** councils stable across parameter variations
- **1** genuinely uncertain council (Croydon)

---

## What happens on election night

We've identified the 20 councils that declare first (Sunderland at 11:30pm, then London boroughs, then northern cities by 4am). We have 8 bellwether councils with specific things to watch for ("if Reform wins Sunderland by 15+ points, they're having a great night nationally"). We have a live tracker that records real results against our projections as they come in. And we have three pre-written analysis pieces (great result, decent result, poor result) ready to publish within hours.

Our success criteria, published before the election:
- 80%+ accuracy on councils where we said "Confident"
- National vote shares within 4 points per party
- 65%+ overall winner accuracy

If we miss these targets, we publish exactly what went wrong and why. The pre-registration hash proves we didn't change our projections after seeing results.

---

*This is a scientific experiment with a publicly stated hypothesis, not a marketing exercise with retrofitted claims.*
