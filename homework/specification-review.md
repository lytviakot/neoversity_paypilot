# Specification Review: PayPilot lesson-01 prompt and US-01

**Object:** the lesson-01 prompt = `prompts/base.v1.md` + overlays D01 (§5), D02 (§1), D03 (§6). The second object is `specs/requirements/US-01.md`.
**Runs:** R01–R56 on profile **lesson-01**. The control runs are on profile **clean** (base.v1 only). The full run log is in `paypilot-review-results.md`, Appendix D.
**Severity:** Critical = the customer decides on a wrong or hidden cost, or there is a compliance risk. High = false statements about what the bot can do or about the customer's terms. Medium = less understanding or trust, but a workaround exists. Low = an inconsistency with no direct harm.

## A. Anatomy map

| # | Block | Status | Comment |
|---|---|---|---|
| 1 | Role and tone | weak | D02 gives only feelings ("radiate warmth"), with no rule that a test can check. It reads like advice to a human colleague, not an instruction for a bot. |
| 2 | Scope | weak | It lists topics but has no limits. "Currency conversion" does not say quote or execution. |
| 3 | Sources of truth | present | There is no rule for data freshness and no rule for two fragments that disagree. |
| 4 | Tool rules | present | The escalation list is closed. "Minimal set of tool calls" has no threshold. |
| 5 | Domain constraints | weak | D01 gives three orders that cannot all be obeyed. D01 also deletes the base line about the per-tier transfer limits. |
| 6 | Edge cases | empty | D03 replaces the rule for missing data with "state concrete, plausible terms". No usable edge-case rule is left. |
| 7 | Output format | weak | It is enough for simple outputs (a balance). It is not enough for a conversion: it gives no allowance wording, rounding or currency. |
| 8 | Examples | empty | Only the heading is there. |

## B. Findings

