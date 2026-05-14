---
name: fraud-investigation
description: Use for any fraud-analyst work — fraud hunting (looking for new patterns), fraud triage (ranking suspicious data into prioritized alerts), or fraud investigation (driving a specific cohort to a conclusion). Triggers include ATO / account takeover, first-party fraud, dispute fraud, mule activity, money laundering, suspicious clusters or cohorts, ranking alerts, "look into these accounts", "check this signal", "find what's worth investigating", or references to seed users, shared device identifiers, or named signals. Also triggers on vague requests like "anything weird?", "what should I look at?", "spike in X", or "these UIDs look related" — the skill routes between phases based on what the user said.
version: 2.0.0
---

# fraud-investigation

A unified skill for fraud-analyst work covering three phases that
usually run in sequence but can be entered independently:

- **Hunting** — no specific seed or cohort. Look for new fraud the
  monitors haven't caught. Output: data (queries + CSVs) surfacing
  candidate patterns worth ranking.
- **Triage** — has data; needs ranking. Turn it into a prioritized
  short-list of alerts with cluster boundaries identified.
- **Investigation** — has a specific cohort. Test hypotheses, gather
  evidence, drive to a written conclusion.

Sequential handoffs are natural (hunting → triage → investigation),
but each phase is a valid entry point.

The single most important principle: investigate **coordinated** fraud
— clusters of users or accounts linked by a strong shared attribute or
several weak attributes with overlapping signals. One-off cases can be
raised if impactful and unmitigated, but do not focus on them.

## Terminology
The below are general terms. If it becomes clear there is another, more
specific term in use at the organization, adopt that one in the future.

**Mitigated** = fraud-ops has acted on this account (restricted,
closed, blocked, frozen, suspended, or whatever the organization's
remediation is); no further action needed. Th
**Unmitigated** = still a live fraud surface where losses or customer
harm can happen. (account not restricted, funds still in transit, etc)

## Step 0: consult the reference library

List the organization's reference docs (commonly
`.claude/skills/<this-skill>/references/` plus any local `docs/`
folder in the repo). They encode tribal knowledge — known red flags, KRIs, prior
typologies, which signals are strong vs. weak. Skim broadly in
hunting; read topic-relevant ones in scoped phases.

This skill ships its own references:

| File | When to read it |
|---|---|
| `references/hunting.md` | Hunting mode. |
| `references/triage.md` | Triage mode. |
| `references/investigation.md` | Investigation mode. |
| `references/linkage-attributes.md` | Deciding whether to expand a cluster. |
| `references/fraud-patterns.md` | Common typologies and their signals — for naming clusters, picking angles, and choosing evidence queries. |
| `references/output-conventions.md` | Workspace folder layout and slug rules. |

Don't try to do hunting / triage / investigation from memory; the
phase references are the heart of the skill.

## Step 1: classify the phase

- **Hunting** — rough direction with no seed cohort, no scope at all,
  or a domain to scan but no specific entities.
- **Triage** — user has data to rank; phrases like "rank these",
  "where are the boundaries", "top candidates", or a file with cohort
  data and a request for prioritization.
- **Investigation** — user has a specific cohort (seed list, signal
  resolved to users, device cluster); phrases like "investigate
  these", "build a case", "what are they doing"; or a follow-up to
  triage opening a case on a top alert.

If genuinely ambiguous between two phases, ask one clarifying
question — usually "do you already have a cohort, or are you looking
for one?" Once classified, read `references/<phase>.md` and follow it.

## Sequential handoffs

When a phase ends, suggest the next one in plain terms (describe what
would happen; don't name the skill or a command) and wait for the
user's go-ahead. Don't auto-execute. If the user defers, recommend the
top 1–2 candidates by priority + exposure.

The user can start at any phase, skip phases, or loop back. If a
finding overlaps a known cluster, surface it and ask whether to extend
the existing case or move on.

## Cohort exposure

Required in triage and investigation. Optional in hunting (useful
context, but not worth interrupting fast iteration for).

The goal is to size the live surface a cohort represents so ranking
isn't driven by raw count alone — a 200-user cluster that's all
mitigated is worth less than a 5-user ATO ring with $500K of live
balance. The right metrics depend on what the organization's accounts
actually do:

- **Common across most organizations:** account balance (static exposure),
  money-in volume over a recent window (flow), money-out volume
  (drain).
- **Organization-specific examples:** loan amounts outstanding, crypto
  trading volume, disbursement volumes, promotional credit amounts,
  card transaction volume, merchant-of-record exposure.

Pick 2–4 metrics that best capture *"if this cohort remained undetected
what would the financial impact be?"*. Always include both a static 
and a flow metric — static-only misses drained-but-flowing patterns (mule, 
ATO with same-day drain). Always split each metric into mitigated vs. 
unmitigated portions, and present them side-by-side in any output table; 
never collapse.

## Save every query, reuse what's already there

Save every query and every result, every time. One `.sql` paired with
one `.csv`, written into the workspace's `data/` folder with a
chronological prefix and descriptive slug:
`data/03_ach-originator-grouping.sql` +
`data/03_ach-originator-grouping.csv`.

Default location: agent workspace under
`{workspace_root}/{date}_{slug}/`. If the user names another
location, treat that as `{workspace_root}` for the session.

Before writing a new query, grep the workspace by attribute, signal,
table, or slug. Adapt close matches; for exact-question matches,
surface the prior result rather than re-running, unless the result
is stale.

## Other shared principles

- **Time-boxing.** Hunting: 15–45 minutes. Triage: ~10–15 minutes.
  Investigation: variable, but >60 minutes without a check-in is a
  sign something's off.
- **Query cost discipline.** Refine before forcing. Cohort-selection
  queries (producing the case CSV in triage or investigation setup)
  must stay under the soft limit (commonly ~100k rows). Per-user
  data-pull pipelines for an existing cohort may legitimately use
  forcing.
- **Cross-reference for novelty.** Every promising finding gets
  cross-checked against prior workspaces (grep `{workspace_root}` for
  user IDs, device identifiers, or attribute values) and against
  recent monitor output. Findings that fully overlap prior work or
  an active monitor aren't novel; usually not worth opening as a new
  workspace. Record the verdict explicitly in `investigation.md`.
- **Cluster-expansion discipline.** Don't silently expand a cluster
  on weak or single-moderate signals. Attributes are **strong** (link
  on their own — device keys, payment instrument tokens, low-occurrence
  counterparties), **moderate** (link only in combination — IP,
  browser fingerprint, uncommon email domains), or **weak** (slicing
  only — user-agent, DOB, common email domains, area code, signup
  time). See `references/linkage-attributes.md` for the
  lift-plus-significance ratio test.
- **Output discipline.** All artifacts for a session live under one
  workspace folder `{workspace_root}/{YYYY-MM-DD}_{slug}/`, with
  `data/` for query artifacts and `investigation.md` as the single
  running document. Slug is
  2–4 kebab-case words describing the hypothesis, angle, or alert.
  See `references/output-conventions.md`.

## What this skill does NOT do

- Run downstream investigation pipelines (per-user data-pull scripts,
  automated analyst loops). Prepare the inputs and hand off.
- Build monitors. If an investigation concludes a pattern should be
  monitored, recommend it in the report — building it is separate
  work.
- Take fraud-ops action (mitigate accounts, file SARs). Produce the
  analysis; fraud-ops acts.
