# Output Conventions

All work for a session lives in one workspace folder.

## Workspace folder layout

```
{workspace_root}/{YYYY-MM-DD}_{slug}/
├── data/
│   ├── 01_<descriptive-name>.sql
│   ├── 01_<descriptive-name>.csv
│   ├── 02_<descriptive-name>.sql
│   ├── 02_<descriptive-name>.csv
│   └── ...
└── investigation.md
```

`{workspace_root}` is the directory the team uses for fraud work.
Default to the agent's workspace; if the user names another path,
treat that as `{workspace_root}` for the session.

## Slug rules

- 2–4 words, kebab-case.
- Describe the substance, not the action. Good: `prepaid-email-onboarding`,
  `ach-reversal-new-banks`, `device-key-abc-loan-ring`.
  Bad: `triage-run`, `investigation-1`, `monday-hunt`.
- Rename the folder if the hypothesis evolves; don't keep multiple
  variants.

## data/

One `.sql` file paired with one `.csv` (or other result format),
numbered chronologically (`01_`, `02_`, …). The numbering is the run
order, so the files read top-to-bottom as the session unfolded — it's
a run log, not a ranking.

## investigation.md

The single source of truth for the workspace. The agent keeps it
updated as the work progresses through phases:

- **Hunting** — direction, angles tried, what was found, novelty
  check (grep across other workspaces).
- **Triage** — adds the ranked alert short-list with priority,
  exposure, linkage, and recommended next step.
- **Investigation** — adds hypothesis, evidence narrative, verdict,
  and recommended fraud-ops action. The verdict belongs at the top so
  a reader sees the conclusion first.

Write for a future reader (often the same user, weeks later). Prefer
short, dense sentences. Avoid throat-clearing. State what was found.

## Cross-workspace lookup

There's no separate index file — workspace folders are the
organization structure. To find prior work on a user, device,
attribute, or signal, grep `{workspace_root}` directly:

```
grep -rli "<user_id or attribute>" {workspace_root}
```

The investigation.md in each workspace is grep-able, and so are the
SQL files in data/. That's the resumption + novelty-check mechanism.
