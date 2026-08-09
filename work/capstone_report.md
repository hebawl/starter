# Capstone Report — Structured Content Archetype Clustering

- **Author:** Heba Walid Awad
- **Lane:** Structured Content Archetype Clustering
- **Repo:** https://github.com/hebawl/starter
- **Date:** 2026-08-08

## 1. Problem framing

The unit of analysis is one content page (pseudonymized). The output is a page-level
archetype (K-Means cluster) plus a ranked action with a reason code. The action a human
takes is opening the top of the queue during weekly editorial triage and deciding whether
to refresh, investigate, or leave a page alone. Getting this wrong costs editor time on the
wrong pages, or — worse — missing a stale, high-traffic page that's quietly losing clicks.
Data/ML helps here because with thousands of pages per client, no editor can eyeball all of
them; a fast, explainable sort is more useful than no sort, as long as it stays honest
about what it can and can't claim.

## 2. Data safety

`content_refresh_anonymized.csv` — 30,000 pages, 32 pseudonymized clients, one 90-day
snapshot. Excluded from every feature/scoring step: `trend_direction` and `trend_pct`
(these define the growth/decline label — using them would be circular), `content_id` and
`client_id` (grouping/splitting only, never features), and any FlyRank product flag
(health_score, priority_score — these encode a decision already made). No client names,
domains, raw queries, or credentials appear anywhere in `work/`.

## 3. Baseline

A transparent tier-cross rule: `freshness_tier × impression_tier`, 14 groups, no fitted
parameters — the same "hand-written if-statement" logic FlyRank's product already runs.
Fair comparison because it uses the same underlying signals (freshness, traffic) as the
model, just without any fitting. On the held-out clients, silhouette = −0.13.

## 4. Model / analysis

K-Means clustering (k=4) on two features: `impressions_90d`, `days_since_last_update`.
Kept small on purpose so every cluster is explainable in one sentence. No target/label —
this is descriptive grouping, not prediction; the "proxy" is simply the shape of the data
along traffic and staleness.

## 5. Evaluation

Grouped-by-client holdout: 20% of clients set aside entirely, model fit only on the
remaining 80%, evaluated only on the held-out clients' pages. This is the same reasoning
as Week 5/6 — pages from one client are too similar for a random row split to be honest.
On this split, K-Means silhouette = 0.84 vs. baseline's −0.13. Caveat: the baseline
produces 14 fine-grained tiers by construction vs. K-Means' 4 broader groups, so part of
the gap reflects granularity, not only fit quality — see the paper's Results section for
the full framing.

## 6. Interpretation

The four archetypes separate mainly along two axes: how much traffic a page pulls, and how
long it's gone unrefreshed. In plain words: some archetypes are high-traffic and stale
(prime refresh candidates), some are low-traffic regardless of freshness (not worth review
time yet), and the rest sit in between. No archetype was a total surprise; the useful
result is that a *simple* two-feature model separates pages about as cleanly as a much more
fragmented rule — a small set of archetypes is easier for a human to reason about than 14
tiers.

## 7. Recommendation

The ranked queue applies a plain rule on top of the archetypes: `refresh` (stale, visible,
underperforming CTR) tops the list, `investigate` (underperforming but not stale — likely
not a staleness problem) is second, `monitor` is everything else. An editor works the
`refresh` tier first, sorted by traffic. Confidence: directional and decision-support only
— every claim here is observed/measured on this snapshot, not causal. Limits: no experiment
was run, so nothing here proves a refresh will help; the model doesn't read page content,
so branded/navigational-query false positives are expected and need a human check.

## 8. Reproducibility

Clone the repo, then from its root:

```
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute work/notebooks/capstone.ipynb
```

Random seed: 42, fixed throughout. `work/outputs/capstone_summary.json` holds the exact
numbers reported here and on the deployed paper; re-running the notebook regenerates it and
the two figures in `docs/assets/`.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
