# PayPilot prompt governance policy (Verta)

**Applies to:** each change to the PayPilot system prompt, overlays and profiles. This includes a change of one word.
**Start date:** next week. Review this policy after the first 5 changes.

## 1. Roles
| Role | Who | Does |
|---|---|---|
| Requirement owner | Product or Support for behaviour and tone. Compliance for regulatory lines (§5, disclosure, thresholds, reviews). | Writes or approves the requirement text. Owns the business reason. |
| Change reviewer | QA | Does the verifiability test. Runs the probes. Writes the changelog entry. |
| Compliance sign-off | Compliance | Mandatory if the change touches §5 or a fee, threshold or disclosure rule. |

The owner and the reviewer are never the same person. If one person must do both, the change waits.

## 2. Ready for release
A change is ready only when all these conditions are true:
1. Each new or changed line passes the verifiability test: observable output, criterion, violation example.
2. The change does not contradict a line that is already in the prompt or in the user story. The reviewer reads the full assembled prompt, not only the diff.
3. Each changed line has a probe question. QA sends it ×3 on the new version and ×1 on the current version. There is no new failure. Old failures that remain are listed.
4. The changelog entry is complete. If needed, Compliance signs.

## 3. Changelog (`prompts/CHANGELOG.md`)
For each version: version, date, owner, reviewer, what changed (section + before/after summary), reason, evidence (finding IDs, probe distributions, links), what was not measured. Each overlay or profile that changes the assembled prompt is also a version.

## 4. Triggers for an out-of-cycle review
- A tariff, fee or knowledge-base fragment changes.
- A tool schema or tool output changes (for example, a new field or a changed `allowance_applied` logic).
- The model or the model settings change.
- A customer complaint or escalation about a figure, a promise or a disclosure.
- A new overlay or profile is enabled.
- Two probe runs give different answers to the same question about cost.
