# Speaker Script — Lexicographic Multi-Preference Routing
### SIGSPATIAL '26 · target 15 min talk + 5 min Q&A · ~13 slides

Timing guide: aim for ~1 to 1.5 min per slide. Practice to **13 minutes** so you have buffer.
Bold = the one sentence you must land on each slide.

---

## Slide 1 — Title (~30s)
"Good [morning/afternoon]. I'm Abdallah Abdelwahab, from the University of Alabama. This is joint work with Ahmad Alsharif and Mahmoud Mahmoud. Our paper is about giving people real control over *how* their route is chosen — by letting them rank what matters, instead of accepting a single travel-time number."

*(Don't read the title aloud verbatim — the slide does that. Set the hook.)*

---

## Slide 2 — The Problem (~1 min)
"Every navigation app you use folds distance and traffic into one travel-time estimate, and then gives you a handful of on/off switches — avoid tolls, avoid highways, avoid ferries. That's it.

But that's not how people actually think about routes. **A parent driving at night wants the safest route above all. A courier wants the fastest route that still avoids congestion. A cyclist wants to dodge signal-heavy streets.** None of these people can express that today — and, crucially, they can't say which concern *wins* when two of them conflict."

---

## Slide 3 — Our Insight (~1 min)
"Here's our key observation. People don't naturally express preferences as a *weight vector* — nobody thinks 'safety 0.6, traffic 0.4.' They express them as a **priority order**: 'safety first, then the usual way.'

So that's exactly what we let them do. Rank your criteria, and we return one route that honors that ranking — and never trades a higher-priority preference for a lower-priority one. That word *honors* is the whole paper."

---

## Slide 4 — Two Obstacles (~1 min)
"Two things make this hard.

First, the criteria aren't on the map. Safety, popularity, congestion — these live in separate external datasets, each in its own units. We have to put them on common ground.

Second, we have to honor an *order*, fast. Prior work does one of two things here — it either collapses the criteria into a weighted sum, which can silently override the very thing the user cared about most, or it hands the user a Pareto menu of trade-offs and makes them pick. **Neither honors a stated priority order. We return one route, respecting the order, at interactive speed.**"

*(This is your entire 'related work' — one sentence of contrast, no survey. If someone asks about specific papers, you have a backup slide.)*

---

## Slide 5 — System Overview (~1.5 min)
"Here's the whole system. It has two halves.

Offline, on the left: we take four open data sources — OpenStreetMap, NYC crash records, historical trips — and convert them into one annotated road graph. We also precompute heuristic tables over a grid of tiles. This happens once.

Online, on the right: the user gives a query — origin, destination, and their ranked preferences. If traffic matters, we pull live speeds from TomTom at query time. Then the search runs and returns the best priority-respecting route.

**The key property: everything offline is computed once and reused for *any* preference order — so at query time there's no recomputation.**"

*(Point at the flow: data → graph → search → best path. Don't read every box.)*

---

## Slide 6 — Preferences on a Common Scale (~1 min)
"So how do four different datasets become comparable? Each one becomes a score between 0 and 1 on every road segment. A route's score for a preference is just the product of its edge scores — so the best route is a max-product search.

On the left you see the three fields over Manhattan: crash exposure, historical usage, signal density. **The important thing is that they're visibly *different* — they light up different parts of the city. That's exactly why changing the priority order gives you a genuinely different route, not a cosmetic tweak.**"

---

## Slide 7 — The Method (~1.5 min)
"The method is Lexicographic FreqA*, and it comes down to two ideas.

First — the search itself. Instead of a single score, each route carries a *vector* — one component per preference. We compare on the top preference first, and only break near-ties with the next one down. Priorities never get multiplied together, so a lower preference can never overrule a higher one. That's what 'honoring the order' means, concretely.

Second — making it fast. On the right you see the idea: we overlay a coarse grid of tiles on the city, and for each tile we precompute an *optimistic estimate* of the best score to the destination. That's an admissible A* heuristic — it focuses the search, and **it's where the 80% node reduction comes from.** We prove it's admissible and consistent, which makes the whole search provably optimal — proofs are in the paper. And because it's precomputed per destination tile, the same tables serve *any* ordering, with no recomputation."

*(Point at the road-graph → tile-graph picture when you say 'coarse grid of tiles.' Say 'proofs in paper' and move on — do NOT walk through them.)*

---

## Slide 8 — Result 1: The Order Matters (~1 min)
"Now results. This is one origin–destination pair, routed under four different priority orders.

Look at how different they are. The safety-first route — in blue — swings wide to avoid crash-heavy corridors. The historical route follows the popular avenue. The traffic routes take yet another path. **These three offline routes are almost completely edge-disjoint — under three percent overlap. So the priority order isn't re-weighting a route; it's producing a fundamentally different one.**"

---

## Slide 9 — Result 2: Fast + Optimal (~1.5 min)
"And it's fast. FreqA* answers a query in **0.42 milliseconds** — a 97.8× speedup over the same-objective baseline, which takes 41 milliseconds. It does that by expanding **80% fewer nodes**, because the tile heuristic focuses the search.

And — this is the part I want to stress — that speed costs *nothing* in quality. On every one of our 197 test queries, FreqA* returns the *provably optimal* route. FreqScore of 100%. **So we're not trading accuracy for speed — we get both.**

The table shows it against the baselines. Dijkstra and A* are there only as references — they solve the easier shortest-distance problem."

---

## Slide 10 — Result 3: Live Traffic (~1 min)
"Because the traffic score can be read from a live feed, the same query reroutes as conditions change. Here's a traffic-first query at two different times of day.

The routes are different — each one keeps to the green, free-flowing edges and steers around the red, congested corridors that were jammed at that moment. And this is re-evaluated live, not cached. **The per-query search still runs in 2.2 milliseconds — the live data changes the answer without slowing anything down.**"

*(If asked in Q&A about the 5-min refresh: that's a per-region background cost, amortized across queries, like any traffic feed. The routing itself is 2.2 ms.)*

---

## Slide 11 — Result 4: Cost of Preference + Budget (~1.5 min)
"Honoring a preference has a price: preference-first routes are longer than the shortest path. Safety-first is about 60% longer. But that's not inefficiency — it's the real, visible cost of avoiding crash-prone roads. We think users deserve to *see* that trade-off rather than have it hidden.

And we give them a knob. An optional distance budget caps the detour at some multiple of the shortest path — one feasibility check, no change to the search. **At a budget of 2×, we halve the detour while keeping 85% of the safety.** Tighten it further and you trade more safety for directness — transparently. The user is in control, not a fixed hidden compromise."

---

## Slide 12 — Scope & Honesty (~1 min)
"I want to be upfront about what we claim and what we don't.

Our historical corpus is synthetic — a stand-in for popularity — so our quality numbers measure optimality against our own field, not against real driver choices. Our safety score is crash *exposure*, not per-vehicle risk, so that 60% detour is likely an upper bound. And we evaluate on one city.

**But — and this matters — none of that touches the search guarantees or the efficiency results. Those hold for any preference field you put on the graph.** The limitations bound how we interpret the routes, not whether the method works."

---

## Slide 13 — Takeaway (~45s)
"So, to close. The idea is simple: users should be able to say 'safest, then fastest' — and get back a provably optimal, priority-respecting route, in under a millisecond, built entirely from open municipal data, that adapts to live traffic.

Everything — code and data — is on GitHub. Thank you. I'm happy to take questions."

---

# Likely Q&A — prep answers

**Q: Why not just use weighted sums / learn the weights?**
"A weighted sum forces one fixed trade-off and can override the user's top priority when scores are close. A priority order guarantees the top preference wins. They answer different questions — ours honors a *stated* order rather than learning a fixed combination."

**Q: Isn't the synthetic historical data a problem?**
"It's a stand-in for popularity, and our search guarantees don't depend on it — a real trajectory corpus drops in and changes only the frequency table, not the algorithm or its optimality. Grounding it in real trips is our top future-work item."

**Q: The 5-minute live-traffic runtime?**
"That's the region-wide data fetch and heuristic refresh — a background, amortizable cost shared across all queries to a region, exactly like any production traffic feed. The per-query *routing* is 2.2 ms."

**Q: Does it generalize beyond Manhattan's grid?**
"The method is topology-agnostic — the tile heuristic and the search make no grid assumption. We evaluated on Manhattan; testing irregular networks is future work, and we'd expect the heuristic's benefit to vary with how the tiles align to the street pattern."

**Q: How does ε affect optimality?**
"At ε=0 it's strictly lexicographic and provably optimal. We use a small ε=0.05 so secondary preferences can act on near-ties; at that point primary-preference optimality is still 99.5%. It's a tunable knob between strict priority and secondary influence."

**Q: Bounded vs strict priority — which is it?**
"Strict at ε=0. At our operating tolerance it's *bounded* priority — a lower preference can only act where the higher ones agree to within ε. We're precise about that in the paper."
