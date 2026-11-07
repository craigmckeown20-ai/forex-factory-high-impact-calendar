# USD High Impact Calendar

A Google Calendar feed with only the **high-impact US economic events** (CPI, jobs report, Fed decisions and similar), filtered from the Forex Factory weekly feed.

The full Forex Factory calendar has every currency and every impact level, which is too much for a phone calendar. This repo keeps just USD + High and publishes it as an `.ics` URL you can subscribe to.

## How it works

1. A GitHub Actions workflow runs daily (05:15 UTC) and on Sunday evening.
2. `src/build_calendar.py` downloads both `ff_calendar_thisweek.json` and `ff_calendar_nextweek.json` when the next-week endpoint is available, then keeps `country == "USD"` and `impact == "High"`.
3. It merges the results into `data/events.json` and writes `docs/usd-high-impact.ics`, served by GitHub Pages.

This provides a rolling outlook for this week and next week. Forex Factory does not always publish the next-week JSON endpoint (it can return 404), so the workflow treats it as optional: the current week still updates, and previously stored next-week entries are not deleted just because that endpoint is unavailable. The merge step also keeps past events for 45 days. When a feed is available, events that disappear from its covered date range are removed, so cancelled or moved releases do not leave stale duplicates.

Standard library Python only (3.10+). No dependencies.

Each event's calendar ID is derived from country, title and exact start time, so
two entries with the same title on one day (two Powell appearances, for example)
both survive. A rescheduled event gets a new ID, and the stale one is removed
because it sits inside the feed's week but is no longer in the feed.

## Set up your own copy

1. Fork this repo, or create a new repo and push these files.
2. **Settings > Pages**: Source "Deploy from a branch", branch `main`, folder `/docs`. Save.
3. **Actions** tab: enable workflows if asked, open "Update calendar", click **Run workflow**.
4. After it finishes, your feed is at:
   `https://<your-username>.github.io/<repo-name>/usd-high-impact.ics`

## Subscribe in Google Calendar

1. On a computer, open Google Calendar.
2. **Other calendars > + > From URL**, paste the feed URL, click **Add calendar**.
3. Open the calendar's **Settings > Event notifications** and add a default notification, for example 60 minutes before.

Notes:
- Google refreshes subscribed calendars on its own schedule, often every few hours. That is fine for releases scheduled days ahead.
- Google ignores alarms inside subscribed `.ics` files, so notifications must be set in the calendar's settings as above.
- Times are stored in UTC. Google shows them in your own time zone, including daylight saving changes.

## Change the filter

Edit the settings at the top of `src/build_calendar.py`:

```python
COUNTRIES = {"USD"}
IMPACTS = {"High"}
KEEP_DAYS = 45
EVENT_MINUTES = 15
```

Some weeks have no USD High events at all. The feed is then valid but the
calendar contains only past events, which is expected, not a failure.

## Security notes

- No secrets or API keys are used.
- Only the scheduled workflow can write, with `permissions: contents: write`. The test workflow is read-only.
- Workflows run on schedule, manual trigger, push and pull request. The write workflow never runs on pull requests from forks.
- Text from the feed is escaped before it goes into the `.ics` file.

## Data source and disclaimer

Event data comes from [Forex Factory](https://www.forexfactory.com/calendar) via its public weekly feed. This project is not affiliated with Forex Factory. Check their terms of service before redistributing the data. Nothing here is financial advice.
