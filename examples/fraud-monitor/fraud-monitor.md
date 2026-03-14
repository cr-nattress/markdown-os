# High Value Order Monitor

## Purpose

Monitor a Shopify store for orders exceeding a configurable dollar
threshold. Enrich each qualifying order with customer history from
the CRM, score it for fraud risk, and post a structured summary to
Slack. This is a fraud-scoring monitor, not an order management tool.

## When an order arrives

Validate that the payload contains an order ID, customer email,
line items, and a total amount. If any of these are missing, reject
the request and log a validation error.

Check whether we've already processed this order ID. If we have,
skip it silently — this is a duplicate.

Only process orders where the total exceeds the configured threshold,
which defaults to $500.

## Scoring

For the validated order, evaluate these fraud signals:

- The customer has never ordered before — this is a moderate signal.
- The shipping address doesn't match the billing address — moderate.
- There were multiple failed payment attempts — strong signal.
- The order was placed between 1 AM and 4 AM local time — weak signal.

Start with a base score of 10. Add 15 points for each weak signal,
20 for moderate, and 25 for strong. Cap the total at 100.

## After scoring

Record the order ID, the score, and the current timestamp in the
processed orders log. Keep this information for 90 days.

Format a Slack message that includes:
- The order number and total (but not payment details)
- The customer's name (mask the email — first 3 characters only)
- The fraud score with a color indicator: green below 40, yellow
  40-70, red above 70
- A list of which signals were triggered

Post this to the configured Slack channel.

If the score is above 90, also post to #security-alerts with an
urgent flag.

## If something goes wrong

If the Slack call fails, hold the message and retry up to 3 times
with increasing delays. After 3 failures, log a critical error.

If scoring fails for any reason, assign a score of 50 (uncertain)
and note in the Slack message that scoring was incomplete.

Never crash on a single bad order. Log it and continue.

## What this app does NOT do

- Does not modify orders in Shopify.
- Does not interact with customers directly.
- Does not make approval or rejection decisions.
- Does not store raw payment information.
