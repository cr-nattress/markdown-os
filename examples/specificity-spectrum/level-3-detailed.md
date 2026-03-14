# Expense Tracker

## Purpose

Track personal daily expenses. Designed for quick capture — the
user should be able to log an expense in under 5 seconds. Show
spending trends to build awareness, not to budget or restrict.

## Adding an expense

Always-visible form at the top of the screen. Three fields:

- Amount (required): numeric input, auto-focused on load. Accept
  decimals to two places. No currency symbol in the input — show
  it as a prefix label.
- Category (required): single-select from the list below. Most
  recently used category is pre-selected.
- Note (optional): single-line text, max 100 characters.
  Placeholder: "what was this for?"

On submit: save the expense, clear the amount and note, keep the
category selection, re-focus the amount field. The list below
should update immediately.

## Categories

Food & drink, transport, entertainment, bills & utilities, health,
clothing, gifts, education, other.

The user can add custom categories from a settings view. Custom
categories appear after the defaults. Limit to 20 total categories.

## The main view

Top section: summary card showing this month's total spending as a
large number, with a small comparison to last month ("12% more than
last month" or "8% less"). Below the number, show a horizontal
stacked bar representing the category breakdown.

Middle section: the quick-entry form (described above).

Bottom section: expense list for the current month, newest first.
Each row shows: amount (bold), category (color-coded chip), note
(if present, muted text), and relative time ("2 hours ago",
"yesterday"). Swipe left to delete on mobile. Click to edit on
desktop.

## Persistence

Remember all expenses with no expiration. Remember the user's
custom categories. Remember which category was used most recently
for pre-selection.

## What this app does NOT do

- No budgeting, goals, or spending limits
- No bank account or card integration
- No receipt scanning (just manual entry)
- No multi-currency support
- No data export (keep it simple)

## Implementation

Build this as a React application with TypeScript and Tailwind.
Use localStorage for persistence.