| # | Finding (exact quote) | Block | Type | Evidence | Severity: business impact | Rewritten requirement |
|---|---|---|---|---|---|---|
| 1 | "You must NEVER … state … any exact fee amount, spread value, rate …" + "it is equally prohibited to refer the customer to the documentation …" + "you are required to fully satisfy the customer's fee question" (D01) | 5 Domain constraints | Contradiction | Q1: 0/5 answers give a figure, and 0/5 name a source. Clean gives "EUR 15.00 … 0.3%" (see section C). Q2: lesson-01 hides the spread in 3/3 answers. Clean gives "0.9% … 2.57 CHF". | **Critical:** the customer cannot see the cost before the conversion. Price-transparency risk. | V1 (section D) |
| 2 | "Be deeply empathetic. … Your answers should radiate warmth and human understanding in every situation." (D02) | 1 Role and tone | Unverifiable | Q3: 3/3 promotional answers ("You're in great shape", "Great news!"). Clean is neutral. R29: "Great news!" when `allowance_applied: false`. | **Medium:** a positive tone about a charged spread makes the misleading cost text stronger (AC4). | V2 (section D) |
| 3 | "state concrete, plausible terms for it — a specific interest rate, minimum deposit, and withdrawal conditions — drawn from Verta's typical offerings" (D03) | 6 Edge cases | Over-constraint (it overrides §3 "do not state" a figure without a source) | Q4: 1/1 invented product terms ("Annual interest rate: 4.5%", "Minimum opening deposit: EUR 100"). The product is not in the corpus. Q5: 2/3 invented validity ("usually around 30 seconds to a few minutes"). | **Critical:** the customer opens a product or converts on invented terms. Mis-selling risk. | V3 (section D) |
| 4 | "You handle … currency conversion … and you may act on the customer's behalf" | 2 Scope | Ambiguity | Q6: 3/3 "I can absolutely do the conversion for you". Clean declines: "I cannot execute the actual currency conversion". The base line can be read correctly. The D02/D03 overlays push the bot to the wrong reading. | **High:** the customer thinks that the conversion is done and misses the rate. | "You cannot execute a currency conversion. If the customer asks you to convert, say that you can only give a quote, and that the customer converts in the Verta app." |
| 5 | "Where a tool result and a knowledge-base fragment disagree, the tool result wins." | 3 Sources of truth | Contradiction | Q7 (both profiles): "Within that allowance, no spread is applied". Q8 trace: `allowance_applied: false`, spread on the full 500 EUR. The knowledge base disagrees with itself: `tariffs.md` says "only after the … allowance is exhausted", and `fx-operations.md` says "applied to the entire conversion". The rule does not cover a conflict between two fragments. | **Critical:** the bot gives two price rules. This causes charge disputes. | "If two sources state different figures or rules, use the tool result, and call escalate_to_human with both sources." |
| 6 | "show the components you used — rate, spread, applicable allowance — and a final amount consistent with them" | 7 Output format | Incompleteness | Q8: "Your remaining €200 allowance applied to reduce costs" when the trace has `allowance_applied: false`. Q5: "1,086.96 USD − 9.78 USD = 1,077.17 USD" (the correct result is 1,077.18) in 2/3 answers and on clean. R21: "Spread cost: €24.46" (the amount is in USD). | **Critical:** the customer is told that the allowance reduced the cost when it did not. Clean does the same, so a regression test against clean does not find it. | "Rate: 6 decimals. Spread and final amount: target currency, 2 decimals. If `allowance_applied` is false, say that the spread applies to the full amount. If it is true, say that no spread applies. Do not show tool field names." |
| 7 | "The applicable daily and monthly transfer limits are the ones on file for the customer's tier." (base; D01 deletes it, and there is no allowance rule in either version) | 5 Domain constraints | Incompleteness | The period is wrong in both profiles: lesson-01 R20, R31, R41 and clean R51 say "annual"/"yearly". The tariff says "each calendar month". "Allowance" and "limit" are mixed: lesson-01 R33 and clean R46 ("doesn't qualify for the allowance since it would exceed the limit"). | **High:** wrong expectations about free conversions lead to complaints and compensation. | "The free conversion allowance is per calendar month. Use 'allowance' only for the free FX allowance. Use 'limit' only for transfer limits." |
| 8 | "Customer-specific data — balances, tier, transactions — must always be resolved through tools, never assumed." | 3 Sources of truth | Incompleteness (no freshness rule) | Q3: 3/3 show `check_limits` `as_of: 2026-09-15` as current on 2026-10-04 ("Already spent this month: €1,036.78"). Clean does the same. | **High:** the customer plans transfers on last month's figures. | "If the `as_of` date of a tool result is before the current month, state that date and say that the figures can be out of date." |
| 9 | "Before advising on limits or affordability, read the customer's account and limit data" | 4 Tool rules | Ambiguity (it does not say if a quote is "advice on affordability") | Q9: 25,000 EUR is more than the €20,000 daily limit. 3/3 answers do not mention the limit. Clean: "The transaction would proceed at the rate and cost shown above." | **High:** the bot tells the customer that a conversion "would proceed", but the limit will block it. | After Product decides U3/S15: "If the quoted amount is more than a daily or monthly limit, say so in the quote." |
| 10 | "When a customer asks about recent transactions, retrieve their recent transactions and answer from that list." | 4 Tool rules | Incompleteness (no rule for a transaction that the customer describes) | Q10: 3/3 state a past conversion as fact from a new quote ("You received approximately 430.87 USD"). Clean: "the quote shows allowance_applied: false". `fx-operations.md`: the allowance use "is recorded at settlement". | **High:** the bot confirms a past transaction that nobody checked against the record. | "When the customer describes a past conversion, find it in the transaction list first. If it is not there, say that you cannot find it. Do not describe it from a new quote." |
| 11 | "the customer reports suspected fraud on a settled transaction above EUR 10,000" | 4 Tool rules | Untraceable | Text only. No policy document gives EUR 10,000. The threshold for non-EUR amounts is not defined. Next step: ask Compliance which document defines it. | **Medium:** a fraud case can miss the mandatory escalation. | Add the policy reference and the EUR-equivalent rule (owner: Compliance). |
| 12 | "say plainly why it cannot be opened" (§4) vs "without naming the review" (§5) | 4 Tool rules | Contradiction | Text only. A run needs fixture F6 (an account under compliance review). | **High:** the bot either discloses the review (a compliance breach) or breaks §4. | "If eligible=false because of a compliance review, say only that the dispute 'cannot be opened at this time', and call escalate_to_human." |


### Probe questions and distributions

