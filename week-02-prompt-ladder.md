# Prompt Ladder: Explaining My Content Scoring Data

---

## Baseline (Weak Prompt)

**Prompt:** "Explain this data."
*(pasted the content_refresh_anonymized.csv columns + a few sample rows, no other instruction)*

**Output (excerpt):**
"This dataset appears to contain information about web pages, likely for content analysis purposes. It includes columns such as page URL, traffic metrics, and possibly engagement data. The data seems to be structured in a tabular format suitable for analysis. There may be patterns related to page performance that could be explored further. Let me know if you'd like me to look at specific columns in more detail."

**Verdict:** Embarrassing. Generic, hedgy ("appears," "likely," "possibly," "may be," "could be"), doesn't commit to a single real observation, doesn't touch precision/scoring, useless as a starting point.

---

## Version 1 — Add a clear goal

**Layer:** Clearer goal
**Prompt:** "Explain this data. My goal is to figure out which pages need a content refresh first, ranked by priority."

**Output (excerpt):**
"Given your goal of prioritizing pages for refresh, you'll want to look at signals like declining traffic, page age, and search ranking drops. This dataset likely has columns relevant to that. I'd suggest sorting by a combination of these factors to identify the pages most urgently needing attention."

**Notes:**
- *Prompt change:* Added the actual objective (prioritize refresh).
- *Output improvement:* Stopped being purely descriptive — it now points toward specific signals (traffic decline, age, ranking) instead of just describing columns.
- *Still failed:* Still hedging ("likely," "I'd suggest") and hasn't actually looked at my real columns — it's guessing at what's in the dataset instead of using it.
- *Try next:* Force it to reference the actual columns I gave it, not generic guesses.

---

## Version 2 — Add real context

**Layer:** Real context
**Prompt:** "Explain this data. My goal is to figure out which pages need a content refresh first, ranked by priority. Context: this is a 30k-page dataset for FlyRank, and I'm framing this as an ML ranking/scoring task using Precision@50 as the success metric, comparing against a baseline model."

**Output (excerpt):**
"With Precision@50 as your metric, the key question is whether your model's top 50 predicted pages are actually the ones that most need refreshing — not overall accuracy across all 30k. Your baseline should be a simple heuristic (e.g., sort by traffic decline) so you have something to beat. Given the columns you shared, you'd want to engineer features around traffic trend, staleness, and ranking volatility, then evaluate whether the model's top-50 list beats the baseline's top-50 list on actual refresh-worthiness."

**Notes:**
- *Prompt change:* Added the real project context — dataset size, metric (Precision@50), baseline comparison.
- *Output improvement:* This is the first version that's actually *right* — it correctly explains what Precision@50 means for a ranking task (top-50 correctness, not overall accuracy) and ties the baseline concept in correctly. No more generic guessing.
- *Still failed:* Still doesn't reference my actual column names — it's reasoning about the task correctly but not the specific data.
- *Try next:* Give explicit output format so I get something structured, not paragraph explanation.

---

## Version 3 — Add output format

**Layer:** Specified output format
**Prompt:** *(same as V2)* + "Format your answer as: (1) one-sentence summary of what the data supports, (2) a bullet list of the 3 most relevant columns and why, (3) one risk to watch for."

**Output (excerpt):**
"**Summary:** This dataset supports building a Precision@50 refresh-priority ranker if traffic-trend and staleness signals are present in the columns.
**Relevant columns:**
- Traffic trend column — most direct signal of declining relevance
- Last-updated/staleness column — pages untouched longest are refresh candidates
- Ranking/position column — volatility here signals SEO risk worth fixing
**Risk:** If the top-50 predictions cluster on high-traffic pages only, you'll miss low-traffic-but-fixable pages — worth checking precision isn't just chasing volume."

**Notes:**
- *Prompt change:* Added a strict 3-part output format.
- *Output improvement:* Massive jump in usability — I can scan this in 10 seconds instead of reading paragraphs. The "risk" section surfaced something genuinely useful I hadn't been told before (precision chasing volume, not fixability).
- *Still failed:* Still using generic column names ("traffic trend column") instead of my actual CSV headers, because I never gave it the real column names, only sample rows.
- *Try next:* Add constraints forcing it to only use the exact columns I provide, nothing invented.

---

## Version 4 — Add constraints

**Layer:** Constraints
**Prompt:** *(same as V3)* + "Constraint: only reference column names that literally appear in the data I pasted. Do not invent or generalize column names. If a needed signal isn't present in the columns, say so explicitly instead of assuming it exists."

