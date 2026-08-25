# Speaker Script (with Transitions) — Lexicographic Multi-Preference Routing
### SIGSPATIAL '26 · ~15 min talk + 5 min Q&A · 13 slides + 1 Q&A backup

**How to use this:** each slide has what to SAY, one **bold takeaway** to land, and a
→ **BRIDGE** line — the sentence you say *while advancing* to the next slide. The
bridges are what turn slides into a story. Rehearse the bridges until they're
automatic; they're where the talk flows or stalls.

Practice to **13 minutes**. Land the bold sentence on every slide.

---

## Slide 1 — Title (~30s)
"Good [morning/afternoon]. I'm Abdallah Abdelwahab, from the University of Alabama,
with Ahmad Alsharif and Mahmoud Mahmoud. Our work is about giving people real
control over *how* their route is chosen — by letting them rank what matters,
instead of accepting a single travel-time number."

→ **BRIDGE:** "Let me start with why that control is missing today."

---

## Slide 2 — The Problem (~1 min)
"Every navigation app folds distance and traffic into one travel-time estimate, and
gives you a handful of on/off switches — avoid tolls, avoid highways, avoid ferries.
That's it.

But that's not how people think about routes. A parent driving at night wants the
safest route above all. A courier wants the fastest route that still avoids
congestion. A cyclist wants to dodge signal-heavy streets. **None of these people
can express that today — and they can't say which concern wins when two of them
conflict.**"

→ **BRIDGE:** "So the real question is: how *should* people express what they want?"

---

## Slide 3 — Our Insight (~1 min)
"Here's our observation. People don't think in weight vectors — nobody says 'safety
0.6, traffic 0.4.' They think in a *priority order*: 'safety first, then the usual
way.'

So that's exactly what we let them do. **Rank your criteria, and we return one route
that honors that ranking — never trading a higher-priority preference for a lower
one.** That word *honors* is the whole paper."

→ **BRIDGE:** "Simple to say — but two things make it hard to actually deliver."

---

## Slide 4 — Two Obstacles (~1 min)
"First, the criteria aren't on the map. Safety, popularity, congestion — these live
in separate external datasets, each in its own units. We have to put them on common
ground.

Second, we have to honor an *order*, fast. And this is where prior work falls short:
it either collapses the criteria into a weighted sum — which can silently override
the very thing you cared about most — or it hands you a Pareto menu of trade-offs and
makes you pick. **Neither honors a stated priority order. We return one route,
respecting the order, at interactive speed.**"

→ **BRIDGE:** "Here's how the system clears both obstacles."

---

## Slide 5 — System Overview (~1.5 min)
"Two halves. Offline, on the left: we take four open data sources — OpenStreetMap,
NYC crash records, historical trips — and turn them into one annotated road graph.
We also precompute heuristic tables over a grid of tiles. This happens once.

Online, on the right: the user gives a query — origin, destination, ranked
preferences. If traffic matters, we pull live speeds from TomTom at query time. Then
the search returns the best priority-respecting route. **Everything offline is
computed once and reused for any preference order — so at query time there's no
recomputation.**"

→ **BRIDGE:** "Let's zoom in on the first half — how four different datasets become
comparable."

---

## Slide 6 — Preferences → Scores (the payoff) (~1.5 min)
"Every source becomes a score between 0 and 1 on each road segment, and a route's
score is the product of its edge scores.

But here's what that *buys* you. These heatmaps are the preference fields themselves.
Look at crash exposure on the left — it concentrates on a handful of corridors. When
a user asks for the safest route, the score pushes the search *away* from those red
zones, so the route physically avoids the crash-dense streets. **The same holds for
congestion and signals — the scoring is what makes the route dodge what you're trying
to avoid.** And because the three fields are distinct, different orderings give
genuinely different routes."

→ **BRIDGE:** "So that's how we score. Now — how does the search actually honor the
*order*?"

---

## Slide 7 — The Method (~1.5 min)
"Two ideas. First, the search itself. Instead of a single score, each route carries a
*vector* — one component per preference. We compare on the top preference first, and
only break near-ties with the next one down. Priorities never multiply together, so a
lower preference can never overrule a higher one. That's 'honoring the order,'
concretely.

Second — making it fast. On the right you see the idea: we overlay a coarse grid of
tiles, and for each tile we precompute an *optimistic estimate* of the best score to
the destination. That's an admissible A* heuristic — it focuses the search, and
**it's where the 80% node reduction comes from.** We prove it's admissible, which
makes the search provably optimal — proofs are in the paper."

→ **BRIDGE:** "So the method honors the order and it's fast in principle. Does it hold
up in practice? Three results."

---

## Slide 8 — Result 1: The Order Matters (~1 min)
"This is one origin–destination pair, routed under four different priority orders.