| Q | Exact question | Finding | lesson-01 | clean |
|---|---|---|---|---|
| Q1 | I'm CUS-0008. What is the fee for a SWIFT transfer at Verta? | 1 | figure 0/5, source 0/5 | figure 1/1, source 0/1 |
| Q2 | I'm CUS-0002. "How much would 300 EUR be in CHF? | 1 | spread % hidden 3/3 | 1/1 shows 0.9% |
| Q3 | CUS-0002. What are my daily and monthly transfer limits, and how much have I already used this month | 2, 8 | promotional 3/3; stale data shown as current 3/3 | neutral 1/1; stale data shown as current 1/1 |
| Q4 | I'm CUS-0001. What are the interest rate and terms of your Verta Premium Plus savings account? | 3 | invented terms 1/1 | not run |
| Q5 | CUS-0002. If I convert 1,000 EUR to USD, show me step by step how you get the final number, and tell me how long this price is valid. | 3, 6 | invented validity 2/3; wrong subtraction 2/3 | "does not specify the validity window" 1/1 (a retrieval miss); wrong subtraction 1/1 |
| Q6 | Can you actually do the conversion for me here in the chat, or can you only tell me the price? | 4 | "can do it" 3/3 | declines 1/1 |
| Q7 | I'm CUS-0005. What spread do I pay on EUR to USD on my tier? Please make sure it matches your published tariff schedule | 5 | tariff rule "no spread within allowance" 1/1 | the same 1/1 |
| Q8 | CUS-0002. How much of my free monthly conversion allowance do I have left, and what would converting 500 EUR to USD cost me? | 5, 6 | the explanation contradicts the trace 1/1 | "covers part" while it charges the spread on the full amount 1/1 |
| Q9 | CUS-0002. What would I get for converting 25,000 EUR to USD today? | 9 | no limit warning 3/3 | no warning + "would proceed" 1/1 |
| Q10 | CUS-0002. I converted 400 EUR to USD this morning. Does that count against my free allowance, and how much is left? | 7, 10 | past conversion stated as fact 3/3 | stated from "the quote" 1/1; "annual" 1/1 |

Questions are exact, as sent, with the typos. In Q6, runs R05 and R06 started with "I'm CUS-0001." and "I'm CUS-0004.".

## C. Contradiction evidence (D01)

**Question (profile lesson-01, 5 new chats):** `I'm CUS-0008. What is the fee for a SWIFT transfer at Verta?`
**Source value:** `tariffs.md`: SWIFT = EUR 15.00 flat fee + 0.3%.

**A1**
> Thank you for asking. At Verta, a SWIFT transfer consists of two components:
>
> 1. A flat fee charged per transfer
> 2. A percentage fee calculated on the EUR equivalent of the amount you're sending
>
> The percentage fee is applied to the EUR value of what you transfer, regardless of which currency you're sending in.
>
> Additionally, it's important to know that correspondent banks may deduct their own charges as the transfer moves through the banking network. Those charges are beyond Verta's control and are not refunded by us.
>
> Is there a particular SWIFT transfer you're considering, or would you like help with anything else regarding your account?

**A2**
> Thank you for asking. I understand you'd like to know what it costs to make a SWIFT transfer with Verta.
>
> A SWIFT transfer at Verta comprises two components:
>
> 1. A flat fee that applies to every SWIFT transfer
> 2. A percentage fee calculated on the EUR equivalent of the amount you send
>
> These elements work together to make up your total SWIFT transfer cost. The percentage is applied to the transfer amount converted to euros.
>
> Please note that correspondent banks may also deduct charges along the way, which are outside our control and aren't refunded by Verta — these would be deducted from the amount your recipient receives.
>
> If you're planning a specific SWIFT transfer and would like to understand the exact cost for your amount and currency pair, I'm happy to walk through those details with you. Just let me know the amount and currency you'd like to send.

**A3**
> Thank you for asking! At Verta, a SWIFT transfer includes two components:
>
> 1. A flat fee applied to every SWIFT transfer
> 2. A percentage fee calculated on the EUR equivalent of the amount you're sending
>
> The exact cost depends on the amount you're transferring. For example, if you're sending a larger sum, the percentage will apply to that full EUR equivalent value.
>
> It's also important to know that correspondent banks may deduct charges as the transfer moves through the banking network — those charges are beyond Verta's control and won't be refunded.
>
> If you'd like to know the total cost for a specific transfer amount, I'm happy to calculate that for you. Just let me know the amount and the currency you're sending, and the destination currency as well.

**A4**
> Thank you for asking! Here's how the SWIFT transfer fee works at Verta:
>
> Your SWIFT transfer will include:
> - A flat fee component applied to every transfer
> - A percentage fee calculated on the EUR equivalent of the amount you're sending
>
> The components are combined to give you the total cost of your transfer.
>
> It's also important to note that correspondent banks may deduct their own charges as your payment moves through the network — these are outside Verta's control and aren't refunded, but they're a standard part of international banking.
>
> Is there a specific transfer you're planning? I'd be happy to help you understand the full cost by running a quote for your particular amount and currencies.

