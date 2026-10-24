# Resuming an interrupted audit

This is a client-side work log, not a ConvRadar write. Save `<host>-audit-state.json` beside the report when file tools are available. A ChatGPT skill cannot guarantee that an interrupted tool request completed or that a local file survived a new session. Therefore never claim exactly-once execution; avoid duplicate paid calls when status is known, and treat unknown status as unresolved.

## Before the first paid call

Record the host, audited locale, business model, `data_complete_to`, fixed `date_from` and `date_to`, report mode (`full`, `tracking_diagnostic`, or `partial`), next step, and a `calls` list. Do not refresh the date window on resume. Read the work log and the visible conversation tool history before making a call.

For each planned call, make a stable `call_key` from tool name plus canonical arguments, including date window and filters. Before sending the call, append a `pending` record with the key, tool, arguments and reason; save the file. Send only one paid ConvRadar call at a time. After its result arrives, replace `pending` with `completed`, save the relevant counts, dimensions, dates, caveats and tool result identifier if available, then advance `next_step`. Keep enough evidence to write the report without calling again. Store aggregate audit evidence only; omit names, emails, phone numbers and raw visitor records.

Example shape (illustrative keys and counts, not a real result):

```json
{
  "host": "example.com",
  "date_from": "2026-07-01",
  "date_to": "2026-09-28",
  "report_mode": "full",
  "next_step": "trust:confirm_conversions",
  "calls": [
    {"call_key": "cr_get_overview_metrics|date_from=2026-07-01&date_to=2026-09-28", "status": "completed", "result": {"sessions": 1000, "conversions": 20}},
    {"call_key": "cr_query_metrics|dimensions=event_name&date_from=2026-07-01&date_to=2026-09-28", "status": "pending"}
  ]
}
```

## On the next turn

1. Open the existing report, work log and visible tool history. Reconstruct missing `completed` records from visible successful tool results before planning another call. Keep the original host and date window.
2. Skip every `completed` call_key and use its saved result. If the report file already exists, continue editing it instead of rebuilding or replacing finished sections.
3. For a `pending` call, inspect the visible tool history. If it has a result, save it as `completed`. If execution cannot be determined, mark it `unknown`, do not automatically repeat it, and continue any independent work. Where the unknown result is required, deliver an explicitly partial report and name the missing check; ask before a potentially duplicate paid rerun.
4. If neither file nor visible tool history is available, recover only facts that remain in the chat. Do not invent missing values or replay the full call sequence. State that the earlier results are unavailable and request the prior report or an explicit fresh audit.
5. Count `completed`, `pending` and `unknown` calls against the total call ceiling. A weekly quota error changes the report mode to `partial`; it is not an app interruption and retrying the same call will not restore quota.

After each logical section, save the draft HTML and update `next_step`. A short progress note in chat may repeat the next step and work-log filename, but it is not the sole checkpoint.