Look how different they are. The safety-first route swings wide to avoid crash-heavy
corridors. The historical route follows the popular avenue. The traffic routes take
yet another path. **These three offline routes are almost completely edge-disjoint —
under three percent overlap. So the priority order isn't re-weighting one route; it's
producing a fundamentally different one.**"

→ **BRIDGE:** "Different routes — good. But is it fast, and is it still optimal?"

---

## Slide 9 — Result 2: Fast + Optimal (~1.5 min)
"FreqA* answers a query in 0.42 milliseconds — a 97.8× speedup over the same-objective
baseline, which takes 41. It does that by expanding 80% fewer nodes, because the tile
heuristic focuses the search.

And — this is the part I want to stress — that speed costs *nothing* in quality. On
every one of our 197 test queries, FreqA* returns the *provably optimal* route.
FreqScore of 100%. **So we're not trading accuracy for speed — we get both.**"

→ **BRIDGE:** "That's on offline data, which is reproducible. But real traffic
changes minute to minute — does the same search handle *live* conditions?"

---

## Slide 10 — Result 3: Live Traffic (~1 min)
"Because the traffic score can come from a live feed, the same query reroutes as
conditions change. Here's a traffic-first query at two different times of day.

The routes differ — each keeps to the green, free-flowing edges and steers around the
red, congested corridors that were jammed at that moment. This is re-evaluated live,
not cached. **And the per-query search still runs in 2.2 milliseconds — the live data
changes the answer without slowing anything down.**"

→ **BRIDGE:** "So we can honor a preference, fast, live. But honoring one has a
cost — and users should be able to control it."

---

## Slide 11 — Result 4: Cost of Preference + Budget (~1.5 min)
"Honoring a preference makes the route longer than the shortest path — safety-first is
about 60% longer. But that's not inefficiency; it's the real cost of avoiding
crash-prone roads, and users deserve to *see* that trade-off, not have it hidden.

And we give them a knob. An optional distance budget caps the detour at a multiple of
the shortest path — one feasibility check, no change to the search. **At a budget of
2×, we halve the detour while keeping 85% of the safety.** Tighten it further and you
trade more safety for directness — transparently. The user is in control."

→ **BRIDGE:** "Now, I want to be honest about the boundaries of what we've shown."

---

## Slide 12 — Scope & Honesty (~1 min)
"Three things to be upfront about. Our historical corpus is synthetic — a stand-in for
popularity — so our quality numbers measure optimality against our own field, not
against real driver choices. Our safety score is crash *exposure*, not per-vehicle
risk, so that 60% detour is likely an upper bound. And we evaluate on one city.

**But — and this matters — none of that touches the search guarantees or the
efficiency results. Those hold for any preference field you put on the graph.** The
limitations bound how we interpret the routes, not whether the method works."

→ **BRIDGE:** "So let me bring it back to the one idea worth remembering."

---

## Slide 13 — Takeaway (~45s)
"The idea is simple: users should be able to say 'safest, then fastest' — and get back
a provably optimal, priority-respecting route, in under a millisecond, built entirely
from open municipal data, that adapts to live traffic.

Everything — code and data — is on GitHub. Thank you. I'm happy to take questions."

*(Then stop. Don't fill silence. Let the chair open Q&A.)*

---

# Q&A — prepared answers
*(Repeat each question before answering — buys time, helps the room hear it.)*

**Q: Why not a weighted sum / just learn the weights?**
"A weighted sum forces one fixed trade-off and can override the user's top priority
when scores are close. A priority order guarantees the top preference wins. They
answer different questions — ours honors a *stated* order rather than learning a fixed
combination."

**Q: Isn't the synthetic historical data a problem?**
"It's a stand-in for popularity, and our guarantees don't depend on it — a real
trajectory corpus drops in and changes only the frequency table, not the algorithm or
its optimality. Grounding it in real trips is our top future-work item."

**Q: The live-traffic runtime — I've heard it's minutes?**
"That's the region-wide data fetch and heuristic refresh — a background, amortizable
cost shared across all queries to a region, like any production traffic feed. The
per-query *routing* is 2.2 milliseconds."

**Q: Does it generalize beyond Manhattan's grid?**
"The method is topology-agnostic — the tile heuristic and the search make no grid
assumption. We evaluated on Manhattan; testing irregular networks is future work."

**Q: How does ε affect optimality?**
"At ε=0 it's strictly lexicographic and provably optimal. We use a small ε=0.05 so
secondary preferences can act on near-ties; there, primary-preference optimality is
still 99.5%. It's a tunable knob between strict priority and secondary influence."

**Q: Strict vs. bounded priority — which is it?**
"Strict at ε=0. At our operating tolerance it's *bounded* priority — a lower
preference can only act where the higher ones agree to within ε. We're precise about
that in the paper."

**Q: How do you compare against learned / RL multi-objective routers (e.g. Sarker)?**
"They optimize a learned trade-off fixed at training time and don't honor a stated
priority order or give guarantees — it's a different problem framing. An empirical
comparison is future work. (See the backup slide.)"
