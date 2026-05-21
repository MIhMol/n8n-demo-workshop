# Workflow 2: Support Ticket Triage with Gmail + Claude

A Gmail inbox is watched on a one-minute poll. Every new INBOX email is sent to Claude (Haiku 4.5) for classification: category (`support` / `bug` / `billing` / `general` / `spam`), urgency on a 1-5 scale (5 = production-down), a one-paragraph suggested reply in the team's voice, and a `needs_human` flag for cases the model can't draft confidently. Every email lands in an Airtable Tickets table with all seven fields. Urgent non-spam tickets (`urgency >= 4 AND category != "spam"`) also fire a Slack incoming-webhook alert to a `#client-alerts` channel.

**Who it's for:** small businesses where customer-facing email lives in a shared Gmail account or a personal inbox that doubles as support. Founders, customer-success teams of two or three, indie SaaS operators, agency owners handling client comms personally.

**Time-sink eliminated:** the morning and afternoon ritual of reading every inbound email, judging urgency, filing it, drafting a holding reply, and pinging the right person on Slack. Easily 45 minutes a day, more on a busy day.

**Dollar value captured:** the suggested-reply field alone often saves 5-10 minutes per ticket because the human only needs to edit, not draft from scratch. At 20 tickets a day and $50/hr support time, that's $80-160/day of reclaimed focused time.

## Files

- `workflow-2-support-triage.json` — importable n8n workflow JSON
- `workflow-2-canvas.png` — screenshot of the 7-node canvas after a successful execution
- `workflow-2-test-emails.md` — three paste-ready test email templates (urgent prod bug, casual billing question, promotional spam) with expected classifier outputs
- `sample-outputs/wf2-airtable-tickets-rows.png` — Airtable Tickets table after running the three test emails

## Sample output

The Airtable Tickets table after the three test emails. From Email column is intentionally hidden in this view for PII hygiene; the field is written by the workflow.

![WF2 Airtable Tickets rows](sample-outputs/wf2-airtable-tickets-rows.png)

## Architecture

7 nodes: Gmail Trigger polls INBOX on a 1-minute cadence → Build Anthropic Payload constructs Claude requests for each new email with multi-item batching → HTTP Request to `api.anthropic.com/v1/messages` → Parse Classification maps responses back to source emails via index-based pairing → Append to Tickets writes the row → Check Urgency gates on `urgency >= 4 AND category != "spam"` → Slack Urgent Alert (HTTP POST) fires on the urgent branch.

The classifier prompt categorizes into 5 buckets (support / bug / billing / general / spam) with a 1-5 urgency scale and a `needs_human` flag for cases the model can't draft confidently. Per-engagement tuning includes the threshold, categories, draft-reply tone, and any Gmail-side filtering to skip promotional traffic.

The public JSON in this repo has the node structure intact but redacts the Claude system prompt and the multi-item Code-node implementations. The fully working version is delivered as part of any tier — see the main [README](README.md) for hire info.
