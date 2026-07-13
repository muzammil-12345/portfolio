# Prompt Iteration Log — FL-01 Task: Content Refresh Priority Scoring Function

**Task:** Write a Python function that computes a content refresh priority score for each page in my dataset, using traffic decline and staleness signals.

---

## V0 — Naive Baseline

**Prompt:** "Write a Python function to score pages for content refresh priority."

**Output (excerpt):**
```python
def score_page(page):
    score = 0
    if page['traffic'] < 100:
        score += 1
    if page['days_since_update'] > 180:
        score += 1
    return score
```

**Verdict:** Generic, toy-level. Made up field names, hardcoded arbitrary thresholds, no explanation, no connection to my actual project or metric (Precision@50). This is exactly the kind of thing I'd have accepted a month ago without questioning it.

---

## V1 — Role Assignment

**Prompt:** "You are a senior ML engineer who specializes in ranking systems. Write a Python function to score pages for content refresh priority."

**Output (excerpt):**
```python
def compute_refresh_score(page: dict) -> float:
    """
    Computes a weighted refresh-priority score.
    Combines normalized traffic decline and staleness signals.
    """
    traffic_decline = (page['sessions_90d'] - page['sessions_30d']) / max(page['sessions_90d'], 1)
    staleness = min(page['days_since_update'] / 365, 1.0)
    return 0.7 * traffic_decline + 0.3 * staleness
```

**Notes:**
- *Prompt change:* Added a role ("senior ML engineer, ranking systems specialist").
- *Output improvement:* Immediately more professional — normalized signals instead of arbitrary thresholds, a docstring, type hints, and an actual weighted formula instead of a naive counter. It "acted like" someone who's built ranking systems before.
- *Still failed:* Still invented field names (sessions_90d, sessions_30d) without knowing my real schema, and the 0.7/0.3 weights are asserted with zero justification.
- *Try next:* Give it real context about my project so the weights and fields aren't guesses.

---

## V2 — Context and Motivation

**Prompt:** *(V1)* + "Context: this is for FlyRank's content refresh project. I have a 30k-page dataset with columns page_url, sessions_30d, sessions_90d, last_modified_date, avg_position. This score feeds into a ranking model evaluated on Precision@50 — so the top 50 scored pages need to be genuinely the highest-priority ones, not just plausible."

**Output (excerpt):**
```python
def compute_refresh_score(page: dict) -> float:
    """
    Refresh-priority score for FlyRank content scoring.
    Optimized for ranking quality (Precision@50), not raw accuracy —
    prioritizes correctly ordering the top candidates over the full set.
    """
    traffic_decline = (page['sessions_90d'] - page['sessions_30d']) / max(page['sessions_90d'], 1)
    staleness_days = (pd.Timestamp.now() - pd.to_datetime(page['last_modified_date'])).days
    staleness = min(staleness_days / 365, 1.0)
    return 0.7 * traffic_decline + 0.3 * staleness
```

**Notes:**
- *Prompt change:* Added real project context — actual column names and the Precision@50 evaluation goal.
- *Output improvement:* Now uses my real columns (last_modified_date, not a fake days_since_update field) and the docstring explicitly reasons about why Precision@50 changes the design ("prioritizes correctly ordering top candidates"). That's a real conceptual improvement, not just cosmetic.
- *Still failed:* The 0.7/0.3 weighting is still asserted with no justification — it "sounds right" but nothing backs it.
- *Try next:* Give a few-shot example of what a good vs bad scored output looks like, so it calibrates against my judgment instead of its own assumption.

---

## V3 — Few-Shot Examples

**Prompt:** *(V2)* + "Here are two example pages and how I'd rank them, so you calibrate correctly:
Page A: sessions_30d=50, sessions_90d=500, last_modified 400 days ago → HIGH priority (steep decline + very stale)
Page B: sessions_30d=480, sessions_90d=500, last_modified 40 days ago → LOW priority (barely declined, recently updated)
Match this reasoning in your scoring logic."

**Output (excerpt):**
```python
def compute_refresh_score(page: dict) -> float:
    """
    Refresh-priority score calibrated against reference examples:
    steep decline + high staleness = high priority;
    stable traffic + recent update = low priority.
    """
    traffic_decline = (page['sessions_90d'] - page['sessions_30d']) / max(page['sessions_90d'], 1)
    staleness_days = (pd.Timestamp.now() - pd.to_datetime(page['last_modified_date'])).days
    staleness = min(staleness_days / 365, 1.0)
    # weighted higher toward decline since Page A example shows decline 
    # alone (90%) drove HIGH priority despite staleness only being ~1.0 too
    score = 0.75 * traffic_decline + 0.25 * staleness
    return round(score, 4)
```

**Notes:**
- *Prompt change:* Added two concrete worked examples with my own priority judgment attached.
- *Output improvement:* This is the first version where the weighting decision is actually justified against something real — it reasoned from my examples ("Page A example shows decline alone drove HIGH priority") rather than asserting a number. The comment now explains *why* 0.75/0.25, not just that it is.
- *This is the "didn't help much" moment, honestly:* the actual score values barely moved (0.7/0.3 → 0.75/0.25) — two examples weren't enough data to meaningfully recalibrate weights, just enough to make the model *narrate* more confidently. I'd need many more labeled examples for this layer to actually change behavior, not just its explanation.
- *Try next:* Force output structure so the function comes with usage instructions, not just code.

