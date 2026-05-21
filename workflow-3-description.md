# Workflow 3: Invoice PDF Parser with Google Drive + Claude

A Google Drive folder is watched on a one-minute poll. When a new PDF file lands, n8n downloads the binary, extracts text via the built-in PDF parser, and sends that text to Claude (Haiku 4.5) for structured extraction. Claude returns JSON with vendor name, invoice number, total amount, currency (3-letter ISO when visible), due date (ISO format), and an array of line items (description + amount). Each parsed invoice becomes one row in an Airtable Invoices table; the raw line-items array is JSON-stringified into a long-text field so the original detail survives even when the table view collapses it.

**Who it's for:** any small business that receives invoices as email attachments or supplier-uploaded PDFs and currently re-types them into an accounting spreadsheet or QuickBooks. Bookkeepers, ops generalists at 5-50 person companies, agencies handling client expenses, indie consultants tracking subcontractor invoices.

**Time-sink eliminated:** the monthly invoice-pile ritual of opening each PDF, copying vendor / amount / date / line items into a sheet by hand, and routing the unclear ones to a human for follow-up. At 30 invoices a month and 3 minutes each, that's 90 minutes plus the cognitive context-switch cost.

**Dollar value captured:** beyond the recovered time, structured invoice data unlocks downstream automation: aging reports, vendor spend analysis, duplicate detection. Most teams have invoice data trapped in PDFs they never query.

## Files

- `workflow-3-invoice-parser.json` — importable n8n workflow JSON
- `workflow-3-canvas.png` — screenshot of the 7-node canvas after a successful execution
- `sample-invoices/invoice-acme-cloud-2026-05.pdf` — single-currency USD SaaS invoice (3 line items, $351.80, invoice number ACS-2026-04812)
- `sample-invoices/invoice-officedepot-eu-2026-04.pdf` — EUR invoice with Dutch VAT (6 line items, €1892.05, invoice number ODE-2026-99214)
- `workflow-3-test-invoices.md` — description of the two sample PDFs and what each tests (currency, VAT, vendor-suffix variants), plus guidance for adding your own
- `sample-outputs/wf3-airtable-invoices-rows.png` — Airtable Invoices table after dropping the sample PDFs into the watch folder

## Sample output

The Airtable Invoices table after dropping the two sample PDFs into the watched Drive folder. Three rows demonstrate Claude correctly extracting vendor, invoice number, amount, currency, and due date across USD and EUR formats; the duplicate Office Depot row shows the append-only design pattern (re-running on the same PDF appends rather than silently overwriting).

![WF3 Airtable Invoices rows](sample-outputs/wf3-airtable-invoices-rows.png)

## Architecture

7 nodes: Drive Trigger polls the watched folder on a 1-minute cadence → Download PDF fetches the file binary → Extract PDF Text pulls plain text from the document → Build Anthropic Payload constructs the Claude request with a tuned invoice extractor prompt → HTTP Request to `api.anthropic.com/v1/messages` → Parse Extraction maps the structured JSON response (vendor, invoice number, amount, currency, due date, line items array) back to its source filename → Append to Invoices writes the row.

The extractor handles single-currency (USD-style) and multi-currency-aware invoices (EUR with Dutch VAT, etc.), normalizes due dates to ISO format, and emits the line-items array as JSON-stringified text for Airtable's long-text field. Text is truncated to the first 8000 characters per invoice; for longer documents the chunking strategy is part of any engagement.

The public JSON in this repo has the node structure intact but redacts the Claude system prompt and the multi-item Code-node implementations. The fully working version is delivered as part of any tier — see the main [README](README.md) for hire info.

For scanned image-only PDFs the built-in text extraction fails; the production path swaps Extract PDF Text for an OCR step (Google Vision or AWS Textract) before Claude. Available on Pro and Business tiers.
