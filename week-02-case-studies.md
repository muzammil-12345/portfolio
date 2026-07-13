# Portfolio Case Studies

## Voice Card
**Direct, plain, proof-first, no hype.**

---

## Case 1: FlyRank Content Refresh Scoring

**The problem**
FlyRank had ~30,000 pages and no clear way to decide which ones needed a content refresh first. "Refresh everything" isn't a plan — it's a guess wearing a to-do list. Someone had to turn "which pages matter" into something a model could actually rank.

**What I did**
I framed it as an ML ranking/scoring task instead of a vague content audit. Picked Precision@50 as the success metric — because in a refresh queue, what matters is whether the top of the list is actually right, not overall accuracy across 30k pages nobody's going to touch this quarter. Built a baseline, then iterated the model until it beat the baseline by a real margin. Documented every decision — metric choice, feature framing, why baseline-vs-model comparison mattered — instead of just shipping a number.

**What came of it**
Precision@50 lift from 0.24 to 0.74. That's the difference between a refresh list that's mostly noise and one you can actually trust and act on.

---

## Case 2: #100DaysOfCode — Weather App

**The problem**
Needed to prove I can work with real-world, unreliable inputs — external APIs, live data, things that fail in ways a clean dataset never does.

**What I did**
Built a weather app using the OpenWeatherMap API from scratch: handled HTTP requests, JSON parsing, and — the actual hard part — error handling for bad inputs, failed requests, and missing data.

**What came of it**
A working app that doesn't break the moment the API misbehaves, and a documented Day 19+ milestone in a public 100-day build streak.

---

## Bio

I'm a 4th-semester AI student who ships. Currently an ML Engineer Intern at FlyRank, building a content-scoring pipeline from a 30k-page dataset. I turn messy, undefined problems into measurable ML tasks — and I document the decisions, not just the results.

## Contact / CTA

If you're a founder or hiring manager looking for someone who can go from ambiguous problem to working pipeline without hand-holding — email me for a 15-minute call.

---

## Before / After: Generic AI Line vs. Edited Version

**Generic AI line:**
"I am a results-driven AI engineering student passionate about leveraging cutting-edge machine learning techniques to deliver impactful solutions."

**Edited version (mine):**
"I turn messy, undefined problems into measurable ML tasks — and I document the decisions, not just the results."
