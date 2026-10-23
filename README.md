# WinCloud

Read hotel bookings, rates, room inventory and guest reservation history from WinCloud, the hotel
management platform by RB Software, Inc., without leaving Claude. Ask in plain language, for example
"find bookings made this month for my hotel" or "how many rooms are vacant right now", and Claude
answers from your own WinCloud data. Built-in skills also produce briefings, forecasts and KPI reports.

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
Claude may run it with `python3` to turn a small JSON spec of the numbers into an SVG file. It uses only the
Python standard library, reads the spec file Claude writes, writes the output file, and makes no network
calls. It is the only code the plugin runs on your machine.

You see only the hotels your WinCloud account can access. Every tool except List companies needs a
permission on your sign-in, so you may see only some of the [tools](#tools) below:

| Tools | Permission |
|---|---|
| Search bookings | `reservations:read` |
| Get rate plans, Search room tariffs, List room types | `rates:read` |
| Get inventory | `inventory:read` |
| Search guests, Get guest reservations | `guests:read` |

The permissions for the other tools are not listed here yet. If a tool is missing, ask your WinCloud
administrator for the permission, then sign in again.

## Tools

| Tool | What it returns |
|---|---|
| List companies | The hotels (property locations) your sign-in can see |
| Search bookings | Bookings filtered by the date made, arrival, departure or cancellation date, by status, reservation number or guest name |
| Get reservation | One reservation in full: rooms, times, rate plan, source, cancellation reason |
| Get reservation analytics | One row per reservation, for booking pace, lead time, ADR, cancellations and no-shows |
| Get rate plans | Room types, rate types, rate plans and channels, without prices |
| Search room tariffs | Nightly tariffs and restrictions for a date range, room type and rate type. Weekend tariffs aren't included |
| List room types | Room types, room counts and base tariffs |
| List master rates | The master rates other rates are derived from |
| Get rate configuration | The formulas that derive a rate from its master rate, not prices |
| List rate templates | When each set of rate formulas applies (days, seasons) |
| List rooms | Physical rooms: numbers, floors, beds, and WinCloud's stored room status code |
| List room features | The amenities the hotel has set up, not which rooms have them |
| Get inventory | Today's room-status snapshot. Future availability isn't available |
| Search guests | Guest profiles by name, email or mobile (at least 3 characters), with the guest's email and mobile |
| Get guest reservations | A guest's past and upcoming reservations |

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

Search guests returns a guest's email address and mobile number. Billing details are never returned to Claude. The data Claude
reads comes from WinCloud and is governed by your organization's agreement with RB Software, Inc.

## Support

For access or sign-in problems, a missing tool, or "You don't have access to company" errors, email
[support@rbsgo.com](mailto:support@rbsgo.com) or visit [www.rbsgo.com](https://www.rbsgo.com).

## License

Proprietary. Copyright RB Software, Inc. See `LICENSE`.
