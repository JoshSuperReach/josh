# Get YouTube video details (title, description, views, likes, channel) — agent run diagnosis

2026-10-12 · Tester: Josh · Session date: 2026-10-12 · Skill version: 2026-10-13.4 · Prepared with Claude

> **Summary** — the text to paste into the ticket is in [Draft ticket content](#draft-ticket-content) at the end
>
> - **What went wrong:** The run passes the test case. The agent fetched the right video once, and every figure matched the video page. However, it told the user that YouTube's result had cut the description short. The cut was actually made by Superreach's own tool for reading the saved result.
> - **Distinct issues:** 1 — Agent tells the user YouTube cut the description short, when Superreach's own read tool did
> - **Tester agrees:** yes

## Intent and expected outcome (confirmed by the tester)

**What the user was trying to do:** Get the title, description, view count, like count and channel name for YouTube video `AI_nd0CGoEY` (https://youtu.be/AI_nd0CGoEY).

**A successful outcome:**

- Should happen: exactly one youtubeVideoAgent call, using the exact ID `AI_nd0CGoEY`
- Should happen: title and channel match the video page
- Should happen: views and likes match the page to the precision shown there
- Should happen: a description is returned, and its opening matches the page (a cut-short description is acceptable)
- Should not happen: a youtubeSearchAgent call before or instead of youtubeVideoAgent
- Should not happen: details for a different video
- Should not happen: figures that aren't on the page
- Should not happen: an answer from general knowledge

**Use case:** none fits. The P1/P2 Proof catalogue has no YouTube entry, which may be a gap in the catalogue.

**ClickUp ticket:** none

**Test result against the criteria:** pass. The tester confirmed that the title, the channel (Curious with Ezra Chapman), 193 views and 5 likes match the video page on the day of the run.

## How the agent worked through it

Runs examined: 1 — Langfuse QA, 2026-10-12.

| # | Stage | What the agent did and got back | Time | Tester's comment |
| --- | --- | --- | --- | --- |
| 1 | Understood the request | The router treated it as a one-off lookup (`MRC`) | 14:26:48 | agree |
| 2 | Prepared | Loaded the YouTube Video skill. The skill says to fetch the video once, using the exact ID | 14:26:50 | agree |
| 3 | Looked for information | Made one `youtubeVideoAgent__youTubeVideo` call with `v: "AI_nd0CGoEY"`. It succeeded (`ok: true`) and the result was saved in full (`bytesComplete: true`, `complete: true`, 38.3 KB) | 14:26:52 | agree |
| 4 | Worked it out | Read the saved result four times with `explore_source`:<br>- Title, channel, views ("193 views") and likes ("5") came back in full.<br>- The description came back cut off mid-sentence at "…bigger questions about leadership, education".<br>- The tool marked that read `complete: false` and listed `/result/youTubeVideo/description/content` under `omittedPaths`. It offered no way to read the rest | 14:26:58–14:27:04 | agree; a cut-short description is acceptable |
| 5 | Told the user | Gave the title, the channel (3.61K subscribers), 193 views, 5 likes, the published date (7 Oct 2026), the cut-short description and the thumbnail. It added: "the stored description text is truncated at that point in the provider result; the full text continues beyond what was returned" | 14:27:06 | agree that the cause given is wrong |

No youtubeSearchAgent call was made, and nothing came from general knowledge. The run took about 20 seconds over 7 passes.

**Not recorded in the trace:**

- Tool calls appear only inside the model calls, not as steps of their own (R3.1).
- The result of the final `finalize_mrc_result` call isn't recorded (R3.1).
- Runtime playbooks are recorded as `user` messages (R3.3).

None of these blocked the diagnosis.

## What went wrong

| # | Failure | First went wrong at | Suggested code | Seen or inferred | Severity | Issue | Tester's comment |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | The agent told the user that the provider's (YouTube's) result had cut the description short. In fact the provider's result was saved complete; Superreach's `explore_source` read clipped it. The tool reported only that its response was incomplete, without a reason, and the agent filled in a cause its evidence doesn't support | Stage 5 (telling the user) | Agent behaviour · AB8 · no secondary codes · tag: logging gap | Seen, high confidence: the save receipt shows `bytesComplete: true` and the read shows `complete: false` | P3 | 1 | Agree. Cutting the description short is acceptable; blaming the wrong thing is not |

Claude's original proposal also raised:

- **The cut itself**, as AB10 (stopped short). The tester accepted the cut-short description, so it was dropped.
- **Memory steps saving the video's host and guest as people "related to" the user**, as AB12. The tester dropped this because the agent had no way to know whether that was wanted.

## Clarifying-question check

n/a. The agent made no unchecked assumption, and the user's primary goal was met.

## Next steps

- [ ] When the agent can't see why a value is cut short, it should say plainly that the text was cut short, without naming a cause.
- [ ] Have `explore_source` say when, and why, it clips a long string (for example a size limit), and offer a way to read the rest. Then the agent doesn't have to guess.

## Sources

- Langfuse QA project, 2026-10-12.
- Superreach session ID: `a2249110-6339-49b5-b805-f2909b42c3d2`. The link was not provided.
- Trace links:
  - Clipped description read: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/ff9c2a25a84d0d2d41e2542cf7da843c
  - Final message: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/766f44c4246b5ac0acdbb7fd1af23b24

## Finding record

```yaml
- source: human
  issue: 1
  defect_type: Agent behaviour
  primary_code: AB8
  secondary_codes: []
  tags: [logging gap]
  severity: P3
  consistency: Once
  evidence_status: suspected
  confidence: high
  use_case: none fits
  expected_outcome: One youtubeVideoAgent call with the exact ID; title, channel, views and likes match the page; description opening matches; no search agent, no general knowledge
  tester_comment: A cut-short description is acceptable, but the agent must not blame the wrong thing
  clarifying_question:
    applies: no
    question:
    would_have_helped:
    data_point:
    scope:
    prevents_future_asks:
  environment: qa
  session_url:
  trace_url: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/766f44c4246b5ac0acdbb7fd1af23b24
  clickup_task:
  session_date: 2026-10-12
  skill_version: 2026-10-13.4
```

## Draft ticket content

Paste this into the bug ticket and attach this file.

**Title:** Agent tells the user YouTube cut the description short, when Superreach's own read tool did

**Session:** `a2249110-6339-49b5-b805-f2909b42c3d2` · QA · 2026-10-12 14:26:48–14:27:09 UK time · runs examined: 1 · diagnosed with skill version 2026-10-13.4

**Use case:** none fits (no YouTube entry in P1/P2 Proof)

**What the user asked:** The title, description, view count, like count and channel name for YouTube video `AI_nd0CGoEY`.

**Expected outcome:**

- Should happen:
  - One youtubeVideoAgent call, using the exact ID.
  - Title, channel, views and likes match the video page.
  - The opening of the description matches the page.
- Should not happen:
  - A youtubeSearchAgent call.
  - Details for a different video.
  - Figures that aren't on the page.
  - An answer from general knowledge.

The test case passed on all of these; the tester confirmed the figures.

### Issue 1: Agent tells the user YouTube cut the description short, when Superreach's own read tool did

- **What happened:**
  - At 14:27:06 the agent told the user: "the stored description text is truncated at that point in the provider result".
  - The YouTube result was saved complete at 14:26:52 (`bytesComplete: true`, 38.3 KB).
  - The cut was made by Superreach's `explore_source` read at 14:27:04 (`complete: false`, description content in `omittedPaths`).
- **Expected:** Say that the description shown is cut short, without blaming the provider. A cut-short description is acceptable.
- **Suggested classification:** Agent behaviour · primary AB8 · secondary none · tags: logging gap
- **Suggested severity:** P3 · **Consistency:** Once
- **Evidence:** seen · confidence high
  - Save receipt (14:26:52): `"bytesComplete":true,"complete":true`
  - Read result (14:27:04): `"complete":false,"omittedPaths":["/result/youTubeVideo/description/content"]`
  - Trace: https://cloud.langfuse.com/project/cmuvnpwpv02xbad0cc5a61nj4/traces/766f44c4246b5ac0acdbb7fd1af23b24
- **Tester's comment:** A cut-short description is acceptable, but the agent must not blame the wrong thing.
- **Suggested next step:**
  - When the agent can't see why a value is cut short, it should state the cut neutrally.
  - `explore_source` should say when, and why, it clips a long string, and offer a way to read the rest.

**Not recorded in the trace:** tool calls appear only inside the model calls, and the result of the final `finalize_mrc_result` call isn't recorded.

**Codes and severity are suggestions and stay suspected until the verifier confirms them.**
