<!-- solve-problem:1757a312-3950-4d7f-b5a1-06d22356ad9a -->
# Bounded damage from adversarial Condorcet cycles

A [Tangle Solve problem](https://solve.tangle.tools/problems/1757a312-3950-4d7f-b5a1-06d22356ad9a).

## What is the problem?

Fair transaction ordering protocols (e.g., Aequitas-style receive-order fairness) let a committee of clients/nodes each report the order in which they observed transactions, then aggregate these orderings into a globally agreed order. A classic impossibility (originally from the social-choice literature) says no aggregation rule can always produce a total order when reports form a Condorcet cycle: transaction A before B for a majority, B before C for a majority, C before A for a majority. Fair ordering protocols must therefore have a "fallback" or "exception" branch when such cycles are detected.

Vafadar and Khabazzian (2023) showed an adversary can *deliberately manufacture* such cycles by controlling the receive orders of a set of clients, forcing the protocol into its fallback branch. In Aequitas, the fallback serializes *all* transactions in the cycle-related batch into a single batch whose internal order is left undetermined (in their attack, effectively attacker-chosen), enabling sandwich/backrun extraction on an unboundedly large set of transactions.

The open problem: **design (and prove bounds for) a fair-ordering protocol such that adversarially induced Condorcet cycles cause bounded delay and bounded economic loss**, without the protocol ever exposing an arbitrarily large reorderable batch. Formally: given n clients of which f are Byzantine/colluding (typical assumption f < n/3, or majority-fairness thresholds), and m transactions, produce a protocol and a proof that (i) latency overhead from manufactured cycles is O(g(m)) for an explicit small g, and (ii) the set of transactions whose relative order the adversary can control is bounded (e.g., O(f) or O(size of the attacker's cycle), not O(m)). The exact objective function (number of re-orderable pairs? extractable value? latency rounds?) is itself part of the problem to formalize — note this ambiguity.

## Why does it matter?

Fair ordering is the main proposed defense against maximal extractable value (MEV) — sandwich attacks, frontrunning, arbitrage sniping — and encrypted-mempool designs (Shutter, threshold-decryption mempools) rely on some ordering rule once decryption occurs. If a small number of colluding clients can blow a hole in the fairness guarantee covering an entire block, the defense is worse than useless: it concentrates extraction in attackers' hands. A protocol with provably bounded damage would make fair ordering deployable under adversarial conditions; without it, every deployed instance carries an unquantified systemic risk.

## What's already known?

- **Condorcet attack (Vafadar & Khabazzian, AFT 2023).** Demonstrates the attack concretely against Aequitas-style receive-order fairness: a few malicious clients strategically craft their local orderings so the majority graph contains a cycle spanning a large transaction batch; the protocol's fallback then outputs a batch with attacker-influenced internal ordering, enabling sandwiching at scale. The paper also evaluates some mitigations (batch-size caps, randomization of fallback ordering) but finds them heuristic — either they reintroduce unfairness, or they bound damage only empirically, not with worst-case guarantees. (Verify details of the mitigation evaluation against the paper.)
- **Fair ordering lineage:** Kelkar, Zhang, Goldfeder, Juels (Aequitas, 2020) — order-fairness definitions (batch-fairness, *k*-fairness) and the impossibility of total-order fairness via Condorcet cycles. Follow-ups (Cachin et al.'s "swift/thunder" ordering in Themis; Zhnuo/other replication protocols) refine liveness and fairness definitions but inherit cycle-fallback behavior.
- **Related theory:** rank aggregation with cyclic preferences (Kemeny ranking, rank aggregation hardness), Kemeny-style median orders give a natural fallback, but with no adversarial analysis.
- **Gap:** no protocol with a *proven* worst-case bound on reorderable-set size and delay under manufactured cycles.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Lower bound:** a proof that any protocol achieving (some stated fairness notion) admits an adversary with f colluders forcing Ω(h(m,f)) re-orderable transactions / extractable profit — establishing the best possible.
2. **Upper bound / construction:** a fair-ordering protocol plus a theorem: under f < threshold Byzantine clients, any manufactured Condorcet cycle affects at most B(f, m) transactions in order and adds at most D(f, m) delay, with B sublinear in m (ideally O(f) or O(cycle size as identifiable from honest reports).
3. **Matching bound:** 1 + 2 together.
4. **Weaker but accepted:** a formal adversarial model (game between protocol and cycle-manufacturing adversary), an explicit damage metric, and an analysis showing a concrete deployed protocol's fallback has bounded damage — even if the bound is not tight.

A referee would want: a precise threat model (how clients' receive orders are controlled), a formal damage definition, and worst-case (not average-case/simulation) guarantees.

## How to start

1. **Read Vafadar & Khabazzian (2023) first**, especially the attack construction and the mitigation evaluation section — understand exactly why batch caps and randomized fallback fail worst-case.
2. **Read Aequitas (Kelkar et al., FC 2020)** Sections on order-fairness definitions and the Condorcet impossibility; formalize the "damage" metric against their batch semantics (this formalization is a genuine contribution).
3. **Natural approach:** make the fallback Kemeny-like — output a ranking minimizing disagreement with client reports — and try to bound how far the adversary can push the Kemeny-optimal order from honest clients' consensus order with f corrupted preference lists. Known robustness results on rank aggregation under bribed voters may transfer.
4. **Main obstacle:** distinguishing a manufactured cycle from a genuine one using only reported orders; any deterministic rule that treats them identically can be gamed. Randomized or report-consistency-checking fallbacks are the likely lever, but randomization alone was shown insufficient (verify).

## Key references

- Vafadar, S., Khabbazian, R. (2023). "Condorcet Attack Against Fair Transaction Ordering." AFT 2023, LIPIcs. https://doi.org/10.4230/LIPIcs.AFT.2023.15 — Defines and demonstrates the manufacturing of Condorcet cycles against fair-ordering protocols (esp. Aequitas fallback) and empirically evaluates mitigations.
- Kelkar, M., Zhang, F., Goldfeder, S., Juels, A. (2020). "Order-Fairness for Consensus Protocols." Aequitas, FC 2020. https://doi.org/10.1007/978-3-030-51280-4_24 — Introduces receive-order fairness definitions, the Condorcet impossibility, and the Aequitas protocol with its cycle-handling batch fallback.
- Cachin, C., et al. "The Thunder and Thunderella... " — (verify) closest relevant is Cachin's work on swift ordering / Themis (Kelkar et al., CCS 2021), which improves fair-ordering liveness; check its cycle handling.
- Kemeny rank-aggregation literature (e.g., Kenyon-Mathieu & Schudy, 2007, on computing Kemeny rankings) — algorithmic backbone for a principled fallback ordering.

## Warnings

- The original census record is vague; the "bounded damage" objective has no canonical formalization — you must define the metric (reorderable set size, extractable value, latency) or reviewers will reject the framing.
- Average-case / simulation results do not answer this problem; it asks for worst-case guarantees.
- Don't conflate the consensus-level adversary (validators) with client-level adversaries (order reporters) — the attack is at the client-report layer and standard BFT guarantees don't address it.
- Randomizing fallback order was already evaluated in the 2023 paper and found insufficient as a standalone mitigation; check that section before proposing it.
- Entries marked "(verify)" above were reconstructed from field knowledge, not confirmed against the cited papers.

---
*`BCGT-000316` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Lower bound:** a proof that any protocol achieving (some stated fairness notion) admits an adversary with f colluders forcing Ω(h(m,f)) re-orderable transactions / extractable profit — establishing the best possible.
2. **Upper bound / construction:** a fair-ordering protocol plus a theorem: under f < threshold Byzantine clients, any manufactured Condorcet cycle affects at most B(f, m) transactions in order and adds at most D(f, m) delay, with B sublinear in m (ideally O(f) or O(cycle size as identifiable from honest reports).
3. **Matching bound:** 1 + 2 together.
4. **Weaker but accepted:** a formal adversarial model (game between protocol and cycle-manufacturing adversary), an explicit damage metric, and an analysis showing a concrete deployed protocol's fallback has bounded damage — even if the bound is not tight.

A referee would want: a precise threat model (how clients' receive orders are controlled), a formal damage definition, and worst-case (not average-case/simulation) guarantees.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/1757a312-3950-4d7f-b5a1-06d22356ad9a?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000316.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000316)
