# WinCloud

Read hotel bookings, rates, room inventory, room and revenue performance, outlet menus and guest profiles from WinCloud, the hotel
management platform by RB Software, Inc., without leaving Claude. Ask in plain language, for example "find bookings made
this month for my hotel" or "how many rooms are vacant right now", and Claude answers from your own
WinCloud data.

This plugin is read-only. It cannot create, change or cancel bookings.

## Install

1. Clone this repository.
2. Start Claude Code with the plugin loaded from your checkout:

   ```bash
   claude --plugin-dir /path/to/wincloud-plugin
   ```

3. Run `/mcp` and sign in to `wincloud`.
4. Ask a question, for example "how many rooms are vacant right now?"

Once the plugin is published to the RBS organisation library, install it from there instead.

## What you need

- A WinCloud account with access to at least one hotel company.
- Sign-in through RBS Center when Claude first connects. Claude opens the sign-in page and keeps the
  session; the plugin itself stores no credentials.

## What it connects to

The plugin adds one remote MCP server, `wincloud`, at `https://api.wincloudone.com/mcp`, and the skills
listed under [Skills](#skills), which tell Claude how to use its tools well.
Claude sends your requests there over HTTPS and receives only the results you are allowed to see. The plugin
sends data to no other service.

The one local component is `skills/wincloud-report-format/scripts/chart.py`. When a report needs a chart,
Claude may run it with `python3` to turn a small JSON spec of the numbers into an SVG file, plus a PNG beside it
when the `cairosvg` package is already installed. It installs nothing, reads the spec file Claude writes, writes
only those output files, and makes no network calls. It is the only code the plugin runs on your machine.

You see only the hotels your WinCloud account can access. Every tool except List companies needs a
permission on your sign-in, so you may see only some of the [tools](#tools) below:

| Tools | Permission |
|---|---|
| Search bookings, Get reservation, Get reservation analytics, Get booking summary | `reservations:read` |
| Get rate plans, Search room tariffs, List room types, List master rates, Get rate configuration, List rate templates, List rooms, List room features, Get room performance | `rates:read` |
| Get property configuration | `rates:read` or `pos:read` |
| Get inventory | `inventory:read` |
| Search guests, Get guest reservations, Get guest | `guests:read` |
| Get outlet menu | `pos:read` |
| List property locations | none |

If a tool is missing, ask your WinCloud administrator for the permission, then sign in again.

## Tools

| Tool | What it returns |
|---|---|
| Search bookings | Bookings by booked, arrival, departure or cancellation date, status, booking number or guest name. No contact details |
| Get reservation | Every reservation sharing one booking number (several for a group), in full. No contact details, deposit or special requests |
| Get reservation analytics | Booking, cancellation and no-show rows, one per reservation dated by its arrival, with lead time, rooms and ADR. No guest names |
| Get booking summary | Arrival and departure room counts for a date range: arrived, walk-ins, still expected, checked out, still due to leave. Totals only |
| Get rate plans | Room types, rate types, rate plans and channels, without prices |
| Search room tariffs | Nightly tariffs and restrictions for a date range, room type and rate type. Weekend tariffs aren't included |
| List room types | Room types, room counts and base tariffs |
| List master rates | Master rate types that other rates derive from, without prices |
| Get rate configuration | Weekday and weekend adjustments each derived rate applies to its master rate |
| List rate templates | When each set of rate formulas applies, by weekday and yearly dates |
| List rooms | Physical rooms with room type, floor, bed type and status code |
| List room features | The room features (amenities) set up, not which rooms have them |
| Get room performance | Actual occupancy, ADR, RevPAR and revenue for past dates, month and year to date, by room type, against an earlier period |
| Get property configuration | Outlets, active room types, and whether WiFi, TV and GST are set up |
| Get inventory | Today's room-status snapshot. Future availability isn't available |
| Search guests | Guest profiles by name, email or mobile, with name, nationality and stay history. No contact details |
| Get guest reservations | A guest's reservations, filtered by status or arrival date |
| Get guest | One guest's profile: nationality, language, preferences, stay totals and newest stays, and on request their spend. No contact details or allergies |
| Get outlet menu | One outlet's menu items with group, price and status. Not sales or stock |
| List property locations | The hotels (property locations) your sign-in can see, with the company ID the other tools need |

## Skills

| Skill | Use it for |
|---|---|
| `wincloud` | Entry point: which tool answers which question, and the order to call them |
| `wincloud-daily-briefing` | Today's arrivals, departures, in-house guests and what needs attention |
| `wincloud-occupancy-forecast` | Expected occupancy from reservations already on the books |
| `wincloud-pickup-pace` | Booking pace, pickup, lead time and comparison with earlier periods |
| `wincloud-adr-revpar` | ADR, RevPAR and pricing performance by room type or rate plan |
| `wincloud-cancellations` | Cancellation and no-show rates and where revenue is lost |
| `wincloud-performance-review` | KPI and chart reports for a period, one hotel or several |
| `wincloud-report-format` | Shared presentation rules (headline, KPIs, chart, table) used by the others |

## Repository layout

```text
.claude-plugin/plugin.json   plugin manifest (name, version, author)
.mcp.json                    remote MCP server definition
skills/<name>/SKILL.md       one folder per skill
icon.png  LICENSE  README.md
```

## Develop and validate

Check the manifest and files before every commit:

```bash
claude plugin validate .
```

Bump `version` in `.claude-plugin/plugin.json` for each release. Build output (`dist/`, `*.zip`) is
git-ignored.

## Privacy

Guest addresses, email addresses, phone numbers and billing details are never returned to Claude. The data Claude
reads comes from WinCloud and is governed by your organization's agreement with RB Software, Inc.

## Support

For access or sign-in problems, a missing tool, or "You don't have access to company" errors, email
[support@rbsgo.com](mailto:support@rbsgo.com) or visit [www.rbsgo.com](https://www.rbsgo.com).

## License

Proprietary. Copyright RB Software, Inc. See `LICENSE`.
