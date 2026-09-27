# Compute Crossover Model — Project Spec

_Living document. Updated as the project develops. Companion data file: `tech_trends_long.csv`._

## 1. Origin and core idea

The starting idea (30-40 years old, first sketched at the ALU/coprocessor level):
computing could be organized like physical manufacturing — complex jobs broken into
parts, shipped to specialized "compute factories," and reassembled — with brokers /
directories acting as a "yellow pages" for finding the right compute service, and a
message-passing "highway" for routing requests and responses.

Key refinements made during discussion:

- **Move compute to data, not data to compute.** This is the actual dominant real-world
  pattern (Hadoop/MapReduce data locality, in-database compute, edge computing), and it
  sidesteps the "who do you trust with your data" problem the original broker idea had,
  turning it into a more tractable code-supply-chain trust problem instead.
- **The ALU-level history is the same problem at smaller scale.** Register-width
  operations have low arithmetic intensity relative to transport cost, so they're never
  worth offloading; bulky, high-intensity operations (image/video, later GPU workloads)
  are. This is the same crossover logic that later reappears as MQ/message-passing
  architectures. The formal name for the underlying ratio is **arithmetic intensity**
  (FLOPs per byte moved), central to the **roofline model** in performance engineering.

## 2. The crossover threshold model

Core model: **local cost** and **offload cost** are both increasing functions of task
size (heaviness) — local has ~zero fixed overhead but a steep slope; offload has a
higher fixed overhead (connection setup, latency, minimum transport cost) but a
flatter slope (specialized/parallel remote compute, bulk transport efficiency). The two
lines cross exactly once; past that point, larger tasks favor offloading.

Two earlier draft versions of this chart contained real errors, since corrected:

1. Axis-direction error (cost appeared to decline with task size instead of rising).
2. Modeling error (offload cost was drawn as monotonically *decreasing* with size,
   which is physically wrong — total cost must rise with size, just at a flatter rate
   than local cost, for a real breakeven to exist).