---

## V4 — Output Structure

**Prompt:** *(V3)* + "Structure your response as: (1) the function code, (2) a one-line explanation of the formula, (3) a usage example with sample input/output, (4) one limitation of this approach."

**Output (excerpt):**
```python
def compute_refresh_score(page: dict) -> float:
    traffic_decline = (page['sessions_90d'] - page['sessions_30d']) / max(page['sessions_90d'], 1)
    staleness_days = (pd.Timestamp.now() - pd.to_datetime(page['last_modified_date'])).days
    staleness = min(staleness_days / 365, 1.0)
    return round(0.75 * traffic_decline + 0.25 * staleness, 4)
```
**Explanation:** Weighted sum favoring traffic decline (75%) over staleness (25%), based on calibration examples.

**Usage:**
```python
page = {'sessions_30d': 50, 'sessions_90d': 500, 'last_modified_date': '2024-06-01'}
compute_refresh_score(page)  # → 0.9375 (high priority)
```

**Limitation:** This formula can't distinguish "low traffic because irrelevant" from "low traffic because undiscoverable" — a great page with poor internal linking would score the same as genuinely dead content.

**Notes:**
- *Prompt change:* Forced a 4-part output structure.
- *Output improvement:* Went from "just code" to something a teammate could actually pick up and use in 30 seconds — runnable example plus a stated limitation I hadn't been given before (the "irrelevant vs undiscoverable" distinction is a genuinely new, useful caveat).
- *Still failed:* Still a single flat function — no error handling for missing/malformed fields, which real 30k-row data will absolutely have.
- *Try next:* Step decomposition to force it to build this more robustly, piece by piece.

---

## V5 — Step Decomposition

**Prompt:** *(V4)* + "Before writing the final function, break this into steps: (1) validate required fields exist, (2) compute each signal separately with edge-case handling, (3) combine into final score, (4) then write the complete function following those steps."

**Output (excerpt):**
```python
def compute_refresh_score(page: dict) -> float:
    # Step 1: validate
    required = ['sessions_30d', 'sessions_90d', 'last_modified_date']
    missing = [f for f in required if f not in page or page[f] is None]
    if missing:
        raise ValueError(f"Missing required fields: {missing}")

    # Step 2: compute signals with edge-case handling
    sessions_90d = max(page['sessions_90d'], 1)  # avoid div-by-zero
    traffic_decline = max((sessions_90d - page['sessions_30d']) / sessions_90d, 0)
    staleness_days = (pd.Timestamp.now() - pd.to_datetime(page['last_modified_date'])).days
    staleness = min(max(staleness_days, 0) / 365, 1.0)

    # Step 3: combine
    score = 0.75 * traffic_decline + 0.25 * staleness
    return round(score, 4)
```

**Notes:**
- *Prompt change:* Forced explicit step decomposition before final code generation.
- *Output improvement:* This is the first version that's actually production-safe — it validates missing fields, clamps negative values, and guards div-by-zero. None of the earlier versions would have survived real messy data without crashing.
- *Still failed:* Still doesn't handle the case where last_modified_date is malformed/unparseable — the decomposition covered missing fields but not invalid ones.
- *Try next:* At this point, further prompt layers hit diminishing returns — the remaining gap needs actual testing against real rows, not more prompting.

---

## Cross-Model Comparison (Final Prompt — V5, run on both)

**Claude (Sonnet):** Produced the version above — cautious, explicit validation, comments explaining *why* each guard exists (e.g., "avoid div-by-zero"), conservative about clamping values to safe ranges. Slightly verbose in its step-by-step reasoning before the code block.

**ChatGPT:** Produced functionally similar logic but skipped the explicit missing-field validation step unless re-prompted to confirm it followed all four steps — it jumped straight to a working function faster, with less narration, but the first pass silently assumed all fields were present. When asked to double check against the four steps, it added the validation and matched Claude's version closely.

**Specific differences observed:**
- **Accuracy:** Both landed on the same formula and weights — the reasoning quality was equivalent once both were pushed to follow all steps.
- **Structure adherence:** Claude followed the 4-step decomposition unprompted on the first try; ChatGPT's first pass compressed steps 1 and 2 together and needed a follow-up nudge to fully separate them.
- **Tone:** Claude's comments explained *why* (defensive reasoning); ChatGPT's comments were more terse, stating *what* the line does rather than why.
- **Failure point:** ChatGPT's first-pass output would have crashed on missing fields in production; Claude's didn't. This is the one concrete, real difference — not "both were fine."

---

## Final Reusable Template

```
You are a [relevant expert role] working on [task domain].

Context: [real project details — data schema, what this feeds into, 
the metric or outcome that defines success].

Here are calibration examples of correct vs incorrect output:
[2+ concrete input/output pairs with your own judgment attached]

Structure your response as:
1. The solution
2. A one-line explanation of the core logic
3. A usage example with real sample input/output
4. One explicit limitation of this approach

Before producing the final solution, decompose the problem into steps 
(validation → core logic → edge cases → combination), and build the 
solution following those steps explicitly — don't skip straight to 
the final version.
```

**Why this works for a stranger:** every bracketed slot is generic (role, domain, context, examples, structure) — nothing requires knowing my dataset or FlyRank specifically. It works for a scoring function, a data-cleaning script, or a totally different domain, because the technique layers (role, context, few-shot, structure, decomposition) are domain-agnostic.
