# Workflow 1: Lead Intake with Claude Classification

A webhook receives inbound contact-form submissions. Claude (Haiku 4.5) classifies each one in a single API call: is it a real lead or spam, urgency on a 1-10 scale, intent category (pricing, support, demo, partnership, spam, other), and a one-sentence next-action suggestion. Every classified record lands in an Airtable CRM table. Urgent non-spam leads (`urgency_score >= 8 AND is_spam == false`) also fire a Slack incoming-webhook alert so a human sees them within seconds.

**Who it's for:** small B2B service businesses where contact forms drop into a Gmail inbox nobody monitors in real time. Founders, two-person agencies, freelance consultants, indie SaaS teams.

**Time-sink eliminated:** the daily 20-minute "scan the inbox, decide what's urgent, copy into CRM, notify the team" ritual. Roughly 2 hours a week per inbox.

**Dollar value captured:** at a 5% conversion on inbound leads worth $2k each, missing one urgent lead per month costs $100 in expected value. Most teams miss more.

## Files

- `workflow-1-lead-intake.json` — importable n8n workflow JSON
- `workflow-1-test-payloads.json` — three sample inbound payloads (urgent lead, casual lead, SEO spam) with expected classifier output
- `workflow-1-canvas.png` — screenshot of the 7-node canvas after a successful execution
- `sample-outputs/wf1-airtable-leads-rows.png` — Airtable Leads table after running the test payloads
- `sample-outputs/wf1-slack-alert.png` — Slack `#client-alerts` message fired by the urgency gate

## Sample output

The Airtable Leads table after running the three test payloads plus an extra urgent test:

![WF1 Airtable Leads rows](sample-outputs/wf1-airtable-leads-rows.png)

The Slack alert that fires when `urgency_score >= 8 AND is_spam == false`:

![WF1 Slack alert in #client-alerts](sample-outputs/wf1-slack-alert.png)

## Architecture

7 nodes: Webhook receives the form payload → Build Anthropic Payload constructs the Claude request with a tuned classifier prompt → HTTP Request POSTs to `api.anthropic.com/v1/messages` → Parse Classification cleans and maps the JSON response → Append to Airtable writes the lead → Check Urgency gates on `urgency_score >= 8 AND is_spam == false` → Slack Urgent Alert (HTTP POST to incoming-webhook URL) fires only on the urgent branch.

The classifier prompt is tuned for B2B-service inbound on a 1-10 urgency scale with 6 intent categories (pricing, support, demo, partnership, spam, other). Urgency threshold and category list can be re-tuned in minutes per engagement.

The public JSON in this repo has the node structure intact but redacts the Claude system prompt and the multi-item Code-node implementations. The fully working version is delivered as part of any tier — see the main [README](README.md) for hire info.
