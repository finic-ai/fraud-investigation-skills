# Linkage Attributes

Which attributes are strong enough to define a cluster on their own,
which are only useful in combination, which are slicing dimensions only,
and how to decide which category a given attribute falls into. Used in
hunting (cluster discovery), triage (cluster boundary work), and
investigation (avoiding silent cohort drift).

## The single most important rule

**Never expand a cluster on a weak signal alone, and never on a single
moderate signal alone either.** Linkage on weak or single-moderate
signals is the #1 way fraud investigations produce false coordination
narratives — finding a "ring" that's actually just shared infrastructure
used by unrelated actors.

## The three categories

The category an attribute falls into depends less on what the attribute
*is* and more on how concentrated it is in the target population
relative to the general population. The ratio test (next section) is the
definition; the lists below are starting defaults.

### Strong — links entities on this attribute alone

An attribute qualifies as strong if **two entities sharing the value is,
on its own, sufficient evidence of linkage.** Two ways this happens:

**Inherently unique identifiers.** Hard to spoof or share by accident,
unique enough that two unrelated entities sharing it is highly unlikely,
durable for a single entity over time.

- Hardware device keys / device-attestation identifiers.
- Payment instrument tokens (same underlying card or bank account).
- Direct relationship signals (P2P transfers between entities, shared
  owners, shared beneficiaries).
- Identity-document hashes (same KYC document across multiple
  accounts).

**Low-occurrence-in-general, high-occurrence-in-target identifiers.**
The attribute may exist in the general population, but it's rare there
and disproportionately concentrated in the target cohort. Examples:

- A merchant or counterparty ID that only ~50 entities in the general
  population have ever transacted with, where 38 of those 50 are in
  the target cohort.
- A rare ACH originator, wire counterparty, or business-bank routing
  number that appears in a few dozen accounts overall.
- A long-tail device fingerprint, app-version string, or carrier
  identifier appearing in only dozens of entities population-wide.
- A specific signup-flow variant or referral code rare across the
  population but common in the cohort.

The test is **concentration**, not absolute rarity. See "the ratio test"
below.

Two entities sharing a strong attribute defines a cluster on that
attribute alone.

### Moderate — links entities only in combination

A moderate attribute is too common to define a cluster on its own but
useful as a corroborating signal when multiple moderate attributes
overlap on the same cohort.

- **IP address.** Shared via NAT, mobile carriers, VPNs, public wifi —
  match alone is not evidence, but match plus other co-occurring
  signals can be strong.
- **Browser fingerprint.** High entropy but easily shared in shared
  computing environments and easily spoofed.
- **Uncommon email domains.** A small-business or unusual ESP domain
  shared by a few dozen entities is more meaningful than `@gmail.com`
  but less than a strong identifier on its own.
- **Device model + OS version combination.** Common enough alone, but
  a useful co-occurring signal.
- **Funding-source bank (uncommon institutions).** Particularly for
  small regional banks, credit unions, or non-mainstream fintechs.

Moderate attributes are useful as **filters** and as **co-occurring
evidence**. Two or three overlapping moderate attributes can rise to
cluster-defining; see "Combining moderate attributes" below.

### Weak — slicing dimensions only, never linkage

A weak attribute carries essentially no signal pairwise — the
attribute is reused legitimately by such a large population that
overlap between any two entities is expected. Use these for *slicing*
the data ("look at signups from this country", "compare this hour
window to baseline") but never as a linkage signal, even in
combination.

- User-agent strings.
- Date of birth.
- Common email domains (`@gmail.com`, `@yahoo.com`, etc.).
- Phone area code.
- Signup time (hour or day bucket).
- Country / state of residence.
- Language and locale settings.
- Major-bank funding source (Chase, BofA, Wells, etc.).

Weak attributes are filters and report dimensions, never linkage
signals.

## The ratio test (investigation-phase only)

The defaults above (device key = strong, IP = moderate, user-agent =
weak, etc.) are enough for hunting and triage. **Don't try to compute
the ratio test during hunting or triage** — getting `n_general` and
`general_size` typically means an extra query against the full
population, which is expensive and slows the partner-with-the-user
cadence those phases need. In hunting and triage, go off the qualitative
defaults plus your judgement. If something looks like a strong
attribute, treat it as strong; if it's moderate, look for combinations.

The formal ratio test belongs in **investigation**, when the case is
already scoped and the cost of a couple of extra queries is justified
by the need to defend a conclusion. The metric is **lift**:

```
lift = (n_target / target_size) / (n_general / general_size)
```

For low-count attributes (e.g. 5 in target, 5 in general), pair lift
with a significance check — **Fisher's exact test** (or the
hypergeometric distribution) on the 2×2 contingency table — to drop
noise from small samples.

