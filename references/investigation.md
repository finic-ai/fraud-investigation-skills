# Phase: Investigation

Investigation is the deepest phase. The user has a specific cohort of users
or accounts (and maybe a hypothesis about what links them) and wants to
drive to a conclusion: are these actors coordinated? What are they doing?
What's the loss exposure? What should fraud-ops do?

The output is a written conclusion in `investigation.md` grounded in
evidence saved under `data/`.

## What success looks like

An investigation session produces:

1. **A workspace** that conforms to the standard layout (see
   `output-conventions.md`) — `data/` for queries and results,
   `investigation.md` for the running document.
2. **A written conclusion** at the top of `investigation.md` stating
   the hypothesis, the evidence, and the verdict.
3. **An updated investigation.md** that future grep can find via user
   IDs, devices, or attributes.

An investigation that ends "the hypothesis didn't hold up — these
users look unrelated" is a valid outcome. Record it the same way; the
value is in the negative answer and in keeping the workspace
discoverable.

## Procedure

### Step 1 — Frame the case

Confirm with the user, in one message:

- The cohort (where it came from, size).
- The hypothesis, if any. If none, propose 1–2 based on what's
  visible in the input (shared device? shared funding pattern?
  co-signal hit?).
- The investigation depth — quick triage-confirmation, or full
  drive-to-conclusion?

If the cohort is too large for any downstream pipeline (commonly
~thousands of users max), agree on a narrowing rule with the user
before proceeding.

### Step 2 — Set up the workspace

If continuing from a triage in the same workspace, just start adding
queries to `data/`. Otherwise create
`{workspace_root}/{YYYY-MM-DD}_{slug}/` with an empty `data/` and
a stub `investigation.md`.

Confirm the user-id column name once with the user — different
downstream pipelines expect different aliases (`customer_id`,
`user_id`, `account_id`). Get it right; it's the contract.

### Step 3 — Pull the cohort

Write the cohort-producing query (e.g. `data/01_cohort.sql`). Before
writing it, grep `{workspace_root}` for prior cohort queries that
might already define this cohort or something close — same seed
attribute, overlapping signals, recent triage for this alert. Adapt
rather than reinvent. Run the final query through the organization's
query runner with output to `data/01_cohort.csv`. Verify the row
count is non-zero and within the downstream pipeline's limit.

### Step 4 — Review what's already known

Before pulling more data, look at what already exists for this
cohort:

- **Prior workspaces.** Grep `{workspace_root}` for every user ID and
  every strong attribute (device key, payment instrument token,
  low-occurrence counterparty) the cohort shares. If users overlap
  with a prior workspace, read its `investigation.md` and decide:
  extend the existing workspace, or open a new one with the prior
  workspace as a referenced predecessor?
- **Monitor output.** Check recent monitor output for any of the
  cohort's leading attributes. If they're already being monitored,
  note it — the investigation might be confirming a known pattern
  rather than discovering a new one.
- **The cohort's own history.** What signals have these users hit?
  When did they sign up? What's the activity pattern? A 5-minute
  scan of obvious dimensions before forming hypotheses prevents
  wasted evidence-gathering.

Record what you find in `investigation.md` under a `## Prior context`
section.

### Step 5 — Form hypotheses

Based on the cohort and what's known, list 1–3 specific hypotheses in
`investigation.md`. Examples:

- *"These users are an onboarding ring funded through a single ACH
  originator, intending dispute-fraud or ACH-reversal abuse."*
- *"These users are ATO victims linked by a credential-stuffing
  campaign that targeted a specific signup cohort."*
- *"These users are a money-laundering pass-through using P2P chains
  through a small ring of accounts."*

Each hypothesis should be falsifiable: what evidence would confirm
it, and what evidence would rule it out?

Surface the hypotheses to the user in chat and ask which to
prioritize. If they defer, work the most likely one first.

### Step 6 — Gather evidence

For each hypothesis, plan 2–4 queries or data pulls that would
discriminate it. The bias is toward narrow, sharp queries — one good
cut beats five broad scans.

For each query:

1. Write SQL to `data/NN_{short-name}.sql`.
2. Run it; write result to `data/NN_{short-name}.csv`.
3. Read the result; update `investigation.md` with what you saw.

When the organization has a downstream investigation pipeline (a
per-user data-pull script, an automated analyst loop), this
evidence-gathering step may be replaced or augmented by handing the
workspace off to that pipeline. In that case, your job is to produce
a clean cohort and hand off, then pick up the threads when the
pipeline returns results. The pipeline's outputs become evidence
in `investigation.md` and additional files under `data/`.

### Step 7 — Drive to a verdict

After enough evidence accumulates (usually after 4–8 evidence queries
or one pipeline run), step back and write the verdict.

The verdict has three possible shapes:

1. **Hypothesis confirmed.** State the typology, the linkage
   attributes, the exposure (unmitigated users × balance × 30d
   money-in), and the recommended fraud-ops action (e.g. mitigate
   the remaining unmitigated accounts, add a monitor, file
   regulatory paperwork).
2. **Hypothesis refuted, alternative found.** The original
   hypothesis didn't hold but the investigation surfaced a different
   pattern. Write the alternative as the new verdict and recommend
   next steps.
3. **No coherent pattern.** The cohort doesn't appear coordinated, or
   the data is too sparse to conclude. State so plainly. Recommend
   either closing the workspace or pulling more data if there's a
   clear next step.

The verdict goes at the top of `investigation.md` — it should be the
first thing a reader sees, followed by the evidence narrative
underneath.

### Step 8 — Handoff

End with a single message:

- Workspace path.
- One-sentence verdict.
- Priority tag.
- Recommended actions (if any).
- The exact command for the downstream pipeline, if one applies and
  hasn't already been run.

Don't auto-execute the downstream pipeline. Don't take fraud-ops
action (mitigating accounts, filing SARs). Those are the user's
calls.

## Judgement rules specific to investigation

- **Hypotheses before queries.** Every evidence query should be tied
  to a specific hypothesis it's testing. Running queries without a
  hypothesis is fishing, and it produces evidence that doesn't help
  the reader of the final writeup.
- **Negative results are evidence.** A query that returns "the
  hypothesis isn't supported" is just as valuable as one that
  confirms it. Record both with equal seriousness.
- **Time-box per hypothesis.** If one hypothesis is taking 30+
  minutes without converging, step back, write what you've got, and
  check with the user before continuing. Investigations sprawl; the
  user is the brake.
- **Write `investigation.md` as you go.** It's a running document,
  not something written at the end. Each evidence query adds a
  paragraph. This makes the final verdict easy to write and the
  workspace readable for future grep.
- **Don't expand the cohort silently.** If the investigation surfaces
  that the cohort is part of a bigger ring, surface that explicitly
  and ask the user whether to expand the workspace's scope or open
  a separate follow-up workspace. Cohort drift is a common failure
  mode.

## Output layout

```
{workspace_root}/{YYYY-MM-DD}_{slug}/
├── data/
│   ├── 01_cohort.sql           — cohort-producing query
│   ├── 01_cohort.csv           — its result (the user-id column the pipeline expects)
│   ├── 02_*.sql                — evidence queries, chronologically numbered
│   ├── 02_*.csv                — their results
│   └── ...
└── investigation.md            — verdict at top, then hypothesis,
                                  evidence narrative, prior context,
                                  recommended next step
```
