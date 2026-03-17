# Stripe Revenue Dashboard — Real-Time Revenue Tracking

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30 minutes

Daily Stripe revenue reports, MRR tracking, failed payment alerts, and customer LTV. 15-minute setup. Know your numbers every morning.

---

## What's In This Package

- Stripe API setup guide
- Daily revenue report cron
- MRR calculator
- Failed payment alert system
- Customer LTV tracking
- Webhook setup for real-time events

---

## Setup

### Get Your Stripe API Key

1. Log into [dashboard.stripe.com](https://dashboard.stripe.com)
2. Developers → API Keys
3. Copy the **Secret Key** (starts with `sk_live_` or `sk_test_`)

Store it:
```bash
openclaw secret set STRIPE_SECRET_KEY "sk_live_your_key_here"
```

### Test Your Connection

```bash
curl -s https://api.stripe.com/v1/balance \
  -u "$STRIPE_SECRET_KEY:" | jq '.available[0].amount'
# Returns available balance in cents
```

---

## Daily Revenue Report

### Report Prompt (runs at 7 AM)

```
Daily Stripe revenue report. Use Stripe API with stored credentials.

Fetch data for:
1. Yesterday's revenue (created >= yesterday 00:00, <= yesterday 23:59 UTC)
2. Month-to-date revenue (first of current month to now)
3. Last 7 days revenue
4. Failed payments from last 24 hours
5. New customers from last 24 hours

Calculate:
- Yesterday: $[amount] from [X] charges
- MTD: $[amount] from [X] charges
- Day-over-day: +/- X% vs. same day last week
- Failed: [X] failed payments totaling $[amount]
- New customers: [X]

Report format:
💰 Revenue Report — [DATE]
Yesterday: $[X] ([+/-]X% vs last [weekday])
MTD: $[X] / $[monthly goal] (X% of goal)
Failed payments: [X] — $[amount] at risk
New customers: [X]

Flag: any failed payment over $100 for immediate follow-up.

Write full report to finance/stripe/[DATE].md
```

### Stripe API Calls

```bash
# Yesterday's charges
YESTERDAY_START=$(date -v-1d -u '+%s' -j -f '%Y-%m-%d' "$(date -v-1d '+%Y-%m-%d')" 2>/dev/null || date -d "yesterday 00:00:00 UTC" '+%s')
YESTERDAY_END=$(date -d "today 00:00:00 UTC" '+%s')

curl -s "https://api.stripe.com/v1/charges?created[gte]=$YESTERDAY_START&created[lt]=$YESTERDAY_END&limit=100" \
  -u "$STRIPE_SECRET_KEY:" | jq '[.data[].amount] | add // 0'

# Failed payments (last 24h)
curl -s "https://api.stripe.com/v1/charges?created[gte]=$(date -d '24 hours ago' '+%s')&status=failed&limit=100" \
  -u "$STRIPE_SECRET_KEY:" | jq '.data | length'

# New customers (last 24h)
curl -s "https://api.stripe.com/v1/customers?created[gte]=$(date -d '24 hours ago' '+%s')&limit=100" \
  -u "$STRIPE_SECRET_KEY:" | jq '.data | length'
```

---

## MRR Calculator

### Monthly Recurring Revenue Prompt

```
Calculate current MRR from Stripe.

Steps:
1. Fetch all active subscriptions: GET /v1/subscriptions?status=active&limit=100
2. For each subscription, get the plan amount and billing interval
3. Normalize to monthly: annual plans ÷ 12, weekly × 4
4. Sum all active subscription MRR
5. Fetch any canceled subscriptions from last 30 days for churn calculation

Calculate:
- Current MRR: $[amount]
- New MRR (added this month): $[amount]
- Churned MRR (canceled this month): $[amount]
- Net MRR change: +/- $[amount]
- MRR growth rate: X%

Annualized Run Rate (ARR): MRR × 12 = $[amount]

Write to finance/mrr/[MONTH].md
```

### MRR Trending Cron

Schedule: 1st of each month at 8 AM

```
MRR snapshot for [MONTH]. Run full MRR calculation.
Compare to last month in finance/mrr/.
Calculate month-over-month growth.
Add to finance/mrr/history.md summary table.
```

---

## Failed Payment Alert System

### Real-Time Alert Prompt

```
Check for failed payments in last 24 hours.

Fetch: GET /v1/charges?status=failed&created[gte]=[24h_ago]

For each failed payment:
1. Amount
2. Customer email
3. Failure reason (card declined, insufficient funds, etc.)
4. Number of attempts

If ANY failed payment > $50: send Telegram alert immediately.
If total failed value > $200 in 24h: send Telegram alert.

Otherwise: include in daily report summary only.
```

### Recovery Prompt

For failed payments, generate recovery actions:

```
Failed payment recovery for customer [EMAIL], amount $[X].

Failure reason: [reason from Stripe]

Generate:
1. Customer email (professional, not accusatory)
2. Recommended action (retry / update card / contact customer)
3. Timeline: retry at what interval?

Standard recovery sequence:
- Day 1: Auto-retry (Stripe does this)
- Day 3: Email customer
- Day 7: Second email + dunning
- Day 14: Cancel if still failed, send cancellation email

Draft the Day 3 customer email now.
```

---

## Customer LTV Tracking

### LTV Calculation Prompt

```
Calculate customer LTV snapshot from Stripe.

Fetch all customers created in the last 12 months.
For each, calculate:
- Total amount paid (sum of successful charges)
- Subscription status (active/canceled)
- Days as customer
- Average monthly spend

Aggregate:
- Average LTV across all customers
- LTV by cohort (month acquired)
- LTV for active vs. churned customers
- Top 10 customers by total spend

Write to finance/ltv/[DATE].md

Insight: are customers from any particular month worth significantly more?
```

---

## Webhook Setup

Webhooks give you real-time events instead of polling.

### Configure in Stripe

1. Dashboard → Developers → Webhooks
2. Add Endpoint: `https://your-domain.com/webhook/stripe`
3. Select events:
   - `charge.succeeded`
   - `charge.failed`
   - `customer.subscription.created`
   - `customer.subscription.deleted`
   - `invoice.payment_failed`

### n8n Webhook Handler Template

In n8n, create:

1. **Webhook Trigger** — POST `/webhook/stripe`
2. **Switch node** — route by `event.type`
3. Branches:
   - `charge.succeeded` → log revenue + update metrics
   - `charge.failed` → send Telegram alert + log
   - `subscription.created` → log new MRR + welcome workflow
   - `subscription.deleted` → log churn + offboarding
   - `invoice.payment_failed` → dunning workflow

### Webhook Verification

Always verify Stripe webhook signatures:

```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const sig = req.headers['stripe-signature'];

let event;
try {
  event = stripe.webhooks.constructEvent(
    req.rawBody,
    sig,
    process.env.STRIPE_WEBHOOK_SECRET
  );
} catch (err) {
  return res.status(400).send(`Webhook Error: ${err.message}`);
}
```

Get the webhook signing secret from: Dashboard → Webhooks → [your endpoint] → Signing Secret.

---

## Revenue vs. Goal Tracking

Create `finance/goals.md`:

```markdown
# Revenue Goals

## Monthly Targets
| Month | MRR Goal | Actual | Status |
|-------|----------|--------|--------|
| Jan 2024 | $2,000 | $1,847 | ❌ |
| Feb 2024 | $2,500 | — | ⏳ |

## Annual Target: $[X]
## Current ARR: $[X]

## Key Metrics
- Average transaction: $[X]
- Conversion rate: X%
- Churn rate: X%/month
```

### Goal Tracking Prompt (weekly)

```
Revenue goal check. Read finance/goals.md and finance/stripe/*.md.

Current month performance:
1. MTD revenue vs. monthly goal (X% achieved, X days remaining)
2. Projected month-end at current run rate
3. What we need per day to hit goal

If behind pace:
- How much per day needed to catch up?
- Flag if mathematically impossible (>150% of recent daily average)

Update finance/goals.md with current actuals.
```

---

## Troubleshooting

**API returns 401**
- Check secret key is correct and stored as STRIPE_SECRET_KEY
- Verify you're using test key for test mode, live key for production
- Test keys start with `sk_test_`, live keys with `sk_live_`

**Revenue numbers don't match dashboard**
- Stripe charges are in cents — divide by 100 for dollars
- Check timezone: Stripe stores UTC, convert for your local reports
- Refunds reduce gross revenue — check if you want gross or net

**Webhook not receiving events**
- Verify endpoint is publicly accessible (not localhost)
- Check Stripe webhook logs for delivery attempts
- For local development, use Stripe CLI: `stripe listen --forward-to localhost:3000/webhook`

**MRR calculation off**
- Annual subscriptions need ÷ 12 to normalize
- Some customers have multiple subscriptions — sum them
- Trial subscriptions may have $0 amount — filter with `status=active` and `amount > 0`

---

*Built on OpenClaw. Requires Stripe account with API access.*
