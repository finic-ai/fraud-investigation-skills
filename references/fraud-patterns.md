---

## name: fraud-patterns

description: Commonly observed fraud patterns, organized by typology. A field guide for matching what the data is showing to a named typology, and for knowing which signals tend to co-occur within each. Used in hunting (what angles to try), triage (which typology a cluster fits and how to name it), and investigation (which evidence queries discriminate one hypothesis from another).
type: reference

# Fraud Patterns

A field guide to common typologies. Each entry covers what the
typology is, the classic patterns it produces, and the signals that
tend to show up in data. Signal strength labels follow
`linkage-attributes.md` (**strong** = cluster-defining on its own;
**moderate** = useful in combination; **weak** = slicing only).

This is a starting taxonomy. Organizations often use their own labels
— defer to the local naming if it exists. The line between typologies
is fuzzy in practice: a stolen-identity account that the victim later
disputes can look like first-party dispute fraud in the data. Use the
verdict that best fits the evidence, and note ambiguity rather than
forcing a clean bucket.

A given case is also often layered: ATO drains often funnel into mule
accounts; synthetic identities run bust-outs; first-party dispute
abuse appears alongside promo abuse on the same device cohort. Look
for the *primary* driver, then the connected typologies.

---

## Third-Party Fraud

Someone other than the legitimate account holder is the actor — the
true customer is a victim, not a participant.

### Account takeover (ATO)

An attacker gains access to a legitimate user's account and uses it.
The hallmark is a discontinuity in behavior: the same account starts
behaving like a different person.

Patterns:

- Login from a new device + new geo, followed within minutes or hours by a  
sensitive action (password change, payment-method add, withdrawal,  
external transfer, beneficiary add).
- Credential-stuffing fingerprint: many failed logins across many
accounts from the same IP / ASN, followed by sparse successes.
- 2FA tampering — disable 2FA, change recovery email/phone, add an
email-forwarding rule (to silence alerts).
- Removal of trusted contacts, suppression of SMS/email
notifications, or marking the attacker's device as trusted.
- Session-token replay: same session cookie used from two distant
geos in a short window.
- Drain pattern: password change → new payment method, external account link,  
or beneficiary→ maximum withdrawal, all within one session.
- Customer-support contact after the fact ("I didn't do this") — the
ground-truth signal, lagging by hours to days.
- Cross-account reuse of attacker infrastructure: same device key,
IP, or proxy across multiple unrelated victims (cluster-defining).

Signals:

- Device key / device-attestation mismatch vs. user's history —
**strong** when shared across multiple victim accounts.
- Login IP / ASN deviating sharply from baseline — **moderate**.
- Time-to-sensitive-action after login under N minutes — **moderate**.
- New-payee + max-withdrawal in same session — **moderate**.
- Carrier change / phone-port event preceding access (SIM swap, see
below) — **strong** when correlated with sensitive action.

### SIM-swap-enabled ATO

Attacker ports the victim's mobile number, then uses SMS-based 2FA
to reset credentials. Distinguished from generic ATO by the carrier
event preceding the access.

Patterns:

- Mobile carrier change / number-reactivation event within 48 hours
of credential reset.
- SMS 2FA delivered to a number whose carrier just changed.
- Frequently targets high-balance, crypto, or brokerage accounts.

### Social engineering and scams

The legitimate customer is manipulated into sending money or handing
over access themselves. The account holder is the actor, but they're
a victim — distinct from first-party fraud, where the account holder
is the adversary. Sub-types: romance, investment / pig-butchering,
tech support, government / IRS impersonation, lottery, family-emergency,
job / task scams, fake-charity, refund-overpayment.

Patterns:

- Out-of-pattern transfer to a first-time external recipient,
unusually large for the user.
- Escalating series: small "test" transfers followed by a large one.
- Recipient is a crypto exchange, prepaid card, or money-transfer
service when the user has no prior crypto / remittance history.
- Customer-support transcripts containing urgency markers
("emergency", "investment opportunity", "verify with the IRS"),
or evidence the user is on a phone call during the transfer.
- Demographics skew: older users, recently widowed, isolated. (Use
carefully — demographic signals alone are weak and risk false
positives.)
- Repeated transfers to the same external recipient over weeks
before "waking up" — the victim doesn't realize until much later.

Signals:

- New-recipient + large-amount + crypto/MTO destination — **moderate**.
- Concurrent active phone call / long session duration with form
activity — **moderate**.
- Memo line containing scam keywords ("investment", "tax", "fees",
"gift card") — **weak** (filter, not linkage).

### Business email compromise (BEC) / vendor invoice fraud

Attacker compromises or spoofs a business email account and redirects
payments. Often surfaces as a wire or ACH change-of-instruction.

Patterns:

