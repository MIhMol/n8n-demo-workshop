# Workflow 2 test emails

Workflow 2 polls Gmail every minute. Unlike workflow 1's webhook trigger, you can't POST a JSON payload to inject test data; you send a real email to the authenticated Gmail account and wait for the next poll.

Use these three email templates to verify the workflow end-to-end. Send each one (from any account) to the Gmail account workflow 2 is authenticated against, then watch the n8n executions list. Within ~1 minute you should see an Airtable Tickets row appear, and (for the urgent case) a Slack message in `#client-alerts`.

## Test 1: Urgent production bug

Tests that the classifier correctly flags a P1-style incident and the Slack alert path fires.

**Subject:**

```
URGENT: payment processing returning 500s in production
```

**Body:**

```
Hi support team,

Our checkout flow has been throwing HTTP 500s for the past 20 minutes. Stripe payments aren't going through and we've already lost two cart conversions that we know of. This is hitting our live customer-facing site.

Can someone look at this ASAP? Happy to jump on a call.

Thanks,
Mike (CTO, ExampleCo)
```

**Expected classification:**
- `category`: `support` or `bug`
- `urgency`: 5
- `needs_human`: true
- Slack alert: FIRES (urgency >= 4 in workflow 2 gate)

## Test 2: Casual billing question

Tests that a low-urgency request still gets classified and stored, but does not trigger the Slack alert.

**Subject:**

```
Question about my next invoice
```

**Body:**

```
Hey,

Just looking at my last invoice and noticed I was charged for the Pro plan even though I downgraded to Starter mid-month. Could someone double-check whether the prorate was applied correctly? No huge rush, just want to make sure before next month's billing.

Thanks!
Anna
```

**Expected classification:**
- `category`: `billing`
- `urgency`: 2
- `needs_human`: false
- Slack alert: does not fire

## Test 3: Promotional spam

Tests that obvious marketing outreach gets caught and doesn't trigger an alert.

**Subject:**

```
Quick question - 10x your conversion rate with our AI tool
```

**Body:**

```
Hi there,

I noticed your website and thought you'd be a perfect fit for our new AI-powered conversion optimization platform. We've helped over 5000 SaaS companies increase their conversion rates by 10x.

Can I send you a 15-minute demo? I promise you won't regret it!

Best,
Tyler Marketing
PromoBoost.io
```

**Expected classification:**
- `category`: `spam`
- `urgency`: 1
- `needs_human`: false
- Slack alert: does not fire

## How to send these without polluting your real inbox

Three options:

1. **Use a free secondary Gmail account.** Create `yourname+test@gmail.com` (Gmail ignores the `+test` suffix for delivery but treats it differently for filters). Send tests from there.
2. **Use Gmail filters.** Set up a filter that labels any email with subject prefix `[WF2-TEST]` and prepend that to the subject lines above. After testing, archive or delete by filter.
3. **Use a separate test Gmail account.** Create `n8n-test-<yourname>@gmail.com` purely for workflow testing; this is the cleanest option.

## What to verify after each test

1. Check the n8n executions list (`http://localhost:5678/executions`). The execution should be green within ~60 seconds of sending the email.
2. Open Airtable Tickets. A new row should appear with the subject, from-address, classification, and suggested reply.
3. For test 1 only: open Slack `#client-alerts`. A message should appear within seconds of the n8n execution completing.
