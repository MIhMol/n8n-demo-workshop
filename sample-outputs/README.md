# Sample outputs

Real screenshots from each workflow running end-to-end on the same local n8n install used to build this portfolio. All test data is synthetic; no real customer data, no real PII.

## Files

| File | What it shows | Workflow |
|---|---|---|
| `wf1-airtable-leads-rows.png` | The Airtable Leads table after running workflow 1's test payloads. Four rows: two genuine leads (Sarah Chen support / Mike Patel pricing), one SEO spam (Marketing Pro Tools), one extra urgent test (Sarah Chen "Slack swap test" from the Discord-to-Slack migration). Note the urgency-star spread reflecting Claude's classification (9-10 stars for urgent prod issues, 2 stars for casual inquiry, 1 star for spam). | 1 |
| `wf1-slack-alert.png` | The Slack alert message in `#client-alerts` for the urgent lead from workflow 1's swap-test execution. Shows the alert formatting (urgency score, intent classification, suggested action, message excerpt) that fires only when `urgency_score >= 8 AND is_spam == false`. | 1 |
| `wf2-airtable-tickets-rows.png` | The Airtable Tickets table after running workflow 2's three test emails. Three rows: casual general (1 star), promotional spam (1 star), urgent production bug (5 stars). From Email column is intentionally hidden (the field exists in the workflow JSON; hidden in this view to keep the screenshot PII-clean). | 2 |
| `wf3-airtable-invoices-rows.png` | The Airtable Invoices table after dropping the two sample PDFs (`invoice-acme-cloud-2026-05.pdf` once and `invoice-officedepot-eu-2026-04.pdf` twice) into the watched Drive folder. Three rows demonstrate Claude correctly extracting vendor, invoice number, amount, currency (USD vs EUR), and due date across two different invoice formats; the duplicate Office Depot row demonstrates the append-only design pattern (no silent overwrites on re-runs). | 3 |

All four screenshots were captured at 1900-ish-px width to render cleanly on a standard GitHub markdown view.