Thresholds:

- **Strong:** lift ≥ ~100 with `n_target ≥ ~20`.
- **Moderate:** lift between ~5 and ~100, or high lift with low
  `n_target`.
- **Weak:** lift < ~5.

The underlying framework is **Fellegi-Sunter probabilistic record
linkage** (per-attribute log-likelihood ratios summed across
attributes); **Weight of Evidence (WoE)** is the same construct under
a different name. How to actually compute lift and Fisher's exact
(SQL, pandas, scipy, a record-linkage library like Splink, etc.)
depends on the environment — pick what fits.

These thresholds are starting defaults; organization-specific reference
docs may calibrate them to the organization's population.

## Combining moderate attributes

Two or three overlapping moderate attributes can rise to
cluster-defining if the joint concentration is high enough. Examples:

- **Same IP + same browser fingerprint + same signup window + same
  uncommon email domain.** Each alone is moderate; together,
  overwhelming.
- **Same device-model+OS + same funding-source bank + same
  time-of-day activity pattern.** Three moderate attributes
  overlapping on a small cohort.

The judgement call: would unrelated entities plausibly all match this
combination at the rate observed? When in doubt, run the sanity check —
how many entities in the broader population match the same
combination? If the answer is 10,000, your "cluster" is just a common
segment. If the answer is 50, you've found something.

## Organization-specific overrides

Every organization's domain has nuances. The organization's reference docs
(commonly `.claude/skills/<this-skill>/` or `docs/`) may list:

- Organization-specific strong attributes (a particular session token, a
  device-attestation field).
- Attributes that look strong but have known false-positive failure
  modes ("don't cluster on attribute X across institution-Y users —
  they share by design").
- Combinations that have historically been productive vs.
  unproductive.
- Calibrated lift / significance thresholds based on the organization's
  population sizes.

Defer to the organization's references when they exist. The defaults above
are starting points, not absolutes.

## How this shows up in each phase

**Hunting — go off vibes.** Use the qualitative defaults: prefer
pivots on attributes that look strong (device key, payment token, a
counterparty that the cohort surfaces unusually often), use moderate
attributes as slicing dimensions, ignore weak attributes for linkage.
Don't stop to compute lift — the goal of hunting is fast iteration on
plausible patterns, and a full ratio test would burn the cadence. If
an attribute *looks* concentrated in your cohort relative to what you'd
expect, treat it as a candidate strong attribute and keep going.

**Triage — still mostly vibes, surface the type to the user.** When
proposing cluster boundaries, label each by linkage type (e.g. "cluster
A: 80 users sharing strong attribute X; cluster B: 50 users sharing 3
moderate attributes in combination"). The user usually has the context
to validate whether the linkage holds without you having to query the
general population. Skip the formal ratio test unless the user
specifically asks for it, or unless a cluster's coherence is genuinely
contested.

**Investigation — actually compute the ratio.** The case is scoped,
the cohort is known, and the cost of a couple of extra queries to get
`n_general` is justified by the need to defend the conclusion. Run the
ratio test on any attribute the case rests on:

- Confirm strong attributes really do have the concentration the
  defaults assume — calibration in the organization's population may
  differ.
- For combined-moderate clusters, compute the joint concentration to
  rule out coincidence.
- When the question is "should this cohort be expanded on a newly
  surfaced attribute?", the ratio test is what answers it. Expand on
  strong-attribute overlap, expand cautiously on combined-moderate
  overlap, never on weak-attribute overlap or on a single moderate
  attribute alone.

## The cluster-expansion failure mode

The pattern to watch for: a case starts with 8 entities sharing a
strong device key. The investigation surfaces that those 8 entities
also share an email domain. The model then expands the cohort to "all
entities with that email domain who hit our signup signal" — now 800
entities, 99% of whom have nothing to do with the original 8.

This is silent cohort drift, and it ruins investigations. Stay
disciplined: when an investigation surfaces a new linkage candidate,
surface it to the user as a question — *"should we expand the cohort
using this new attribute, or treat the original 8 as the case and
the broader population as a separate inquiry?"* — rather than
silently expanding. Run the ratio test before recommending expansion.