**Output (excerpt):**
"**Summary:** Based on the exact columns provided (page_url, sessions_30d, sessions_90d, last_modified_date, avg_position), this data can support a staleness + traffic-decline ranker, but does NOT contain a direct SEO-volatility signal.
**Relevant columns:**
- sessions_30d vs sessions_90d — direct decline signal
- last_modified_date — staleness proxy
- avg_position — proxy for ranking health, but no volatility/trend version exists in this data
**Risk:** avg_position is a snapshot, not a trend — you can't tell if it's getting worse without a second time point. This is a real gap, not something to paper over."

**Notes:**
- *Prompt change:* Added a hard constraint against invented column names, and a demand to flag missing signals explicitly.
- *Output improvement:* This is the first version that actually caught a real flaw in my dataset — that avg_position is a snapshot, not a trend, so "ranking volatility" (which I casually assumed I could measure) isn't actually available. That's a genuinely new, non-obvious insight, not just cleaner phrasing.
- *This is the "didn't help smoothly" moment:* honestly, V3 → V4 made the output *harder to read* at first glance (it got more clinical, less narrative) — I initially preferred V3's flow. But V4 is objectively more correct and useful, which matters more than readability here. Worth noting constraints can trade polish for accuracy.
- *Try next:* Add quality criteria so I can judge if the final ranking approach itself is sound, not just the column read.

---

## Version 5 — Add quality criteria / review instructions

**Layer:** Quality criteria
**Prompt:** *(same as V4)* + "Before finalizing your answer, check it against these criteria: (1) every claim traceable to an actual column, (2) at least one explicit gap or risk stated, (3) no filler phrases like 'could be useful' or 'may indicate.' Revise your answer if it fails any of these."

**Output (excerpt):**
"**Summary:** Using page_url, sessions_30d, sessions_90d, last_modified_date, and avg_position, this dataset supports a two-signal refresh ranker: traffic decline (sessions_30d vs sessions_90d) and staleness (last_modified_date).
**Relevant columns:**
- sessions_30d / sessions_90d — decline ratio is your strongest available signal
- last_modified_date — staleness proxy, weight this lower than traffic decline since old-but-still-ranking pages shouldn't be penalized
**Gap:** avg_position has no trend version in this data — you cannot measure ranking volatility, only current position. If volatility matters to your model, you need a second time-stamped snapshot, which doesn't currently exist.
**Recommendation:** Weight traffic decline higher than staleness in your initial baseline, since staleness alone produces false positives (old pages that still perform fine)."

**Notes:**
- *Prompt change:* Added explicit self-check criteria (traceability, gap-flagging, no filler) that the model has to pass before finalizing.
- *Output improvement:* The filler is completely gone ("could be useful" type phrases: zero). It also produced an actual recommendation with reasoning (weight traffic decline over staleness, and why) — something none of the earlier versions did. This version gives me something I could act on directly.
- *Still failed:* It still can't verify its own reasoning against real numbers — it's giving sound logic, not tested logic. That's a limit of prompting alone, not something a sixth layer fixes.
- *Try next:* At this point, further layers would have diminishing returns — the next real step is testing this reasoning against actual model output, not more prompt engineering.

---

## Final Reusable Prompt

```
I'm working with [dataset description — size, domain, what it represents].
Goal: [specific decision this data needs to support].
Context: [the actual project framing — task type, metric being optimized, 
baseline being compared against].

Here are the exact columns/fields in my data:
[paste real column names / sample rows]

Format your answer as:
1. One-sentence summary of what this data supports for my goal
2. Bullet list of the most relevant columns and why (only reference 
   columns that literally exist in what I gave you — do not invent 
   or generalize column names)
3. At least one explicit gap or risk — if a signal I'd need isn't 
   present in the data, say so directly instead of assuming it exists
4. One concrete recommendation with reasoning

Constraint: no filler phrases like "could be useful," "may indicate," 
or "appears to." Every claim must be traceable to something I actually 
gave you. Before finalizing, check your answer against these four 
requirements and revise if it fails any of them.
```

**Why this works for a stranger:** it doesn't assume they know my project — it has explicit slots for goal, context, and real data. It forces traceability and gap-flagging regardless of domain, so it works whether someone pastes sales data, survey data, or log files. No inside knowledge of FlyRank or my dataset required to reuse it.