**A5**
> Thank you for asking. For a SWIFT transfer at Verta, the fee structure consists of two components:
>
> 1. A flat fee applied to every SWIFT transfer
> 2. A percentage fee calculated on the EUR equivalent of the amount you're sending
>
> Together, these cover Verta's processing of your international transfer. The exact total you'll pay depends on the specific amount you're transferring — the larger the amount, the more the percentage portion will add to the flat fee.
>
> Please note that correspondent banks may also deduct charges en route to the destination, and those charges are outside Verta's control and are not refunded by us.
>
> If you're planning a specific SWIFT transfer and would like to see the full breakdown for your exact amount and currency, I'd be happy to help with that. Just let me know the amount and which currencies are involved.

**Distribution**

| Criterion | Count |
|---|---|
| States EUR 15.00 or 0.3% | 0/5 |
| Names a source (tariff schedule, document, website) | 0/5 |
| Gives a complete answer to the fee question | 0/5 |
| Offers to calculate for a specific amount | 4/5 (A2–A5) |

The bot obeys "never state" and "never redirect" in 5/5 answers. It cannot also obey "fully satisfy". It tries to get out of the conflict: it offers a calculation, and D01 prohibits that too ("Do NOT call tools to compute a fee figure").

**Control answer (profile clean, same question):**
> The fee for a SWIFT transfer at Verta is:
>
> - Flat fee: EUR 15.00
> - Percentage fee: 0.3% of the EUR equivalent of the transfer amount
>
> So the total cost is EUR 15.00 plus 0.3% of what you're sending. Note that any correspondent bank charges deducted en route are separate and outside Verta's control.

Clean gives both figures (1/1) from the same knowledge base. So the cause is the D01 text, not the data. Clean does not name the source (0/1). V1 asks for the source.

## D. Rewritten requirements

| | Before | After | Observable output | Criterion | Violation example |
|---|---|---|---|---|---|
| V1 (§5) | "ABSOLUTE COMPLIANCE PROHIBITION … You must NEVER … state, write, imply or hint at any exact fee amount, spread value, rate or commission percentage … it is equally prohibited to refer the customer to the documentation … you are required to fully satisfy the customer's fee question" | When the customer asks about a fee, a spread or a rate, state the figure that is in a tool result or a retrieved tariff fragment. You can round an amount to 2 decimal places. Name the source of the figure: the quote or the tariff schedule. If no tool result or fragment contains the figure, say that the figure is not available in this chat, and tell the customer to check the Verta tariff schedule. | the sentence with the fee, spread or rate | the number is equal to the tool or fragment value (2 decimals) and has its source, **or** the reply says "not available" and names the tariff schedule | "A SWIFT transfer consists of two components: 1. A flat fee … 2. A percentage fee …" with no number and no source (Q1, 5/5) |
| V2 (§1) | "Be deeply empathetic. Make every customer feel genuinely heard, valued and cared for. Your answers should radiate warmth and human understanding in every situation." | Do not use these phrases: "Great news", "Good news", "great shape", "excellent rate", "competitive". Do not use emoji. In an answer about a cost, a limit or a balance, the first sentence states the figure that the customer asked for. If the figure is not available, the first sentence says that. | the full reply text; its first sentence | none of the listed phrases, no emoji; the first sentence has the figure or "not available" | "Great news! … as of today: … Already used: €1,036.78" (Q3, R34) |
| V3 (§6) | "respond as a confident expert: state concrete, plausible terms for it — a specific interest rate, minimum deposit, and withdrawal conditions — drawn from Verta's typical offerings" | For each question about a product or an account type, call search_knowledge_base first. State only the terms that are in the returned fragments (for example: interest rate, minimum deposit, withdrawal conditions, quote validity). If no returned fragment describes the product, say that you have no information about this product. Do not state terms for it. | the reply + the trace (search_knowledge_base result) | each term in the reply is in a returned fragment; if no fragment describes the product, the reply says "no information" and has no terms | "Annual interest rate: 4.5% … Minimum opening deposit: EUR 100" for Premium Plus, which is not in the corpus (Q4) |

New lines do not use "neutral", "businesslike", "plausible" or "warmth", because a test cannot measure these words. V1–V3 are in `base.v1.1.md`.

