# Phase: Hunting

Hunting is the phase before any cohort exists. The user wants to look for
fraud the monitors haven't caught yet, with a rough direction at most. The
output is *data* — queries run and CSVs produced — that surfaces candidate
patterns worth ranking in triage.

## What success looks like

A hunt produces one of three outcomes, in rough order of preference:

1. **A promising pattern.** One or more CSVs of candidate users /
   attributes / signals that look novel and actionable. Hand off to
   triage to rank them.
2. **A clean cold result.** Several pivots returned nothing
   interesting. Say so plainly; don't pretend the data was richer than
   it was.
3. **An overlap with existing work.** The pattern is already covered
   by a prior workspace or active monitor. Note it, close out, don't
   escalate.

A "no leads" hunt is still a successful hunt. The bad outcome is a
hunt the user didn't get to steer when steering would have changed it.

## How to hunt

Pick angles by judgement. Adapt prior queries from `{workspace_root}`
before writing new ones — see SKILL.md "Save every query, reuse what's
already there". When a query surfaces users, joining to live account
status to surface unmitigated exposure is useful context (see SKILL.md
"Cohort exposure" for which metrics fit) but optional in hunting —
don't let it slow the iteration.

Decide for yourself when to loop the user in. Their context is
valuable when their input would change your next cut, when an angle
looks cold and the pivot is non-obvious, or when a finding is
significant enough that they'd want to know now. Don't talk to the
user just to narrate — the chat is the log of the hunt; padding it
with running commentary makes the actual signal harder to find.

Stay narrow. A grouped query with a count threshold
(`count(*) > 20` on an email-domain pivot) is more useful than a
broad scan of every domain. Start tight; broaden only if nothing
surfaces.

## Cross-reference before recommending a lead

Before handing a lead to triage, check that it's actually novel:

- Grep `{workspace_root}` for every user ID and strong attribute the
  hunt surfaced. Hits mean prior workspaces have already worked on
  this — read them and decide whether the lead is still novel.
- Check recent monitor output (last few days) for the leading
  attribute. If the fleet already flags it, novelty is low.

Record the verdict in `investigation.md` explicitly.

## Output layout

```
{workspace_root}/{YYYY-MM-DD}_{slug}/
├── data/
│   ├── 01_*.sql        — every query that ran, chronologically numbered
│   ├── 01_*.csv        — its result, paired with the query
│   └── ...
└── investigation.md    — running document for this workspace
```

In hunting, `investigation.md` is the hunt's running brief:

- Direction and final angle.
- What was found — user counts, specific attribute values, pointers
  to the cohort CSVs in `data/` that the triage phase would consume.
- Cross-ref verdict (novelty check).
- Recommended next step. Usually *"triage the candidates in
  `data/NN_*.csv` into ranked alerts"*, sometimes *"add a new
  monitor — sketch below"*, sometimes *"dead end; already covered by
  monitor X"*.

If triage runs in the same workspace later, the same
`investigation.md` gets a triage section appended (and so on for
investigation).

## Handoff

When a lead is solid enough to triage (or the user decides to stop),
end with one message: workspace path, one-sentence headline, the
handoff question — *"want me to triage this into ranked alerts?"*
Don't auto-run triage; the user decides.
