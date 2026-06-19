# Reflection — Day 17 (≤ 200 words)

Answer briefly, in your own words. This is graded on reasoning, not length.

1. **The flywheel.** Day 13 emitted agent traces; today you turned them into an
   eval set and DPO pairs that Day 22 will train on. Which step in
   `traces → Bronze → datasets` would break most silently in production if you
   got it wrong — and how would you detect it?

   **Answer:** The most silent break is the **flatten step (traces→Bronze)**: Bronze is the immutable source of truth, so a recursion bug (dropped children, lost `trace_id` propagation, mis-hoisted `gen_ai.*` attributes) corrupts every downstream dataset with no exception. `verify.py`'s `n_spans >= len(traces)` check is too weak — flattening only roots still passes. Detect it by asserting per-trace span count equals the raw tree's node count, a non-trivial depth distribution, and non-null `gen_ai.*` columns on `chat` spans.

2. **Decontamination.** Your run dropped 2 of 3 preference pairs because their
   prompts were in the eval set. What concretely goes wrong if you *skip* this
   step and train on those pairs? How would the lie show up in your metrics?

   **Answer:** Skipping decontamination lets the model memorize eval answers: eval accuracy inflates while production degrades. The lie shows up as a large gap between eval and a true holdout, with ~100% accuracy on overlapping prompts.

3. **Point-in-time.** The naive join leaked a future `lifetime_spend` into the
   training row. Describe one feature in a system you know that would be
   dangerous to join without an `ASOF`/point-in-time guard.

   **Answer:** `lifetime_spend` for a fraud model — joining the future value means the model "knows ahead" how much a customer will spend, information unavailable at inference. Fraud scoring needs ASOF so only spend at-or-before the transaction is visible.

4. **Graph vs vector.** From `kg_demo.py`, name one question the knowledge graph
   answers well that flat chunk retrieval (`embed.py`) would struggle with, and
   one where the graph is overkill.

   **Answer:** KG wins "where does a widget ship from?" (2-hop: widget→accessory→Hanoi, fact split across chunks). KG is overkill for "what is the widget return policy?" (single chunk, vector suffices).