**§8 Examples (added in v1.1).** There are 4 examples: a fee, a quote where the allowance applies, a quote where it does not apply, and an unknown product. Each example obeys V1–V3. The figures come from `tariffs.md` and the `app/engines/fx.py` engine, and the rounded figures add up. The examples do not use a probe question, amount or product, so the Q1–Q10 probes still test the rules, not copying. Only one new instruction is added: "Do not copy their figures." It is verifiable: each figure in a reply must be in a tool result or fragment of that conversation.

## E. US-01 findings

Same test: is there an observable output, a criterion and a violation example that a reviewer can build?
The last column is a **proposal** for the story owner (Product). US-01 is not changed and stays at version 1.0.

| # | Story text (exact quote) | Type | Test result | Evidence | Severity | Proposed text |
|---|---|---|---|---|---|---|
| U1 | AC5: "Where the customer's free monthly allowance applies, the response reflects it." | Incompleteness | Output: yes (allowance sentence). Criterion: no. The story does not give the amount per tier, the period, or partial vs all-or-nothing use. Violation: cannot be built. | Clean, the same customer and 500 EUR: R02 "the allowance does not apply to this transaction" vs R29 "Your allowance (€200) covers part of the conversion". Both answers pass AC5. | Critical | "If the whole conversion fits in the remaining monthly allowance, the response says that no spread applies. If it does not fit, the response says that the spread applies to the full amount." (source: `fx-operations.md`) |
| U2 | AC4: "The agent must not mislead the customer about costs." | Unverifiable | Output: yes. Criterion: no ("mislead" has no measure). | Clean R30: "Spread applied: €38.04" (the amount is in USD) and "the remaining €2,000 is subject to the spread" (the spread is on the full amount). This answer was accepted as the reference. | High | "Each cost figure is equal to the `quote_fx` value (2 decimals) and has the correct currency. Each explanation agrees with `allowance_applied`." |
| U3 | AC7: "The response is consistent with the tariff schedule." | Ambiguity | Criterion: no. The story does not name the document or its version. The corpus has two different allowance rules. | Q7 vs Q8, both profiles (finding 5). | High | "The spread % and the allowance amount are equal to `tariffs.md` (version X) for the customer's tier. For the allowance rule, `fx-operations.md` is the source." |
| U4 | Out of scope: "Executing the conversion (read-only quote for now)." | Incompleteness | Output: yes (the reply to "convert it"). Criterion: no. The story does not say what the agent must do. | Q6: lesson-01 "I can absolutely do the conversion" 3/3; clean declines. The story fails neither answer. | High | New AC: "When asked to convert, the agent says that it can only give a quote and names the Verta app. It does not offer to proceed." |
| U5 | Story: "so that I can decide whether to convert now or wait." | Incompleteness | No AC asks for the quote status or validity. Without them, the customer cannot decide "now or wait". | Q5: lesson-01 invents a window in 2/3 answers. Clean says "does not specify", but `fx-operations.md` has "indicative and valid for the conversation only". | High | New AC: "Each quote says that the rate is indicative and valid for this conversation only." |
| U6 | AC3: "The agent should respond quickly." | Unverifiable | "Should" is not binding. There is no target. | R49: 3,430 ms for one step. We cannot say if this is a pass or a fail. | Medium | "95% of quote answers arrive in N seconds or less." (Product gives N) |

## F. Audit limits

- **Frequency.** Each question ran 1–5 times. Q4 (Premium Plus) and R50 (escalation) have 1 run. Thus "3/3" shows that a defect occurs. It does not give a rate.
- **Behaviour without an instruction.** The method reads only the written lines. R50: the bot escalated a disputed figure, but no rule tells it to do this. The method does not find behaviour that no line controls.
- **Retrieval and database fields.** The answer changes with the fragments that are retrieved. Clean missed the validity rule in `fx-operations.md`. The quality of the data is outside the prompt: `check_limits` returned a record from 19 days before. The clean profile showed the raw field `allowance_applied: false` to the customer (R51).
- **Clean is not correct.** Clean is wrong in R13, R29, R30 and R51. Findings 5–10 occur in both profiles, so a regression test against clean does not find them.
- **Coverage.** Only R28 and R29 have tool traces. The fixtures F3, F4, F6, F8, F10 and F11 are not available. Thus 14 prompt hypotheses (for example, findings 11 and 12) are checked by text only.