**Where the crossover point sits is not fixed** — it depends on the relative
improvement rates of several independent dimensions (storage, network, compute), and
those rates diverge over time (e.g. Moore's Law-era compute density vs. Dennard-scaling
breakdown vs. Nielsen's-Law-era bandwidth growth), so the crossover point drifts over
decades rather than staying still.

## 3. Forecasting framework

- A minimal working dimension set: **Storage** (cost, speed), **Network** (cost,
  speed), **Compute** (cost, speed) — each further split where a single "speed" or
  "cost" label turned out to hide two genuinely different populations (see §4).
- **Velocity (rate of change) matters more than the current value** for predicting how
  the crossover point will move; and the *rate of the rate* (acceleration / steady /
  deceleration) matters more still over a 5-10 year horizon.
- Considered: damped-trend forecasting (Holt's linear trend method), regime
  classification instead of raw 2nd-derivative estimates (more robust to noisy data),
  and a variable sliding window (3-7 years as a starting point, ideally
  changepoint-detected rather than fixed) for regime-shift detection.
- **Established fields doing versions of this already:**
  - Technology forecasting (Farmer, Nagy, Trancik, Lafond @ Santa Fe Institute / Oxford
    INET) — Wright's law vs. Moore's law tested against 62-470+ real technologies;
    forecast error grows ~2.5%/year (log-error) with horizon.
  - Econometric regime-switching dynamic factor models — compress many macro variables
    into a few latent trend factors with discrete regime states, detected from data
    rather than assumed.
  - Semiconductor industry roadmapping (ITRS → IRDS) — a real, decades-old, rolling
    15-year multi-dimensional roadmap across ~12 technical fronts, self-correcting
    against a 5-year actuals window. Renamed ITRS→IRDS in 2016-17 specifically because
    Moore's Law deceleration forced a broader model.
  - Weather/climate ensemble forecasting and chaos theory (Lorenz) — some of these
    dimensions may behave more like the atmosphere (chaotic, bounded predictability
    horizon) than like GDP (smooth, gently-degrading forecast error); worth asking
    per-dimension which regime applies.

## 4. Data file: `tech_trends_long.csv`

Tidy/long format — one row per observation. Chosen over a wide matrix so the
(genuinely ragged) data doesn't force NaNs; pivot to a wide matrix or an `xarray`
tensor only at analysis time, per question.

**Columns:** `time, domain, metric, value, low_estimate, high_estimate,
uncertainty_type, unit, note, source`

**Domains:** `storage`, `network`, `compute`, `climate_risk`

**Deliberate metric splits** (same domain, different populations — do not average
across these):

| Domain | Split | Why |
|---|---|---|
| compute | `speed` (clock GHz) vs `speed_gpu` (flagship GPU FP32 GFLOPS) | Clock speed ~plateaued since 2005 (Dennard scaling breakdown); GPU FLOPS kept compounding ~40%/yr. Same word "speed," two unrelated trend lines. |
| network | `cost` (home broadband, $/month) vs `cost_backbone` (wholesale IP transit, $/Mbps) vs `cost_leased_line` (legacy T1/leased circuits) | Home broadband cost is roughly flat for decades (~$20-50/mo, service-priced); backbone cost fell steeply (~$1200→~$0.07/Mbps, 1998-2025, manufacturing/capacity-priced); leased lines are now a *legacy* technology whose $/Mbps is rising again as it's phased out — a real (not artifactual) cost reversal driven by shrinking economies of scale. |
| network | `speed` (home, kbps/Mbps) vs `speed_ethernet` (datacenter standard link rate) | Different population entirely; Ethernet standard cadence is itself accelerating (6 speeds in first 30 years, next 6 in <10 years) even as % annual growth decelerates. |

**`uncertainty_type` values in use:**
- `none stated` — plain historical figure, no error bar in the source.
- `90% CI` — a real statistical confidence interval (e.g. Epoch AI's dollar-training-cost
  threshold-year forecast: 2032, 90% CI 2031-2036).
- `IPCC scenario range (not a probability - two named emissions pathways)` — a
  scenario branch conditioned on a policy/emissions choice, explicitly *not* a
  probability distribution. Currently used for the two 2052 cable-landing-station
  exposure rows (SSP1-2.6 vs SSP5-8.5).

**`climate_risk` domain** — structurally different from the other three: milestone
snapshots at named future years (not continuous rate series), heterogeneous units
(USD/year, USD total, miles of conduit, facility counts, %), not currently amenable to
the same log-growth-rate analysis as storage/network/compute. Current rows:
- `cost_overlay` / `cost_overlay_cumulative` — WEF/Accenture data-center climate cost
  forecast ($81B/yr by 2035, $168B/yr by 2065, $3.3T cumulative by 2055).
- `infrastructure_exposure` — Durairajan et al. 2018 (4,400 miles US fiber conduit +
  1,100 colocation facilities at flood risk by 2030); Clare et al. 2023 (50-97% of
  global cable landing stations projected >500mm sea-level rise by 2052, depending on
  emissions scenario).

## 5. Methodology: rate-of-change analysis

- **Annualized rate** between two points: `(ln(v2) - ln(v1)) / (t2 - t1) * 100` (%/year,
  continuous log-growth — comparable across series regardless of native units, since
  cost/speed trends here are multiplicative, not additive).
- **Acceleration/deceleration diagnosis (crude but workable):** compare the full-period
  average rate to the most-recent-interval rate.
- **Caution:** sparse or heterogeneous-source series (few points, inconsistent
  measurement conventions between sources) can produce large, spurious single-interval
  rates — e.g. an early version of `network/cost` showed a -78.6%/yr "recent" rate that
  was a one-year artifact of comparing two different pricing tiers, not a real trend.
  Treat any 2-3-point series's computed rate with real caution.

## 6. Key findings so far

- **Storage cost** and **compute cost/GFLOPS** both show strong, externally-corroborated
  deceleration (storage: ~-41%/yr full-period → ~-7%/yr recently, matching the
  industry-recognized "Kryder's Law slowdown"; compute cost similarly ~-44%/yr →
  ~-7%/yr).
- **Compute clock speed** is essentially flat since 2005 (~2%/yr) — confirms the
  Dennard-scaling-breakdown story independently.
- **Compute GPU FLOPS** shows no deceleration in the data collected so far (~41%/yr
  full-period vs. ~42%/yr recent) — the real compute-growth story of the last 15+ years
  lives almost entirely in this axis, not clock speed.
- **Network home cost** is nearly flat for decades; **network backbone cost** is falling
  as steeply as storage/compute — same domain, opposite behavior, because one is a
  market-priced subscription service and the other is a manufacturing/capacity-priced
  commodity.
- **Open, unresolved question:** is the storage/compute cost deceleration *structural*
  (approaching real physical limits — areal density, transistor scaling) or a
  *measurement-convention artifact* (early data = record-setting bleeding-edge
  configurations, recent data = broad commodity-fleet averages, which is an inherently
  slower-moving quantity)? Not yet resolved — would need an internally consistent
  like-for-like series to test.
- **Forecast availability is asymmetric across domains:** speed/capacity roadmaps are
  common and often vendor-committed (an engineering target the vendor controls); cost
  forecasts are rarer and usually only exist as third-party trend extrapolations —
  except wholesale network bandwidth, where cost forecasting is standard commodity-
  market practice (TeleGeography publishes "historical and projected" pricing
  directly).
- **Climate-risk forecasts** are, surprisingly, among the most specific and explicitly
  scenario-conditioned in the whole dataset (named years, named emissions pathways) —
  more rigorous in their uncertainty framing than most pure-technology forecasts found.

## 7. Open questions / possible next steps

- Resolve the storage/compute cost-deceleration structural-vs-artifact question with a
  cleaner like-for-like series.
- Look for a *continuous* (year-by-year) climate-cost or exposure trajectory, rather
  than milestone snapshots, so it can go through the same rate-of-change analysis as
  the other domains.
- Consider whether specialized compute units (GPU, DSP, future NPU/etc.) generalize
  into grouped "primitive-operation families" (per the BLAS/OpenCL/MLIR precedent)
  rather than one bespoke axis per device type.
- Decide whether `network/cost_leased_line`'s reversal deserves a 4th data point in the
  2010s to make the reversal show up numerically in the rate table, not just in a note.
- If this file gets large, consider splitting `climate_risk` into its own file, since
  its shape (milestone/scenario snapshots) is already structurally distinct from the
  other three domains.

## 8. File inventory

- `tech_trends_long.csv` — the data, tidy/long form, 90 rows as of this spec's last
  update.
- `spec.md` — this file.
