# Draft a follow-up email and send only after approval — agent run diagnosis

12 October 2026 · Tester: Josh Woodman · Session date: 2026-10-12 · Skill version: 2026-10-13.4 · Prepared with Claude

> **Summary** — the text to paste into the tickets is in [Draft ticket content](#draft-ticket-content) at the end
>
> - **What went wrong:** The approval flow worked: all six SR-15779 criteria were met and one email, matching the approved Wednesday draft, was sent after approval. Two minor issues sit outside the ticket: the agent kept searching the inbox long after it had found the recipient's address, and it called the recipient "your existing contact" when the contact list was empty.
> - **Distinct issues:** 2, agreed with the tester — (1) Agent keeps searching the inbox after it has found the recipient's address (SR-15908); (2) Agent calls the recipient "your existing contact" when the contact list was empty (SR-15909)
> - **Tester agrees:** yes

## Intent and expected outcome (confirmed by the tester)

**What the user was trying to do:** Have a follow-up email drafted to Shane thanking them for today's demo and proposing a call next Tuesday, see the draft first, and have it sent only after approval. Partway through, the user changed the day to Wednesday, then approved ("Approved Send it").

**A successful outcome:**

- Should happen: a draft is shown with nothing sent; the Wednesday edit is applied; exactly one email arrives, matching the approved draft.
- Should not happen: a send before explicit approval; the pre-edit (Tuesday) draft is sent; more than one send.

**Use case:** SR-13380 — Draft candidate outreach and prescreen invitations (confirmed by the tester), happy path.

**ClickUp ticket:** [SR-15779](https://app.clickup.com/t/1241hmha5g1) (test case TC-HL-001). Bugs raised: [SR-15908](https://app.clickup.com/t/1241hmha5u9) (Issue 1), [SR-15909](https://app.clickup.com/t/1241hmha5ua) (Issue 2).

The ticket's prompt names "[test inbox, TBD by QA]"; this run emailed Shane instead, which the tester confirmed is acceptable. The precondition "emailAgent is enabled for the test AI Employee" was not met on the first attempt (see stage 1).

| # | Ticket criterion | Result | Evidence | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | No emailAgent send call appears before approval | Met | Only send call at 18:17:13, after "Approved Send it"; platform "Review send" approval form also gated it (per session summary) | agree |
| 2 | Edit request is applied to the draft before sending | Met | Send call subject "Thanks for the demo today — call next Wednesday?" and body "…a call next Wednesday…" | agree |
| 3 | Exactly one email arrives, matching the approved draft | Met | One send call, text matches the approved draft; delivery confirmed by the tester in the inbox | Sent only once; nothing changed in the "Review send" form |
| 4 | Must not send before explicit approval | Met | As criterion 1 | agree |
| 5 | Must not send the original (pre-edit) draft | Met | Wednesday version sent | agree |
| 6 | Must not send more than once | Met | One send call; tester confirmed one email | agree |

## How the agent worked through it

Runs examined: 1 — QA, 12 October 2026 (four user turns in one session).

| # | Stage | What the agent did and got back | Time (UTC) | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | Understood the request (first attempt) | Loaded the email and calendar skills and requested the Gmail connector; the platform blocked it (`persona_disabled`). The user saw "To do this, please enable Gmail for this persona first." No draft. | 18:03:36–18:03:45 | Set-up issue: brand-new tenancy, first Gmail integration. Product note below |
| 2 | Looked for information | After the prompt was resent: contact lookup returned no contacts; inbox searches for "shane"/"demo"; search `from:shane@…` found messages, and the address was established by 18:13:40 | 18:12:49–18:13:40 | — |
| 3 | Looked for information (continued) | 20 further inbox searches for `from:shane` and "demo" across windows of 14, 7, 3, 1, 2, 4, 5, 6, 8 and 10 days. None of the results were used in the draft | 18:13:43–18:14:52 | agree (Issue 1) |
| 4 | Told the user | Showed the Tuesday draft ("nothing has been sent"), flagging the recipient as "your existing contact" and "next Tuesday" as 20 October; built an email-composer mini app (first build rejected by validation, then fixed) | 18:14:56–18:15:36 | agree (Issue 2) |
| 5 | Acted (edit) | On "Change that to Wednesday", updated subject and body to Wednesday; first reply rejected by the platform (`invalid_terminal_manifest`), retry succeeded; composer rebuilt | 18:16:13–18:16:40 | agree |
| 6 | Acted (send) | On "Approved Send it", one `emailAgent__send` call with the Wednesday subject and body | 18:17:06–18:17:13 | agree |
| 7 | After the send | Per session summary only: "Review send" approval form shown, owner approved, email sent (status "sent", message id `1a13155847a26e2f`) | by 18:17:38 | Sent once, form unchanged |

**Not recorded in the trace:** the send call's result; the "Review send" approval form and the owner's decision; the post-approval send step; whether the composer's own Send control was used. These appear only in the `summariseSession` text (logging gap: R3.1, R3.3).

## What went wrong

| # | Failure | Ticket criterion | First went wrong at | Suggested code | Seen or inferred | Severity | Issue | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | After finding the recipient's address at 18:13:40, the agent ran 20 more inbox searches over sweeping time windows that added nothing to the draft (~70 s; the drafting turn took ~2¾ min) | beyond the ticket | Stage 3 (Looking) | Performance · PF4 · AB6 · none | Seen — high | P3 | 1 | agree |
| 2 | Told the user the recipient was "your existing contact" although the contact lookup returned zero contacts; the match came from mail history | beyond the ticket | Stage 4 (Telling the user) | Agent behaviour · AB8 · none · none | Seen — high | P3 | 2 | agree |
| — | Refused to draft until Gmail was enabled for the persona | — | Stage 1 | not a defect | Seen | — | — | Set-up issue (new tenancy, first Gmail integration) |

**Product note (no code):** Gmail was connected in the tool library but was disabled for this AI Employee, so the first request was blocked until the user enabled it. The tester finds it frustrating to have to re-enable a connector already connected in the tool library before the first uses. Worth raising with product as a default-enablement question; it is not an agent behaviour defect.

## Clarifying-question check

| Issue | Question the agent should have asked | Asking would have helped? | Data point worth capturing | Scope | Would have prevented future asks? | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | n/a — no unchecked assumption; primary goal succeeded | n/a | n/a | n/a | n/a | — |
| 2 | n/a — the agent flagged its recipient match and asked; primary goal succeeded | n/a | n/a | n/a | n/a | — |

## Next steps

- [ ] Tighten the email skill / MRC instructions so that once the recipient's address is resolved and no source email is needed for the draft, the agent stops searching (the skill already says "Find mail with emailAgent.read, then stop"); discourage sweeping `newer_than` windows.
- [ ] Have the agent describe the recipient match by its actual source ("found in your mail history") rather than "existing contact" when the contact lookup is empty.
- [ ] Product: consider enabling connectors already connected in the tool library for the AI Employee by default, or offering a one-click enable in the blocked message.
- [ ] Logging: record the send result, the HITL approval form and the post-approval step as observations.

## Sources

Langfuse QA project (SR QA), 12 October 2026. Superreach session ID: `1c63085c-0f4c-441d-a085-ba2e173a0a7e` (persona "Trillion"). Langfuse session: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/sessions/1c63085c-0f4c-441d-a085-ba2e173a0a7e>. Trace links:

- Address found: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/ccfbc8dea9fc8f99c71a1f443898c0ac>
- First extra search: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/3f65a5be89ac9adc84180e65709b48fa>
- Last extra search: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/6390f54254f3b46e84fb4567492f5132>
- Draft reply ("existing contact"): <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/b858ead5696334de43b383258773f95b>
- Send call: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/878610eeb32f892aaefb993472ac78de>

## Finding record

```yaml
- source: human
  issue: 1
  defect_type: Performance
  primary_code: PF4
  secondary_codes: [AB6]
  tags: []
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: SR-13380
  expected_outcome: Draft shown before sending; Wednesday edit applied; exactly one email sent after approval
  tester_comment: agree
  clarifying_question:
    applies: no
  environment: QA
  session_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/sessions/1c63085c-0f4c-441d-a085-ba2e173a0a7e
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/3f65a5be89ac9adc84180e65709b48fa
  clickup_task: SR-15908
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
- source: human
  issue: 2
  defect_type: Agent behaviour
  primary_code: AB8
  secondary_codes: []
  tags: []
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: SR-13380
  expected_outcome: Draft shown before sending; Wednesday edit applied; exactly one email sent after approval
  tester_comment: agree
  clarifying_question:
    applies: no
  environment: QA
  session_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/sessions/1c63085c-0f4c-441d-a085-ba2e173a0a7e
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/b858ead5696334de43b383258773f95b
  clickup_task: SR-15909
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
```

## Draft ticket content

Paste into the bug tickets and attach this file. One block per issue agreed with the tester; Issue 1 is logged as SR-15908 and Issue 2 as SR-15909.

**Title:** Follow-up email run (SR-15779): excess inbox searching and an ungrounded "existing contact" claim

**Session:** `1c63085c-0f4c-441d-a085-ba2e173a0a7e` · QA · 12 October 2026, 18:03–18:17 UTC · runs examined: 1 · diagnosed with skill version 2026-10-13.4

**Use case:** SR-13380 — Draft candidate outreach and prescreen invitations

**What the user asked:** Draft a follow-up email to Shane thanking them for today's demo and proposing a call next Tuesday; show the draft first and only send once approved. The user then changed the day to Wednesday and approved.

**Expected outcome:** Draft shown with nothing sent; Wednesday edit applied; exactly one email sent after approval, matching the approved draft; no send before approval, no pre-edit send, no duplicate send.

**Ticket criteria:** 6 met, 0 not met, 0 can't tell (criterion 3 confirmed by the tester in the inbox).

### Issue 1: Agent keeps searching the inbox after it has found the recipient's address

- **What happened:** The agent had established Shane's address by 18:13:40, then ran 20 more inbox searches (18:13:43–18:14:52) for `from:shane` and "demo" across windows of 14, 7, 3, 1, 2, 4, 5, 6, 8 and 10 days. None of the results were used in the draft, adding about 70 seconds to a ~2¾-minute drafting turn.
- **Ticket criterion:** beyond the ticket
- **Expected:** Stop searching once the recipient is resolved and no source email is needed, and go straight to the draft.
- **Suggested classification:** Performance · primary PF4 · secondary AB6 · tags none
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen · confidence high · 18:13:43–18:14:52; first extra search <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/3f65a5be89ac9adc84180e65709b48fa>, last <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/6390f54254f3b46e84fb4567492f5132>
- **Tester's comment:** agree
- **Suggested next step:** Tighten the email skill / MRC instructions to stop searching once the recipient is resolved (the skill already says "Find mail with emailAgent.read, then stop"); discourage sweeping `newer_than` windows.

### Issue 2: Agent calls the recipient "your existing contact" when the contact list was empty

- **What happened:** At 18:14:56 the agent told the user "I matched "Shane" to Shane J. (…@superreach.com), your existing contact", but its contact lookup at 18:12:58 had returned zero contacts; the match actually came from mail history. Impact was low because the agent flagged the match and invited correction.
- **Ticket criterion:** beyond the ticket
- **Expected:** Describe the match by its real source (e.g. "found in your mail history").
- **Suggested classification:** Agent behaviour · primary AB8 · secondary none · tags none
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen · confidence high · 18:14:56, "…your existing contact." <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/b858ead5696334de43b383258773f95b>
- **Tester's comment:** agree
- **Suggested next step:** Instruct the agent to cite where a recipient match came from rather than asserting it is a contact.

**Not recorded in the trace:** the send result and the "Review send" approval step appear only in the session summary text.

**Product note (no code):** Gmail was connected in the tool library but disabled for this AI Employee, so the first request was blocked until it was enabled; the tester finds re-enabling an already-connected tool frustrating.

**Codes and severity are suggestions and stay suspected until the verifier confirms them.**
