# Pull company profile for Canva (coresignalAgent) — agent run diagnosis

2026-10-12 · Tester: Josh · Session date: 2026-10-12 · Skill version: 2026-10-13.4 · Prepared with Claude

> **Summary** — the text to paste into the ticket is in [Draft ticket content](#draft-ticket-content) at the end
>
> - **What went wrong:** The agent opened the top Coresignal company-search result without checking its name. That record was "Zeetings (acquired by Canva)", so the agent needed a second lookup by domain to reach Canva. That made 3 Coresignal calls where the test expects 1. The final answer was correct and complete.
> - **Distinct issues:** 1. Agent opens the top company search result without checking its name, so pulls an acquired subsidiary (Zeetings) before the real Canva record
> - **Tester agrees:** yes

## Intent and expected outcome (confirmed by the tester)

**What the user was trying to do:** Get a company profile for Canva covering employee headcount, headquarters, industry, founded year and LinkedIn URL.

**A successful outcome:**

- Should happen: exactly one coresignalAgent call for Canva; all 5 fields in the response; values match the connector output.
- Should not happen: a different company with a similar name returned; linkedinAgent or googleSearchAgent called; missing fields filled with guesses.

**Use case:** none recorded (the tester chose none; the ticket's Use Case field says "Company enrichment", which has no entry in the P1/P2 Proof catalogue)

**ClickUp ticket:** [SR-15805 — TC-SC-004 — Pull company profile for Canva (coresignalAgent)](https://app.clickup.com/t/1241hmha5gw)

| # | Ticket criterion | Result | Evidence | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | Trace shows exactly one coresignalAgent call for Canva | Not met | Stages 3–5, 20:57:40–20:57:56: three Coresignal calls — `searchCompanies`, `collectCompanyById` (id 1123830, Zeetings), `enrichCompany` (`canva.com`) | Agree (strict reading: one Coresignal tool call) |
| 2 | Response includes all 5 fields: headcount, HQ, industry, founded year, LinkedIn URL | Met | Stage 6, 20:58:07: all five fields given | Agree |
| 3 | Values match the connector output | Met | Stage 5 → 6: 20,413; Surry Hills, New South Wales, Australia; Software Development; 2012; `https://www.linkedin.com/company/canva`, all identical to the `enrichCompany` record 1954586 | Agree |
| 4 | Must not return a different company with a similar name | Met | Stage 6: Zeetings record "discarded"; none of its values reached the answer | Agree |
| 5 | Must not call linkedinAgent or googleSearchAgent | Met | No such calls in the trace | Agree |
| 6 | Must not fill missing fields with guesses | Met | No field was missing; the conflict between headcount (20,413) and size band (1,001–5,000) was flagged, not smoothed over | Agree |

What the user typed matches the ticket's prompt word for word.

## How the agent worked through it

Runs examined: 1 — QA, 12 October 2026, 20:57:32–20:58:11 UK time (single turn, about 39 seconds), model `deepseek/deepseek-v4.1-flash` throughout.

| # | Stage | What the agent did and got back | Time | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | Understood the request | The router classified it as a one-off information lookup (MRC, "Canva company profile lookup") | 20:57:32 | Agree |
| 2 | Loaded the Coresignal skill | `load_skills` → `coresignal@2.6.0` loaded | 20:57:38 | Agree |
| 3 | Looked for the company | `coresignalAgent__searchCompanies` with `match_phrase company_name "Canva"`, 5 per page → 5 bare IDs (1123830, 83860789, 93682080, 95534968, 96139569), read at 20:57:46 | 20:57:40 | Agree |
| 4 | Opened the top result without checking it | `coresignalAgent__collectCompanyById` with `company_id 1123830` → "Zeetings (acquired by Canva)", 1–10 employees, founded 2013, LinkedIn `/company/zeetings`, website domain `canva.com`; inspected and read the fields at 20:57:51–20:57:56 | 20:57:48 | Agree |
| 5 | Switched to a lookup by domain | `coresignalAgent__enrichCompany` with `website canva.com` → record 1954586 "Canva", legal name Canva Pty Ltd, with all five fields; fields read at 20:58:03 | 20:57:56 | Agree |
| 6 | Told the user | `finalize_mrc_result` (outcome `completed`) with the five fields, a note on the stale size band, and a note that the Zeetings match was discarded | 20:58:07 | Agree |
| 7 | Memory | `extractContext` / `summariseSession` stored public Canva facts only; no personal data | 20:58:12 | — |

**Not recorded in the trace:** nothing material. As usual, tool calls are visible only inside generation inputs and outputs (no separate tool observations).

## What went wrong

| # | Failure | Ticket criterion | First went wrong at | Suggested code | Seen or inferred | Severity | Issue | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Opened the top company search result (an acquired subsidiary, Zeetings) without checking its name, then needed a second lookup by domain to reach Canva | 1 | Stage 4 (Looking) | Agent behaviour · AB3 · PF4 · no tags | Seen — high confidence (pass_4 collect on id 1123830 returns "Zeetings (acquired by Canva)"; pass_7 `enrichCompany` follows) | P3 | 1 | Agree |

Note: the defect guide says a brief wrong turn that is reversed with no effect on the answer needs no ticket on its own merits. This one is logged because it breaks ticket criterion 1, and the Zeetings record is listed in the answer's evidence (labelled "discarded"). Even a good run makes at least two Coresignal calls (search, then collect) unless it goes straight to `enrichCompany` by domain, which the user didn't supply. Criterion 1 may therefore need rewording.

## Clarifying-question check

n/a. The primary goal was met, and no unchecked assumption led to it failing.

## Next steps

- [ ] Coresignal skill: after collecting a search result, check the record's `company_name` against the requested company before using it. Where a company is well known, or its domain becomes known, prefer `enrichCompany` by domain.
- [ ] Coresignal connector: look into why an exact-phrase `company_name` search for "Canva" ranks "Zeetings (acquired by Canva)" first. Consider returning names alongside IDs so the agent can choose without collecting each record.
- [ ] Test case TC-SC-004: decide whether criterion 1 means one Coresignal *tool call* or one *lookup* (search + collect), and reword it if needed.

## Sources

Langfuse QA project (SR QA), 12 October 2026. Superreach session: <https://qa.superreach.ai/superreach-new/app/chat/persona/6accdbd8f59326fbc9bb3fa7/session/46572d51-1090-48fd-bd99-1aa1da7802ec>. Trace links: wrong record opened <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/35c6557d1641ea69de9a5ffa95523ebb>; domain lookup <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/d4c7260d804de971a9887ffadfe8abec>; final answer <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/8c2590384ee3dd65ad3a88dfc1ffd044>

## Finding record

```yaml
- source: human
  issue: 1
  defect_type: Agent behaviour
  primary_code: AB3
  secondary_codes: [PF4]
  tags: []
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: none fits
  expected_outcome: Exactly one coresignalAgent call for Canva; all 5 fields returned, matching connector output; no similarly named company, no linkedin/googleSearch agents, no guessed fields
  tester_comment: Agrees with the reading of criterion 1 (one Coresignal tool call)
  clarifying_question:
    applies: no
    question:
    would_have_helped:
    data_point:
    scope:
    prevents_future_asks:
  environment: QA
  session_url: https://qa.superreach.ai/superreach-new/app/chat/persona/6accdbd8f59326fbc9bb3fa7/session/46572d51-1090-48fd-bd99-1aa1da7802ec
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/35c6557d1641ea69de9a5ffa95523ebb
  clickup_task: SR-15805
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
```

## Draft ticket content

Paste into the bug ticket and attach this file.

**Title:** Agent opens the top company search result without checking its name, so pulls an acquired subsidiary (Zeetings) before the real Canva record

**Session:** <https://qa.superreach.ai/superreach-new/app/chat/persona/6accdbd8f59326fbc9bb3fa7/session/46572d51-1090-48fd-bd99-1aa1da7802ec> · QA · 12 Oct 2026, 20:57:32–20:58:11 UK time · runs examined: 1 · diagnosed with skill version 2026-10-13.4

**Use case:** none fits

**What the user asked:** Get the company profile for Canva: employee headcount, headquarters, industry, founded year and LinkedIn URL.

**Expected outcome:** Exactly one coresignalAgent call for Canva; all 5 fields returned and matching the connector output. Must not return a similarly named company, call linkedinAgent or googleSearchAgent, or guess missing fields.

**Ticket criteria:** 5 met, 1 not met (criterion 1), 0 can't tell from the trace.

### Issue 1: Agent opens the top company search result without checking its name, so pulls an acquired subsidiary (Zeetings) before the real Canva record

- **What happened:** At 20:57:40 `searchCompanies` for "Canva" returned 5 bare IDs. At 20:57:48 the agent collected the first, id 1123830, which was "Zeetings (acquired by Canva)". It recovered at 20:57:56 with `enrichCompany` by domain `canva.com` (record 1954586, Canva Pty Ltd). That made 3 Coresignal calls where the test expects 1. The final answer was correct, and it disclosed the discarded Zeetings match.
- **Ticket criterion:** 1
- **Expected:** One Coresignal lookup that lands on Canva directly, or a check of the collected record's name before relying on it.
- **Suggested classification:** Agent behaviour · primary AB3 · secondary PF4 · tags none
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen · confidence high · 20:57:48, `collectCompanyById {"company_id": "1123830"}` → `"company_name": "Zeetings (acquired by Canva)"` — <https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/35c6557d1641ea69de9a5ffa95523ebb>
- **Tester's comment:** Agrees with the reading of criterion 1 (one Coresignal tool call)
- **Suggested next step:** In the Coresignal skill, check a collected record's `company_name` against the request before using it, and prefer `enrichCompany` by domain where the domain is known. Look into the connector's search ranking, which put the acquired subsidiary first. Clarify whether criterion 1 means one tool call or one lookup.

**Codes and severity are suggestions and stay suspected until the verifier confirms them.**
