# PayPilot system prompt: changelog

Each entry has: version, date, owner, reviewer, what changed, reason, evidence, what was not measured.

## v1.1 (2026-10-04)
- **Owner:** Product (Verta Payments)
- **Reviewer:** Andrii Lytviakov (QA)
- **Changed:**
  - §1 Role and tone: the D02 text ("Be deeply empathetic … radiate warmth") is replaced by a list of prohibited phrases, a prohibition of emoji, and a rule that the figure comes in the first sentence.
  - §5 Domain constraints: the D01 paragraph "ABSOLUTE COMPLIANCE PROHIBITION …" is replaced by "state the figure from a tool result or tariff fragment and name the source; if there is no figure, say so and point to the tariff schedule". The other §5 lines are not changed.
  - §6 Edge cases: the D03 text ("state concrete, plausible terms") is replaced by "state only the terms from search_knowledge_base; if there is no fragment, say that you have no information".
  - §8 Examples: was empty. 4 examples are added (SEPA fee, quote with allowance, quote without allowance, unknown product), plus the instruction "Do not copy their figures."
- **Reason:** D01 has three orders that cannot all be obeyed (contradiction). A test cannot check D02 (unverifiable). D03 orders the bot to invent terms, against §3 (over-constraint). The empty §8 gave no reference format (finding H29: 3 rate precisions, final line in different positions).
- **Evidence:** `homework/specification-review.md`, findings 1–3 and sections C and D. SWIFT fee: 0/5 answers give a figure. Spread hidden: 3/3. Promotional tone: 3/3. Invented product terms: 1/1. Invented quote validity: 2/3.
- **Not measured:**
  - v1.1 has not run on the stand. The probe questions Q1–Q6 must run ×3 on v1.1.
  - The §8 example figures come from `tariffs.md` and `app/engines/fx.py`. If the tariff or the rates change, the examples must be reviewed.
  - Frequency: there are only 1–5 runs for each question.
  - Not fixed in v1.1: §7 allowance wording (finding 6), freshness (finding 8), scope/execution (finding 4), the limits line that D01 deleted.
  - Compliance has not reviewed the new §5 paragraph.

## v1.0 (2026-09-24)
- **Owner:** Product (Verta Payments)
- **Reviewer:** none before the release. Post-release audit by Andrii Lytviakov (QA) on 2026-10-04.
- **Changed:** this is the first record. The lesson-01 build = `base.v1.md` + overlay D01 (replaces §5) + D02 (replaces §1) + D03 (replaces §6).
- **Reason:** baseline for the audit.
- **Evidence:** runs R01–R56 (`homework/paypilot-review-results.md`).
- **Not measured:** no review and no test runs before the release. Side effect: D01 deleted the base line "The applicable daily and monthly transfer limits are the ones on file for the customer's tier." Nobody recorded this.