- Vendor payment instructions changed via email shortly before a
scheduled payment.
- Wire to a never-before-used beneficiary at a different bank than
prior payments to the same vendor name.
- Display name matches a known executive but the underlying email is
a lookalike domain.
- Recipient account is a new business banking account, often opened
in the last 30–90 days.

### Card-not-present (CNP) fraud / stolen card

Stolen card credentials used at e-commerce or P2P merchants.

Patterns:

- BIN-testing: many low-value authorization attempts ($0.01, $1.00)
across many BINs from one IP / device.
- Sequential card numbers attempted from the same source.
- Shipping address differs from billing in ways consistent with
reship / freight-forwarding fraud (CMRA, freight forwarder,
hotel).
- High decline-to-approve ratio per device or IP.

### New-account / stolen-identity fraud

A real identity is used to open an account without the victim's
knowledge. The opener controls the account; the victim discovers it
later (often via credit report). The account itself is the fraud
vehicle — for cash-out, promo abuse, bust-out, or mule activity.

Patterns:

- KYC documents pass automated checks but don't match the selfie /
liveness signal.
- SSN appears across multiple accounts with different names (or with
the same name but different DOBs).
- Address is a CMRA, virtual mailbox, freight forwarder, or a known
fraud-farm address — cluster of unrelated accounts share it.
- Phone is recently-activated VOIP / non-fixed mobile / from a
prepaid carrier.
- Funding source name doesn't match account-holder name.
- Quick-close pattern: KYC passes, low or no activity, account
closes within 30 days (testing the kit).
- IP geo doesn't match claimed address; persistent VPN / Tor / proxy
use during onboarding.

Signals:

- Same identity-document hash across multiple accounts — **strong**.
- Same SSN across multiple accounts — **strong**.
- Same device key across many onboardings — **strong**.
- IP + browser fingerprint + signup-time cluster — **moderate** in
combination.
- VOIP carrier — **weak** alone, **moderate** combined.

---

## First-Party Fraud

The account holder is the adversary. The legitimate-looking account
is the fraud vehicle. The defining question is intent at onboarding
or at the point of the abusive action.

### Synthetic identity fraud

A fabricated identity built from a mix of real and fake PII —
typically a real SSN (often a child's, a deceased person's, or a
fabricated CPN) paired with a fabricated name, DOB, and address.
The "person" has no prior history, but the SSN may pass instant
verification.

Patterns:

- Thin or no credit file at onboarding, then rapid build-up via
authorized-user tradelines or secured cards.
- SSN was issued after 2011 (post-randomization) but is being used
by someone who claims to be older.
- SSN belongs to a minor or deceased person (Death Master File / SSA
inconsistency).
- DOB + name combo doesn't appear in public records or skip-trace
databases.
- Address has been associated with many SSNs over time (often
abandoned property, multi-family with high turnover, or a
fraud-farm address).
- Phone has been associated with many identities.
- A cluster of synthetic identities sharing partial PII — same DOB
with varying names, same address with varying SSNs, same employer
that doesn't exist.
- After credit is built, the synthetic runs a bust-out (see below) —
synthetic + bust-out is the classic combined typology.

Signals:

- SSN reused across accounts with different names — **strong**.
- DOB + address combination reused across accounts — **moderate**.
- Employer domain registered in the last 12 months and listed across
multiple unrelated accounts — **moderate**.
- Authorized-user tradeline-piggyback pattern visible at credit
bureau pull — **moderate**.

### Bust-out

A long-game first-party pattern: build credit and trust over months,
then max out all available limits and disappear. Common on credit
cards, lines of credit, and BNPL.

Patterns:

- Slow, on-time payment history establishing trust → sudden
maxing-out across multiple lines simultaneously.
- "Doubling" / payment-from-the-fraudster: a large payment is made
(often by ACH from a controlled external account or a kited
check), credit availability immediately re-opens, fraudster spends
against it, then the original payment is returned NSF or
reversed — net loss equals the spend + the returned payment.
- Cash-out channels: cash advances, gift-card purchases, gas-station
fuel-pump BIN-out, money-order purchases.
- Customer-service contact in the weeks before bust-out claiming
hardship or asking about credit-limit increases.
- Account abandoned shortly after maxing — no contact, no payment,
number goes dead.

Signals:

- ACH-payment-then-reversal pattern preceding a credit spike —
**moderate**, **strong** when clustered across users.
- Sudden utilization jump to >90% across multiple lines in <72h —
**moderate**.

### Dispute / friendly / chargeback fraud

The cardholder disputes legitimate transactions as unauthorized to
get refunds while keeping the goods. Distinguished from real ATO by
the dispute being filed by the actor who made the transactions.

Patterns:

