# Weather Wardrobe

## Purpose

Help someone decide what to wear today. Given their location, check
the current weather and today's forecast, then recommend clothing
that makes sense for the conditions — not just temperature, but
precipitation, wind, humidity, and how the day will change from
morning to evening.

This is not a weather app. This is a wardrobe advisor that happens
to use weather data.

## When the user opens the app

Ask for their location. If they've used the app before, remember
their last location and offer it as a default. Accept a city name
or zip code.

Once we have a location, fetch the current conditions and today's
hourly forecast.

## Understanding the weather

From the forecast, extract these key factors:

- **Temperature range**: the low and high for today, and the current temp.
- **Precipitation**: is rain, snow, or sleet expected? What probability? When?
- **Wind**: is it notably windy? Sustained speeds above 15mph matter.
- **Humidity**: is it muggy (above 70% with temp above 75°F) or dry?
- **UV index**: is sun protection needed (index above 5)?
- **Temperature swing**: does the day start cold and get warm, or vice versa?

These factors combine into a **comfort profile** — not just "it's
55°F" but "it's cool now, warming to mild by afternoon, with a
chance of rain after 3pm and moderate wind all day."

## Building the recommendation

Based on the comfort profile, recommend clothing across these
categories: base layer, mid layer, outer layer, bottom, footwear,
accessories.

Don't just list items. Explain *why* each recommendation makes
sense. "Light jacket — it's mild now but drops to 52°F by evening"
is better than just "light jacket."

## Handling tricky conditions

When the temperature swings more than 20°F through the day,
recommend layers and explicitly say "dress in layers."

When rain is possible but not certain (30-60%), recommend keeping
an umbrella handy but don't make rain gear the focus. Above 60%,
lead with rain protection.

When conditions are severe — extreme cold, heat advisory,
thunderstorms — lead with a safety note before any clothing advice.

## The display

Show the comfort profile visually — a human-readable summary.
Show clothing recommendations grouped by category with reasoning.
Show an hour-by-hour mini forecast for context.
Include a simple indicator for outfit complexity: "simple day,"
"plan ahead," or "stay flexible."

## Remembering preferences

Remember the user's location. If the user dismisses a recommendation
type consistently, deprioritize it — move it to an "also consider"
section. Remember the last 7 days of recommendations.

## What this app does NOT do

- No extended forecast beyond today.
- No shopping links or product recommendations.
- No social features.
- No outfit photos or style advice.
