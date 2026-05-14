# fraud-investigation-skills

Agent skills for fraud-analyst work — fraud hunting, triage, and investigation. Drop into Claude Code, Claude Cowork, or OpenAI Codex to make your agent investigate fraud with structure and produce consistent, actionable results.

Built by [Finic](https://finic.ai).

## Why

Out of the box, AI agents asked to investigate fraud autonomously tend to run unstructured queries, hop between angles, and produce observations that may be unusual but difficult to attribute to fraud, as opposed to users doing weird-but-not-illegal stuff.

This skill imposes structure based on how real fraud teams work.

- A clear phase model — **hunting** (find new patterns), **triage** (rank suspicious data into prioritized alerts), **investigation** (drive a cohort to a written verdict).
- Disciplined linkage rules (when does a shared attribute actually link two accounts? when does cluster expansion drift into false coordination?).
- A reference library of common fraud typologies and the signals each produces.
- A standard workspace layout so every session is grep-able and auditable later.

## How to invoke it

Once installed, the skill triggers automatically on fraud-adjacent prompts. Examples:

- *"Anything weird in the last 7 days of signups?"* → hunting
- *"Rank these 200 accounts into alerts I should look at first."* → triage
- *"These 12 UIDs look related — investigate them."* → investigation
- *"Look into the spike in ACH returns last weekend."* → hunting → triage → investigation
- *"Check this signal — shared device key across new accounts."* → investigation

Just describe the work in natural language. The skill classifies the phase based on whether you already have a cohort and routes accordingly.

## Installation

You can either clone this repo with `git`, or download the files as a ZIP from the green **Code → Download ZIP** button on GitHub (then unzip into the install location below).

### Claude Code

Install at the user level so it's available across every project:

```bash
# clone
git clone https://github.com/finic-ai/fraud-investigation-skills.git \
  ~/.claude/skills/fraud-investigation

# or download + unzip
unzip fraud-investigation-skills-main.zip -d ~/.claude/skills/
mv ~/.claude/skills/fraud-investigation-skills-main ~/.claude/skills/fraud-investigation
```

Restart Claude Code, then verify with `/skills` or trigger it with a fraud-related prompt.

For a per-project install, use `.claude/skills/fraud-investigation/` inside the project root instead.

### Claude Cowork

Cowork installs *plugins* (bundles that contain skills) rather than raw skills. Two options:

**Easiest — use the Claude Code skill path.** If you're running Cowork via the Claude desktop app, the same user-level skills directory above is read by Cowork. Install per the Claude Code instructions; the skill becomes available in Cowork too.

**As a custom plugin.** Cowork's "Customize → Browse plugins" UI also supports uploading a custom plugin file. The repo doesn't currently ship a `.claude-plugin/plugin.json` manifest — if you need this path, fork the repo and add a minimal manifest, or open an issue and we'll add one.

### OpenAI Codex

For a per-repository install, place the files under `.agents/skills/` at the root of the repo you're working in:

```bash
# clone
mkdir -p .agents/skills
git clone https://github.com/finic-ai/fraud-investigation-skills.git \
  .agents/skills/fraud-investigation

# or download + unzip
mkdir -p .agents/skills
unzip fraud-investigation-skills-main.zip -d .agents/skills/
mv .agents/skills/fraud-investigation-skills-main .agents/skills/fraud-investigation
```

Codex auto-discovers skills under `.agents/skills/` walking up from your working directory.

For a global install, place the files anywhere on disk and register them in `~/.codex/config.toml`:

```toml
[[skills]]
path = "/absolute/path/to/fraud-investigation-skills"
enabled = true
```

Invoke with `/skills` in the Codex CLI/IDE, or just describe a fraud task.

## Customizing for your organization

The skill is designed to be extended. Every fraud team has its own terminology, named signals, schema, mitigation states, and known typologies — and the phase references explicitly tell the agent to consult organization-specific docs first when they exist.

To layer your own context on top:

1. Fork this repo (or clone and keep your changes local).
2. Add markdown files under `references/` covering things like:
   - Your team's named signals / KRIs / monitors and what each one means.
   - Your data schema and table names (so the agent doesn't hallucinate column names).
   - Org-specific typologies, jargon, prior incidents worth pattern-matching against.
   - Calibrated linkage thresholds for your population.
3. Reference the new files from the table in `SKILL.md` so the skill knows when to read them.

Organization-specific knowledge always overrides the defaults shipped here.

## Repo structure

```
fraud-investigation-skills/
├── SKILL.md                       — entry point; routes between phases
├── references/
│   ├── hunting.md                 — phase: hunting (find new patterns)
│   ├── triage.md                  — phase: triage (rank suspicious data)
│   ├── investigation.md           — phase: investigation (drive a cohort to a verdict)
│   ├── linkage-attributes.md      — when shared attributes link entities (strong / moderate / weak)
│   ├── fraud-patterns.md          — common typologies and the signals each produces
│   └── output-conventions.md      — workspace folder layout and slug rules
├── LICENSE
└── README.md
```

## About Finic

[Finic](https://finic.ai) builds AI agents for fraud and compliance teams.

## License

MIT — see [LICENSE](LICENSE).
