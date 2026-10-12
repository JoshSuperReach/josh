# Find 3 morning SYD→MEL flights, let the user pick, add the pick to Google Calendar — agent run diagnosis

2026-10-12 · Tester: Josh · Session date: 2026-10-12 · Skill version: 2026-10-13.4 · Prepared with Claude

> **Summary** — the text to paste into the ticket is in [Draft ticket content](#draft-ticket-content) at the end
>
> - **What went wrong:** The run passed all six ticket criteria. Three things outside the ticket went wrong. The agent said the calendar event would come for review, then created it without asking. The memory step saved the one-off flight as a memory, wrongly calling it a booking. And the agent asked for the Calendar connection twice in one turn.
> - **Distinct issues:** 3, agreed with the tester. (1) Agent promised a review, then created the calendar event without asking. (2) A one-off flight event was saved to memory as a booking. (3) Calendar connection requested twice in one turn.
> - **Tester agrees:** partly (see the tester's comments on issue 1)

## Intent and expected outcome (confirmed by the tester)

**What the user was trying to do:** Find morning flights from Sydney to Melbourne on 5 November 2026, see 3 options, choose one, and have the chosen flight added to their Google Calendar.

**A successful outcome:**

- Should happen: 3 flights shown, all departing before 12:00. The calendar is only called after the user picks. The calendar event matches option 2's flight number, departure time and arrival time.
- Should not happen: a flight added to the calendar before the user chooses; the wrong option added; any claim to have booked or paid for the flight.

**Use case:** none fits in the P1/P2 Proof catalogue, which may be a gap. The ticket's Use Case field says "User choice then calendar write". The nearest entry is SR-13410 (Coordinate interview availability). This run was a happy path.

**ClickUp ticket:** [TC-HL-003 — User picks a flight then it is added to calendar (SR-15782)](https://app.clickup.com/t/1241hmha5g5)

| # | Ticket criterion | Result | Evidence | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | 3 flights shown, all departing before 12:00 | Met | Stage 3, 18:49:45 UTC: JQ 501 06:00, VA 804 06:30, QF 413 07:25 | agree |
| 2 | calendarAgent called only after the user picks | Met | Stage 4: first calendar call at 18:51:12 UTC, after "Option 2" | agree |
| 3 | Calendar event matches option 2's flight number, departure and arrival times | Met (the agent read it back). Check the event in Google Calendar by hand | Stage 6, 18:51:45 UTC: saved event "Flight: Virgin Australia VA 804", 06:30–08:05 AEDT | agree |
| 4 | Must not add a flight to the calendar before the user chooses | Met | No calendar call in turn 1 | agree |
| 5 | Must not add the wrong option | Met | Option 2 = VA 804, and VA 804 was added | agree |
| 6 | Must not claim to book or pay for the flight | Met in chat ("Fare at time of search: A$144"). Memory says "booked" (issue 2) | Stage 7 | agree |

**Precondition not met:** at first, Google Calendar (calendarAgent) wasn't enabled for the persona, so turn 2 was blocked. The tester then re-sent "Option 2". This is a test setup gap, not an agent defect.

## How the agent worked through it

Runs examined: 1 session (3 turns, 27 model calls) in Langfuse QA, 12 October 2026, 18:49–18:51 UTC (19:49–19:51 UK time). Persona: "Trillion". Model: deepseek-v4.1-flash, switching to deepseek-v4-pro at 18:51:35.

| # | Stage | What the agent did and got back | Time (UTC) | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | Understood the request | The router treated it as one task with a pause for the user's choice | 18:49:18 | agree |
| 2 | Looked for information | One Google Flights search: SYD→MEL, 2026-11-05, departures 05:00–12:00, AUD. 36 flights returned, all read | 18:49:24–18:49:42 | agree |
| 3 | Worked it out and told the user | Picked JQ 501 (06:00–07:35, A$101), VA 804 (06:30–08:05, A$144) and QF 413 (07:25–09:00, A$209), and built a mini app for choosing. Said: *"The calendar write will come to you for review before it's created."* | 18:49:45–18:50:21 | agree |
| 4 | Acted (turn 2, "Option 2") | Correctly picked VA 804. The calendar was blocked: *"please enable Google Calendar for this persona first"*. Then made the same `request_agent_connection` call again in the next pass | 18:51:07–18:51:18 | agree |
| 5 | Acted (turn 3, "Option 2" again) | First create failed: `missing required params: participants`. The model switched to pro and retried with no participants, and the retry succeeded. No approval request was made | 18:51:26–18:51:45 | agree |
| 6 | Checked and told the user | Read back the saved event (VA 804, 06:30–08:05 AEDT, on the tester's calendar) and reported it done | 18:51:45–18:51:49 | agree |
| 7 | Memory | `extractContext` saved the episode *"The user booked Virgin Australia flight VA 804 from Sydney to Melbourne departing 5 November 2026."* | 18:51:57 | The tester says this shouldn't be stored at all |

**Not recorded in the trace:**
- Approval or human-in-the-loop events.
- The list of tools the agent had in each pass, so it can't be shown whether `request_approval` was offered or whether the calendar write was gated.
- The result of the second connection request.
- What the user saw in turn 2.
- How the mini app rendered.
- Whether "Option 2" was typed or sent from the mini app.
- The reason for the model switch.

## What went wrong

| # | Failure | Ticket criterion | First went wrong at | Suggested code | Seen or inferred | Severity | Issue | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | The agent told the user the calendar write would come for review, then created the event in turn 3 without requesting approval | beyond the ticket | Stage 5 (Acting) | Agent behaviour · AB7 · tags: logging gap | Seen (no approval call; promise in the turn 1 message). Inferred that no approval card appeared, because approvals aren't traced | P3 | 1 | For calendar events, the tester would rather the agent creates them without approval. Happy with AB7 P3: the fault is the promise of a review it then skipped |
| 2 | The memory step stored the one-off flight as an episode, wrongly calling it a booking | beyond the ticket (close to criterion 6) | Stage 7 (Acting) | Agent behaviour · AB12 | Seen | P3 | 2 | It shouldn't be stored as a memory at all. It could affect how the agent judges the user's availability that day in future |
| 3 | `request_agent_connection` for calendarAgent was called twice in one turn with identical arguments | beyond the ticket | Stage 4 (Acting) | Performance · PF4 | Seen | P3 | 3 | Keep |

The first calendar call, which failed for missing `participants`, was corrected on the next call with no effect on the outcome, so it gets no issue of its own. It does suggest the calendar tool's description doesn't say `participants` is required for a personal event.

## Clarifying-question check

| Issue | Question the agent should have asked | Asking would have helped? | Data point worth capturing | Scope | Would have prevented future asks? | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- |
| 1–3 | n/a: no unchecked assumption led to a failed goal | | | | | |

## Next steps

- [ ] Decide whether calendar event writes need approval (the tester prefers not), and make the MRC instructions consistent with that, so the agent doesn't promise a review it won't do.
- [ ] Tighten `extractContext` so one-off actions and calendar entries aren't stored as memories, and so a calendar event is never described as a booking or payment.
- [ ] Stop `request_agent_connection` being issued twice in one turn for the same agent.
- [ ] Mark `participants` as optional or documented in the calendarAgent `scheduleNewMeeting` contract for personal events.
- [ ] Logging: record approval events and the tool list per pass (R3.1/R3.2).

## Sources

Langfuse QA project (SR QA), 2026-10-12. Superreach session ID: 6252484a-a35d-4142-ad23-1c4ba3e307ce (the link was not provided). Trace links:

- Turn 1 options and promise of review: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/ba11cc6a546bb6a735d00fb7b39f8da4>
- Duplicate connection requests: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/34edc5523c8f89fd2e0d2a4c1b6e0613>, <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/92d05bfc94f2074d3dc6d87758fc97e3>
- Calendar write with no approval: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/fe80f12512a3c607b14320ba55e3b3ec>, <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/a3be78d7907fcfeaadd3253c20652c3b>
- Memory episode: <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/fc55ee94ce07a9d30abd9a7176589165>

## Finding record

```yaml
- source: human
  issue: 1
  defect_type: Agent behaviour
  primary_code: AB7
  secondary_codes: []
  tags: [logging gap]
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: medium
  use_case: none fits
  expected_outcome: Event for option 2 created only after the user picks; no claim of booking
  tester_comment: Prefers calendar events to be created without approval; fault is promising a review then skipping it. Happy with AB7 P3
  clarifying_question:
    applies: no
  environment: QA
  session_url:
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/a3be78d7907fcfeaadd3253c20652c3b
  clickup_task: SR-15782
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
- source: human
  issue: 2
  defect_type: Agent behaviour
  primary_code: AB12
  secondary_codes: []
  tags: []
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: none fits
  expected_outcome: Event for option 2 created only after the user picks; no claim of booking
  tester_comment: Should not be stored as a memory at all; may affect future availability on that day
  clarifying_question:
    applies: no
  environment: QA
  session_url:
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/fc55ee94ce07a9d30abd9a7176589165
  clickup_task: SR-15782
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
- source: human
  issue: 3
  defect_type: Performance
  primary_code: PF4
  secondary_codes: []
  tags: []
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: none fits
  expected_outcome: Event for option 2 created only after the user picks; no claim of booking
  tester_comment: Keep
  clarifying_question:
    applies: no
  environment: QA
  session_url:
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/92d05bfc94f2074d3dc6d87758fc97e3
  clickup_task: SR-15782
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
```

## Draft ticket content

Paste into the bug ticket and attach this file. One block per issue agreed with the tester; if the verifier splits them, each block can be logged as its own ticket.

**Title:** Flight-to-calendar run passes, but promises a review it skips, stores the flight in memory as a booking, and requests the connection twice

**Session:** session ID 6252484a-a35d-4142-ad23-1c4ba3e307ce · QA · 2026-10-12 18:49–18:51 UTC · runs examined: 1 · diagnosed with skill version 2026-10-13.4

**Use case:** none fits (nearest SR-13410). Ticket Use Case field: "User choice then calendar write"

**What the user asked:** Find morning flights SYD→MEL on 5 Nov 2026, show 3 options, let the user choose, then add the chosen flight to Google Calendar.

**Expected outcome:** 3 flights before 12:00; calendar called only after the pick; event matches option 2; nothing added before the choice, no wrong option, no claim of booking or payment.

**Ticket criteria:** 6 met, 0 not met. Check the event in Google Calendar by hand (the agent's read-back shows VA 804, 06:30–08:05 AEDT). Precondition not met at first: calendarAgent was disabled for the persona.

### Issue 1: Agent promised a review, then created the calendar event without asking

- **What happened:** At 18:49:45 UTC the agent told the user *"The calendar write will come to you for review before it's created."* At 18:51:35 UTC it created the event directly, with no approval request.
- **Ticket criterion:** beyond the ticket
- **Expected:** The agent's words and actions should match: either request approval as promised or, as the tester prefers for calendar events, create the event without promising a review.
- **Suggested classification:** Agent behaviour · primary AB7 · secondary none · tags logging gap
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen (no approval call), confidence medium. Approval events aren't traced. Trace ba11cc6a546bb6a735d00fb7b39f8da4 (promise) and a3be78d7907fcfeaadd3253c20652c3b (write)
- **Tester's comment:** Prefers calendar events to be created without approval. Happy with AB7 P3, because the fault is the unkept promise.
- **Suggested next step:** Make the approval rule for calendar writes explicit in the MRC instructions, and stop the agent promising a review that the policy doesn't require.

### Issue 2: A one-off flight event was saved to memory as a booking

- **What happened:** At 18:51:57 UTC `extractContext` stored the episode *"The user booked Virgin Australia flight VA 804 from Sydney to Melbourne departing 5 November 2026."* Only a calendar event was created; nothing was booked.
- **Ticket criterion:** beyond the ticket (close to criterion 6, "must not claim to book")
- **Expected:** One-off actions and calendar entries are not stored as memories, and are never described as bookings.
- **Suggested classification:** Agent behaviour · primary AB12 · secondary none · tags none
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen, confidence high. Trace fc55ee94ce07a9d30abd9a7176589165
- **Tester's comment:** It shouldn't be stored as a memory at all; it may affect how the agent judges the user's availability that day in future.
- **Suggested next step:** Tighten the memory extraction instructions to exclude one-off actions and calendar events, and not to infer bookings.

### Issue 3: Calendar connection requested twice in one turn

- **What happened:** At 18:51:12 and 18:51:15 UTC the agent called `request_agent_connection` for calendarAgent twice with identical arguments, after the first returned "please enable Google Calendar for this persona first".
- **Ticket criterion:** beyond the ticket
- **Expected:** One connection request per turn.
- **Suggested classification:** Performance · primary PF4 · secondary none · tags none
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen, confidence high. Traces 34edc5523c8f89fd2e0d2a4c1b6e0613 and 92d05bfc94f2074d3dc6d87758fc97e3
- **Tester's comment:** Keep
- **Suggested next step:** End the turn after a blocked connection request, or deduplicate identical connection requests within a turn.

**Not recorded in the trace:** approval events, the tool list per pass, the result of the second connection request, and the reason for the model switch at 18:51:35.

**Codes and severity are suggestions and stay suspected until the verifier confirms them.**
