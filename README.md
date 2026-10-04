# Queuechella – Discrete-Event Simulation of a Music Festival

A discrete-event simulation (Python) of a two-day music festival, built to test whether
operational investments are worth making. Two improvement alternatives are compared
against the current setup using paired confidence intervals with a Bonferroni correction.

*Course project – Simulation, Industrial Engineering & Management.*

**Key result:** Of two investment alternatives, only Alternative A (more stage capacity plus popular
mainstream acts, ₪950K) significantly raised both revenue (+12%, about ₪136K per festival) and visitor
satisfaction (+0.3 points). Alternative B (₪850K) raised satisfaction strongly (+1.3) but not revenue.
If the cost is a one-time investment, A pays back in about 7 festivals.

## Business question

The festival management is considering two investment packages. Which one improves
**total revenue** and **average visitor satisfaction** the most compared with the current
operation?

## What was built

**Input modeling**
- Fitted distributions to historical data: exponential for friends-group inter-arrival
  times (λ = 0.72927, MLE) and normal for main-stage show duration (MLE).
- Validated the fits with histograms, QQ plots, CDF comparisons and Kolmogorov–Smirnov tests.
  The exponential fit passes at the 5% level. The normal fit for show duration is rejected at 5% but
  not at 1%, and was accepted together with the graphical checks.
- Implemented custom random-variate generators from uniform random numbers
  (inverse transform, acceptance–rejection and others) for the remaining system
  parameters: service times, stage durations, food choices, satisfaction events, and more.

**Simulation engine**
- Event-driven simulator on a priority queue (`heapq`) with a global clock.
- Visitor entities: single visitors, couples and friend groups, each with its own
  arrival pattern and route through the festival.
- System components: entrance with ticket scan and security check (5 clerks), main stage,
  side stage and DJ stage with capacity limits, photo stations, phone charging, merchandise
  tent, body-art stand, and a food court with three restaurants.
- Behavioral logic: queue abandonment when patience runs out, overnight stays, and
  satisfaction updates from shows, waiting times and food quality.

**Experiment design**
- Performance measures: total festival revenue and average visitor satisfaction.
- Pilot of 10 runs to estimate variance (α = 0.1, 10% relative precision). The pilot
  suggested a very small sample, but 30 replications were used for every scenario.
- Common random numbers: all scenarios use the same seeds, so each run is compared with
  its matching base-case run.
- Six paired t-based comparisons with a Bonferroni correction (overall 90% confidence,
  α = 0.0167 per test).

## Scenarios

| Scenario | Changes | Stated cost |
|---|---|---|
| **Base case** | Current operation | – |
| **Alternative A** | Stage capacity +30%; investment in popular mainstream acts (higher satisfaction weight and band-shirt purchase probability) | ₪950,000 |
| **Alternative B** | Better kitchen (bad-food probability 0.4 → 0.1); one more photo station (3 → 4) and one more body-art artist (2 → 3); gift bags raise initial satisfaction from 5 to 6.5 | ₪850,000 |

## Results (30 replications per scenario)

| Scenario | Avg revenue (₪) | Avg satisfaction |
|---|---|---|
| Base case | 1,140,225 | 5.29 |
| Alternative A | 1,275,863 | 5.59 |
| Alternative B | 1,152,233 | 6.62 |

Paired differences (90% overall confidence, Bonferroni-adjusted):

| Comparison | Revenue difference (₪) | Satisfaction difference |
|---|---|---|
| A vs. Base | +135,638  [108,657, 162,618] – significant | +0.303  [0.228, 0.377] – significant |
| B vs. Base | +12,008  [−18,014, 42,030] – not significant | +1.334  [1.252, 1.416] – significant |
| A vs. B | +123,630  [87,676, 159,583] – A better | −1.032  [−1.108, −0.956] – B better |

**Recommendation in the analysis:** Alternative A, since it is the only one that significantly
improves both measures over the base case.

## Verification, cost-benefit and robustness (added after the course submission)

**Verification checks.** Three basic checks on the base case all pass: identical seeds
reproduce identical runs, people departed is never below entities departed, and revenue
and satisfaction are positive in every run.

**Cost-benefit.** The statistical comparison does not weigh gains against cost. Assuming a
one-time investment and a repeating revenue gain, Alternative A (₪950K) pays back in about
7 festivals (about 9.4 using the lower end of the confidence interval). Alternative B (₪850K)
has no statistically significant revenue gain, so no reliable payback. If the cost recurs every
festival, neither alternative pays back on revenue alone. These are revenues, not profits, and
satisfaction is not converted to money, so the original recommendation is kept but the
economics should be read alongside it.

**Robustness.** Re-running the notebook in a new session gives slightly different numbers even
with the same seeds, because the code converts sets of strings to lists (for example
`list(self.eligible_stations)`) and set order varies between Python processes. Within a session
everything is consistent, so common random numbers still hold. To check that the conclusions do
not depend on this, the full experiment (30 runs per scenario, all six paired comparisons) was
repeated 5 times with different set orderings. All six comparisons gave the same conclusions in
every repetition: A significantly improves revenue and satisfaction; B significantly improves only
satisfaction; A beats B on revenue and B beats A on satisfaction. The revenue gain of A ranged
from about ₪103K to ₪135K across repetitions (₪136K in the original run). Replacing those
conversions with `list(sorted(...))` would make runs fully reproducible, at the cost of slightly
changing the published numbers, so it was not applied.

## Limitations and next steps

- Only revenue and satisfaction are modeled; profit and the monetary value of satisfaction are not.
- The model is verified with basic consistency checks but not validated against real festival
  operating data.
- A sensitivity analysis on key parameters (for example the satisfaction weights) would
  strengthen the conclusions.

## Running the notebook

**In the browser (no install):**
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OmriBirk/queuechella-simulation/blob/main/queuechella_simulation.ipynb)
then choose Runtime, Run all.

**Locally:**
```bash
pip install -r requirements.txt
jupyter notebook queuechella_simulation.ipynb
```
Then run all cells. A full run takes about a minute. It was tested end to end on Python 3.12.

The first section (distribution fitting) reads `samples_for_simulation.xlsx`, included in this
repository: 100 historical observations each for friends-group inter-arrival times and main-stage
show durations, as provided with the course. If the file is missing, that section prints a note and is
skipped; the simulation itself does not depend on it, because the fitted parameters are set in the
base-case configuration. The notebook also keeps the saved outputs and charts of the original run.

Because of the set-ordering effect described above, a fresh run gives slightly different numbers
than the saved ones (for example, Alternative A's revenue gain was about ₪125K in one fresh run
versus ₪136K in the original), with the same statistical conclusions.
