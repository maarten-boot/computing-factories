# Day2

Me:

While it is still very abstract to me I have started pondering about a minimal set of values and how to write them down.

Initially I would include both the speed and the cost of an item and note that in the development and progress of many items in IT the flow is almost often step wise and with a higher cost at the beginning of a new capacity as it is not yet common.

my initial set would look like

1. Storage: Cost and Speed although capacity of slower media often greatly outnumbers modern faster storage units.
2. Network bandwidth and cost (Transport would also be a good term here)
3. Compute: Speed and Cost, forgetting temporarily about specialized compute units as the initial trend was to move specialized compute units on the same die.
4. Time as a common for all 6 above

If I remember well, a matrix would be the most appropriate for multidimensional representations.

And starting from known historical Cost and Speed/Bandwidth numbers we could graph each individual item both discrete in the steps they  manifested and a synthetic hypothetical trend line.

The next steps would be the change rate and the acceleration.

S: storage
N: network
C: compute

c: cost
s: speed
t: time

Ss Storage speed, Sc storage cost, and similar: Ns, Nc, Cs, Cs,

You:

The matrix instinct is right, and it's worth being precise about the shape, because your six variables actually have two independent axes buried in the notation (domain: S/N/C, and metric: speed/cost), which a flat matrix can either preserve or accidentally erase depending on how you lay it out.

**The base structure**

At its simplest, each historical time step t gives you a row: `[Ss(t), Sc(t), Ns(t), Nc(t), Cs(t), Cc(t)]`. Stack rows over T time steps and you get a T×6 matrix — this is literally the standard input shape used in the econometric regime-switching models from a moment ago (the FRED-MD dataset those papers use is exactly a matrix of months × economic variables). But because your six columns are really 3 domains crossed with 2 metrics, you may get more mileage treating it as a small tensor — T × 3 × 2 (time, domain, metric) — rather than flattening to T×6. The tensor form lets you slice cleanly: fix the metric axis and you get "cost across all three domains over time" (useful for the crossover comparisons we've been drawing); fix the domain axis and you get "storage's cost and speed together" (useful for a per-domain S-curve). Flattening to T×6 loses that slicing convenience, though it's easier to drop into a spreadsheet, so which one you want depends on whether you'll mostly query by domain, by metric, or both.

**Velocity and acceleration stack as additional layers, not new columns**

Once you have the T×6 (or T×3×2) level matrix, first-differencing it gives you a velocity matrix of the same shape (one row shorter), and differencing that again gives acceleration. So the natural object isn't one matrix but three, stacked: level, velocity, acceleration — each T×6. Your regime buckets (accelerating/steady/decelerating) become a simple sign-and-threshold read on the acceleration layer, cell by cell.

**One correction that matters before you fill in real numbers: work in log space, not raw units.** Cost and speed for these domains change multiplicatively, not additively — storage cost per GB doesn't fall by a fixed dollar amount each year, it falls by a roughly fixed *percentage* each year (this is exactly the Moore's/Wright's-law shape from earlier, and it's why the Santa Fe Institute group's forecast-error number was stated as a logarithmic error, not a raw one). If you difference raw values, your "velocity" will look wildly different in magnitude across domains just because the units differ (dollars per GB versus Mbps), and it'll be non-constant even when the underlying trend is a clean exponential. If instead your matrix stores log(Ss), log(Sc), etc., a first difference becomes an approximate percentage growth rate — directly comparable across all six columns regardless of their native units — and a second difference tells you whether that percentage rate is itself speeding up or slowing down. That's the representation that makes "compare acceleration across storage vs. network vs. compute" a meaningful operation rather than an apples-to-oranges one.

**Two definitional traps worth settling before the numbers go in**

First: does "cost" mean total cost, or cost per unit of capacity/throughput? It needs to be per-unit ($/GB, $/Mbps, $/FLOP) for the matrix to mean anything consistent over time, since total cost conflates "getting cheaper per unit" with "we're buying more units." This connects directly to your own observation about old slow media outnumbering new fast media in raw capacity — that's a real, well-studied pattern (technology substitution / diffusion curves, the classic example being SSD vs. HDD: SSD cost-per-GB fell steeply for two decades following its own Wright's-law curve while installed HDD capacity kept dominating for years on legacy stock alone). If "cost" isn't unit-normalized, that substitution dynamic gets tangled into your cost column instead of being a separate, visible phenomenon.

Second: "speed" may need splitting the same way network already forced you to split bandwidth from latency. Storage has an equivalent split — throughput (MB/s) versus access latency (seek time for spinning disks, effectively near-zero but not zero for SSD) — and these have historically moved at very different rates, the same way network bandwidth and latency did. Whether that's worth a 7th/8th column now or a note to revisit later is your call, but it's the same trap you already caught once.