- High dispute-to-transaction ratio per user, sustained over time.
- Dispute reasons inconsistent with the transaction (claiming "item
not received" on a digital good, on a transaction with delivery
confirmation to the verified address, or where the merchant has
evidence of use).
- Multiple disputes across multiple merchants in a short window.
- "Refund arbitrage": dispute filed after the merchant has already
refunded — user gets paid twice.
- Disputes follow a recognizable post-success pattern — the user has
had prior disputes resolved in their favor and the volume
accelerates.
- Rings: a cluster of accounts sharing a device key or address all
disputing through the same merchant or merchant category.
- "Item not received" disputes on shipments to known reship
addresses.

Signals:

- Disputes / transactions ratio per user above population p95 —
**moderate**.
- Shared device key across high-dispute accounts — **strong**.
- Same merchant disproportionately disputed across a cohort —
**strong** for ring detection.

### ACH-return / ACH-reversal abuse (first-party)

The user pulls funds in via ACH, spends or withdraws them, and then
the original ACH is returned — leaving the institution holding the
loss. R10 (unauthorized) and R05/R07/R51 (consumer / corporate
unauthorized) are the main return codes used adversarially. R01
(insufficient funds) abuse is a related but distinct pattern.

Patterns:

- ACH pull from a linked external account → fast spend / withdraw /
P2P out → R10 return on the original pull, days later. Net loss
= the spend.
- Same user repeats the pattern across multiple linked external
accounts. The external accounts often turn out to be unrelated
third-party accounts (i.e. theft of bank credentials from a
victim) or controlled by the fraudster.
- Burst funding: many small ACH pulls from many routing numbers in
a short window, each individually small enough to clear initial
velocity checks.
- Elevated ACH return rate on a specific cohort — clusters defined
by shared funding-source bank, shared device, or shared signup
cohort.
- First ACH pull happens immediately after account opening, often
for the max allowed amount, with funds drained the same day.

Signals:

- ACH-returned-as-unauthorized rate per user — **moderate**.
- Same external account number linked to multiple users — **strong**.
- Same routing number disproportionately concentrated in a cohort —
**moderate**, **strong** for small-bank originators.
- Time-from-ACH-credit to outbound-drain < 24h — **moderate**.

### Check fraud (first-party)

Patterns where the user is the depositor of fraudulent checks or is
otherwise abusing the check rails.

Sub-patterns:

- **Mobile-deposit fraud (RDC).** Deposit an altered, fictitious, or
third-party check by mobile, withdraw the funds before the check
clears or bounces.
- **Check kiting.** Move funds in circles between accounts at
different banks, exploiting float — each account looks funded only
because of an unposted deposit.
- **Duplicate deposit.** Same check deposited at multiple
institutions (deposit at your bank, then deposit again at another).
- **Counterfeit / washed checks.** Stolen real checks chemically
altered to change payee or amount, then deposited.
- **Stop-pay abuse.** Deposit a check, withdraw against the
availability, then claim non-receipt or that the check was
unauthorized.

Patterns:

- Deposit-then-withdraw within minutes — funds withdrawn against
Reg-CC availability before the check has actually cleared.
- New account with no prior check activity suddenly deposits a
large check.
- Multiple checks deposited from different payors, all bouncing.
- Sequential check numbers from a brand-new payor account.
- Image-match hits across institutions (same check, two banks).
- Customer claims they didn't deposit the check after the fact (the
return-then-dispute combo).

Signals:

- Same check image hash deposited at multiple institutions —
**strong**.
- Same payor account across many fraudulent-deposit accounts —
**strong**.
- Mobile-deposit + same-day withdrawal pattern per user —
**moderate**.

### First-party application / misrepresentation fraud

The applicant lies on their application to qualify for credit, a
loan, an account, or favorable terms. Distinct from synthetic in
that the identity is real — only the application data is falsified.

Patterns:

- Stated income materially inconsistent with verified flows (Plaid,
paystubs, tax transcripts).
- Employer that doesn't exist, or whose domain was registered in the
last few months.
- Stated assets inconsistent with linked-account balances.
- Address misrepresentation to qualify for a regional product or
rate.
- Identity-document tampering — name / DOB / address fields
altered, watermark anomalies.

### Loan stacking

Same borrower takes out multiple loans across multiple lenders in
rapid succession, before bureau reporting catches up. Each lender
sees a clean file; in aggregate the borrower is massively
over-leveraged and likely intends not to repay.

Patterns:

- Application velocity across lenders within 24–72 hours.
- Inconsistent stated income / employment across applications (if
cross-lender data is available via a consortium).
- Borrower disappears after the last loan funds.
- Funded loans drained the same day to a P2P or crypto destination.

### Promo / referral / bonus abuse

Exploiting onboarding incentives, referral bonuses, or promotional
credits at scale. Often runs adjacent to multi-accounting and
synthetic identity.

Patterns:

