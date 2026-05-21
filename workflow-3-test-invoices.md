# Workflow 3 test invoices

Workflow 3 polls Google Drive every minute. To test, drop a PDF into the watched folder and wait for the next poll.

The repo ships two sample invoices in `sample-invoices/` that exercise different parts of the extractor. Both are real-looking but entirely fictional (no real vendors, no real customers).

## Sample 1: `invoice-acme-cloud-2026-05.pdf`

A typical US-vendor invoice with monthly subscription line items.

| Field | Value |
|---|---|
| Vendor | ACME CLOUD SERVICES, INC. |
| Invoice Number | ACS-2026-04812 |
| Amount | 351.80 |
| Currency | USD |
| Due Date | 2026-05-31 |
| Line item count | 3 |
| Notable features | Single currency, USD, standard 30-day terms |

**What this tests:**
- Basic vendor name extraction with `LLC` suffix
- Currency detection (USD)
- Date parsing in ISO-like format
- Multiple line items rolled into the `Line Items` field
- Two-decimal amount precision

## Sample 2: `invoice-officedepot-eu-2026-04.pdf`

A European-vendor invoice with a longer line-item list, Dutch VAT registration, and net-45 terms.

| Field | Value |
|---|---|
| Vendor | Office Depot Europe B.V. |
| Invoice Number | ODE-2026-99214 |
| Amount | 1892.05 |
| Currency | EUR |
| Due Date | 2026-06-12 |
| Line item count | 6 |
| Notable features | EUR currency, Dutch VAT number visible in metadata (NL801938472B01), B.V. (Besloten Vennootschap) suffix, net-45 payment terms |

**What this tests:**
- Non-USD currency (EUR) extraction
- Vendor suffix variations (BV vs LLC)
- Dutch tax rules (VAT shown per line item)
- Longer line-item list (6 lines) does not truncate
- Different invoice numbering scheme (OFD-NL-prefix)

## How to run

1. Confirm workflow 3 is active (toggle in n8n upper right)
2. Open your watched Google Drive folder
3. Upload either PDF (or both)
4. Wait up to 60 seconds for the Drive Trigger to poll
5. Watch the n8n executions list; the run should complete in ~10-15 seconds (PDF download + text extract + Claude call + Airtable write)
6. Open Airtable Invoices; a new row should appear with the fields above

## Adding your own test invoices

The two samples were generated programmatically via `reportlab` (see `.scripts/generate-sample-invoices.py` if you want to regenerate or vary them). For real-world testing, drop in 3-5 invoices from your own vendors to verify the extractor handles your invoice formats.

Known limitations to watch for:

- **Scanned image-only PDFs:** the workflow uses pdftotext-style extraction, which fails on raster-only PDFs. Production hardening would add an OCR fallback (Tesseract, Google Vision, or Anthropic with vision).
- **Non-Latin scripts:** Claude handles Japanese, Chinese, Arabic invoices but you may need to tune the system prompt for date format conventions and currency symbol detection.
- **Mixed-currency line items:** the system prompt assumes one currency per invoice. Multi-currency invoices need a prompt edit.
- **Handwritten notes or stamps:** ignored by text extraction; not present in the structured output.

## What to verify after each test

1. Check the n8n executions list. The execution should be green within ~30 seconds of the upload.
2. Open Airtable Invoices. A new row should appear with vendor, invoice number, amount, currency, due date, and the line-items text.
3. Spot-check the Line Items field; it should contain a readable summary of the PDF's line entries, one per line.
4. No Slack alert fires for workflow 3 (it's pure extraction; no urgency logic).
