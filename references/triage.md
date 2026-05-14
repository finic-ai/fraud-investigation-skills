# Phase: Triage

Triage starts from data — a CSV of suspicious users, a monitor dump,
the output of a hunt — and produces a ranked short-list of alerts
worth investigating. The hard part isn't ranking; it's **defining
alert boundaries**: deciding which subset of the input belongs to
which alert.

## What success looks like

1. **Cluster boundaries.** A scatter of suspicious users typically
   breaks into 2–6 coherent sub-clusters. Triage names them.
2. **A ranked short-list.** Each alert has a size, an exposure read,
   the linkage attribute(s) that define it, a novelty check, and a
   one-line pitch.
3. **A user-confirmed pick of which 1–2 alerts to investigate first.**

A triage that returns "all 200 belong to one giant alert" usually
means the boundary work wasn't done — most large inputs have
sub-structure worth surfacing. A triage that legitimately finds no
coherent alerts ("uncorrelated one-offs") is also valid; say so
plainly.

## How to triage

Two kinds of work, alternating as needed:

- **Sift the input.** Filter, sort, slice. Group by candidate strong
  attributes (device key, payment instrument token, low-occurrence
  counterparty), look at histograms, spot the dense pockets. See
  `linkage-attributes.md` for strong vs. moderate vs. weak.
- **Run follow-up queries** when more context is needed — verifying a
  candidate cluster, pulling exposure, checking whether a boundary
  attribute really concentrates in the proposed cohort. Save into
  `data/` per SKILL.md "Save every query".

Surface proposed boundaries to the user early; their context usually
saves wasted exposure queries ("looks like these 200 break into 3
clusters: A is N=80 sharing device key X; B is N=50 sharing ACH
originator Y; the rest are isolated — does that match your
intuition?").

For each candidate cluster, run cohort exposure (see SKILL.md
"Cohort exposure"). Batch candidates into a single UNION ALL query,
not one per cluster.

## Cross-reference each cluster

Before recommending a cluster, grep `{workspace_root}` for its user
IDs and strong attributes, and check recent monitor output for the
lead attribute. A cluster that fully overlaps a prior workspace or
active monitor is usually not worth opening — note as
`OVERLAPS <slug>` and move on.

## Present the short-list

Update `investigation.md` with a "Triage" section, and surface the
same content in chat as a numbered table:

| # | alert | size | unmitigated | unmitigated_balance_$ | money_in_30d_$ | linkage | novelty | reason |

- `alert` — short cluster name (`device-key-X-onboarding-ring`,
  `ach-originator-Y-reversal`).
- `unmitigated` — `N / total (pct%)`.
- Both dollar columns required; never collapse them.
- `linkage` — the attribute or combination defining the cluster.
- `novelty` — `NEW`, `OVERLAPS <slug>`, or `IN monitor Y`.
- `reason` — one-line pitch.

If the team uses an organization-specific priority taxonomy (e.g.
ATO / first-party / mule, fraud-type categories, risk tiers), add it
as an extra column — defer to whatever the team's reference docs
encode. The skill itself doesn't impose a taxonomy.

Ask which alert(s) to investigate. If the user defers, recommend the
top 1–2 by combined live exposure (`unmitigated` weighted by
`max(unmitigated_balance_$, money_in_30d_$)`), respecting any
org-specific priorities.

## Handoff

Mark the user's selection in `investigation.md`. Ask: "open the
investigation in this workspace, or start a new one?" Wait. Don't
auto-open.

## Judgement rules

- **Surface boundaries early**, before computing exposure on all
  candidates.
- **Don't force a cluster.** If users don't share strong attributes
  and don't co-occur on moderates, "uncorrelated one-offs" is the
  right answer.
- **Don't over-segment.** 6 clusters of N=3 are usually less useful
  than 2 clusters of N=10. Reconsider whether tiny groupings belong
  together as one heterogeneous alert.
- **Triage is not investigation.** Rank, don't conclude. If you're
  digging into one cluster's behavior for 5+ minutes, stop — that's
  the next phase.

## Output layout

```
{workspace_root}/{YYYY-MM-DD}_{slug}/
├── data/
│   ├── 01_*.sql        — boundary, exposure, follow-up queries
│   ├── 01_*.csv
│   └── ...
└── investigation.md    — triage adds a "Triage" section with the
                         ranked short-list, exposure, linkage notes,
                         and the user's selection.
```

Reuse the hunt's workspace if continuing from a hunt; otherwise
create a new workspace named for the input or theme.