- Many accounts from the same device or device cluster all hitting
the promo trigger.
- Self-referral chains — A refers B refers C, all sharing device,
IP, or funding-source signals.
- Quick-cash: account opens, completes promo qualifier (a deposit, a
transaction), promo credit clears, funds withdrawn, account
abandoned.
- Cohort of accounts with identical onboarding-flow timing aligned
to the promo launch window.
- Gmail `+tag` or dot-trick variations of the same address across
multiple accounts.
- Disposable / temporary-email-provider domains.

Signals:

- Same device key across many promo-claiming accounts — **strong**.
- Funding source / payment instrument shared — **strong**.
- Email canonicalization (strip dots, drop `+tag`) collapses
multiple accounts to one — **strong**.

---

## Money laundering / mule activity

Accounts being used to move illicit funds rather than to defraud the
institution directly. The institution's risk is regulatory (BSA /
AML) and reputational. Mules can be witting (paid participants),
unwitting (recruited via job ads), or somewhere in between.

### Mule accounts — general

Patterns:

- Money-in / money-out same day or same week; balance hovers near
zero. Account is a conduit, not a wallet.
- Counterparties don't repeat — many one-time senders or one-time
receivers.
- Account-holder profile (age, stated occupation, prior activity)
doesn't match the flow volume (a student account moving $50k/mo).
- Sudden activation of a long-dormant account, often after a
password reset or contact-info change.
- Mule-recruitment fingerprints in support tickets ("I was hired as
a payment processor", "my friend sent this to me to forward").
- The account is funded entirely by external party, with the holder
taking a small "commission".

### Funnel accounts

One account receives from many unrelated counterparties, then
forwards to one or a few off-ramp destinations.

- Many-to-one inflow pattern over a short window.
- Outflows concentrated on a small number of destinations: an
overseas wire counterparty, a crypto exchange, a prepaid-card
loader, a money-transmitter.

### Layering

Long chains of transfers designed to obscure the origin of funds.

- Multi-hop P2P chains where each leg shows little economic purpose.
- Card load → P2P transfer → withdrawal → crypto on-ramp pattern in
rapid sequence.
- Crypto on-ramp followed by an off-ramp through a mixer or a
high-risk exchange.
- High-velocity transactions with no clear consumer or merchant
purpose.

### Structuring / smurfing

Splitting transactions to stay under regulatory or internal
reporting thresholds.

- Cash deposits clustered just under $10,000 (CTR threshold) or just
under organization-specific velocity limits.
- A cluster of related accounts each transacting just under the
same threshold.
- Same external counterparty splitting transfers across many
recipient accounts in the same narrow range.
- Transfers with amounts ending in patterns ($9,500.00, $9,850.00,
$9,900.00) suggesting threshold awareness.

### Pass-through / nested-account activity

- Crypto exchange or money-services-business deposits / withdrawals
not consistent with the stated business of the account holder.
- Activity consistent with a downstream user base (the account is
effectively acting as an unregistered MSB).

---

## Other patterns worth knowing

### Triangulation fraud (e-commerce)

A fake or compromised storefront takes legitimate orders from real
customers using stolen card data to fulfill them via a real
merchant. The end-consumer is happy; the cardholder eventually
disputes; the real merchant takes the chargeback.

### Refund fraud (merchant-side)

Insider or coordinated actor processes refunds without a matching
sale, refunds to a different card than the original, or refunds for
more than the sale amount.

### Card / payment-instrument cloning

Skimmed or cloned card data used at point of sale or ATM. Patterns:
geographic impossibility (used in two cities within minutes), prior
transactions at a known-compromised merchant in the same week.

### Insider fraud

Employees with system access committing fraud directly or
collaborating with external actors. Signals: access patterns
inconsistent with role, after-hours access, manipulation of
specific accounts the employee is associated with.

### Elder financial exploitation

A subset of APP scams plus first-party patterns where a family
member or caregiver controls a vulnerable adult's account. Pattern
of withdrawals or transfers that don't match the account holder's
prior behavior, often combined with the holder no longer being the
session actor.

---

## How to use this list

- In **hunting**, scan the list for typologies that fit the
direction; pick angles that match the strongest signals listed.
- In **triage**, name each candidate cluster with the typology it
best fits — the `alert` column should be readable as
`{typology}-{linkage}` (e.g. `ato-device-key-X`,
`synthetic-ssn-cluster-Y`, `ach-return-bank-Z`).
- In **investigation**, use the typology's signals as a checklist of
evidence queries — if the hypothesis is "this is a bust-out", the
evidence queries should target ACH-payment-then-reversal,
utilization-spike, cash-out-channel concentration, and post-spike
abandonment.
- Where a case shows signals from multiple typologies, record the
combination in the verdict (e.g. *"synthetic identity onboarding
ring transitioning to bust-out"*). Don't force a single label.

