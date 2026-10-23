---
name: wincloud
description: Use when a user asks about their hotel's data in WinCloud, such as bookings, arrivals, departures or cancellations, one reservation in detail, booking pace and demand, room types, rooms and features, rate plans, nightly tariffs and how rates are derived, rooms occupied or vacant today, or a guest's profile and stay history. Also use for any request for stats, summaries, insights, charts, KPIs, reports, forecasts or analysis about that data. Explains which WinCloud tool answers which question, the order to call them in, and what the tools cannot do.
---

# Using WinCloud

**Every answer that summarises numbers (counts, totals, trends, comparisons, KPIs) follows `wincloud-report-format`: headline, KPI row, a real chart, then a detail table, and the user does not have to ask for the chart. When they say "all locations", use the locations from the `companyId` description or `list_companies` and show them side by side in one chart. Never answer such a question with bullets or a table alone. Lists of individual bookings or guests are the exception: a table.**

WinCloud is a hotel property management system. Its tools read one property location's (one hotel's) data at a time. They are read-only today: they never create, change or cancel anything, and they never return payment details.

## Before any call: the location

Every tool needs a `companyId`, the WinCloud code of one property location, that is one hotel (for example `WS`). WinCloud calls locations "companies", so say "hotel" or "location" to the user.

- If the tool's `companyId` description lists the locations the user is signed in to, use one of those. If there is only one, use it without asking.
- If there are several and the user didn't name a hotel, ask which one, or call the tool once per location when they asked about all of them.
- If no locations are listed, call `list_companies` to see which hotels the user has; skip any it marks "not in your sign-in". The `WS` in the description is only an example.
- Never guess a code. If a call fails with an access error, the user can't see that location: say so and offer the ones they can see.

## Which tool answers which question

| The user wants… | Tool |
|---|---|
| Bookings made, arriving, departing or cancelled over a period, by status, number or guest name | `search_bookings` |
| One reservation in full: rooms, times, rate plan, source, OTA reference, cancellation reason | `get_reservation` |
| Booking pace, lead times, ADR, cancellations or no-shows | `get_reservation_analytics` |
| The room types, rate types and rate plans a hotel sells, and the channels it sells through | `get_rate_plans` |
| Prices and restrictions (stop-sell, closed to arrival or departure, minimum or maximum stay) for dates | `search_room_tariffs` |
| Room types with their room counts and base prices | `list_room_types` |
| The master rates other rates are derived from | `list_master_rates` |
| How a derived rate is calculated from its master rate | `get_rate_configuration` |
| When each set of rate formulas applies (days, seasons) | `list_rate_templates` |
| The physical rooms: numbers, floors, beds | `list_rooms` |
| The room features (amenities) the hotel has set up | `list_room_features` |
| How many rooms are occupied or vacant right now | `get_inventory` |
| Find a guest's profile, email or mobile | `search_guests` |
| A guest's past and upcoming stays | `get_guest_reservations` |
| Which hotels (property locations) the user can see | `list_companies` |

**One guest or many?** For one guest's whole stay history, use `search_guests` then `get_guest_reservations`. For arrivals, departures, in-house guests or cancellations across the hotel, use `search_bookings`: today's arrivals are `arrivalFrom` and `arrivalTo` set to today, in-house guests are status `CheckedIn`.

## Analysis, forecast and report questions

These have their own skills. Load the one that fits, then answer as `wincloud-report-format` says.

| The user wants… | Skill |
|---|---|
| A period review, report or comparison across hotels or periods | `wincloud-performance-review` |
| Today's picture: arrivals, departures, how full now | `wincloud-daily-briefing` |
| How full the hotel will be, weak dates, rooms left to sell | `wincloud-occupancy-forecast` |
| Booking pace, pickup, lead time, ahead or behind last year | `wincloud-pickup-pace` |
| Cancellations, no-shows, how late guests cancel | `wincloud-cancellations` |
| ADR, pricing performance, indicative RevPAR | `wincloud-adr-revpar` |

## Call them in the right order

- **Prices for a room and rate:** call `get_rate_plans` first to get the exact room type and rate type ids, then pass those ids to `search_room_tariffs`. Don't pass a room name such as "Deluxe" as an id.
- **A guest's stays:** call `search_guests` first (by name, email or mobile, at least 3 characters), then pass the guest id it returns to `get_guest_reservations`. If several guests match, show them and ask which one.
- **One booking in detail:** if the user doesn't know the number, find it with `search_bookings`, then call `get_reservation`.
- **How a rate is built:** `list_master_rates` and `list_rate_templates` give the ids, `get_rate_configuration` gives the adjustments. For the resulting prices, use `search_room_tariffs`.

## Dates

- Send dates as `yyyy-MM-dd`, and the end date must not be before the start date.
- Turn relative dates ("this month", "next week") into exact dates yourself, and say which range you used.

## Limits to respect

Say plainly when a question falls outside these limits instead of answering it with the wrong tool.

- **`search_bookings` date pairs each filter their own date.** `startDate` and `endDate` are the date the booking was made; use `arrivalFrom`/`arrivalTo`, `departureFrom`/`departureTo` or `cancelledFrom`/`cancelledTo` for the others, and say which one you used.
- **In `get_reservation_analytics` bookings rows, count rooms as booked plus availed.** Reservations that have already happened come back as `0 room(s) booked, 1 availed`, so booked alone gives 0 for any past period. Use booked + availed as a row's rooms, and say you did. A negative "day(s) ahead" means it was booked after the arrival date.
- **`get_reservation_analytics` returns rows, not totals.** Each row is one reservation, dated by its arrival date, not one row per night; add them up across pages for totals, and say if you stopped before the last page. Its date filter means the booking date for bookings, the cancellation date for cancellations and the arrival date for no-shows.
- **`get_rate_configuration` describes formulas, not prices.** A missing adjustment means none is set, not 0.
- **`list_room_features` can't say which rooms have a feature**, and `list_rooms` shows WinCloud's room status code as stored, not today's occupancy.
- **`get_inventory` is today only.** It can't show availability on future dates, and tariffs are not availability.
- **`search_room_tariffs` does not include weekend tariffs.**
- **Guest email and mobile come only from `search_guests`.** Bookings, reservations and a guest's reservation list don't include them. If a guest's profile has none, say so; never invent one.
- **Read-only.** If asked to book, change or cancel something, say this has to be done in WinCloud itself.
- **A tool may be missing.** Each tool except `list_companies` needs a permission on the user's sign-in. If the tool for a question isn't available, say the user's account doesn't have that permission yet and suggest they ask their administrator, then sign in again. Use another tool only if it really answers the question (such as `search_bookings` by guest name), and say what it can't show.

## Paging

Results come a page at a time (20 by default, up to 50). Ask for 50 a page.

- **Summary questions** ("find bookings", "how many", "bookings by month", a trend or a comparison): read every page before answering, up to 15 pages per location, then show the headline, KPI row and chart from `wincloud-report-format`. Don't stop at page 1 and offer to continue. If there are more than 15 pages, say the figures cover only what you read.
- **List questions** ("list", "show me the bookings", one guest's stays): show the first page as a table and say there are more pages.

`get_rate_plans` isn't paged: it shows at most 100 rate plans and says when the rest are not listed. Room types and rate types are always listed in full, so pass those ids to `search_room_tariffs` for a specific room type and rate type.

## Answering

- **Start with a one-sentence summary, in every response:** the count, the total, the trend or the yes or no, in at most 25 words. This holds for lists, lookups and "can't do that" answers too.
- **Then show the detail as `wincloud-report-format` says:** load it for any answer with more than one number. Use a table for lists of bookings, guests, room types or prices, and plain text for a single fact.
- Report only what the tools returned. Don't fill gaps with estimates.
- Use the hotel's own names for room types and rate plans, as the tools return them.
- Say "pricing" or "rates" to the user rather than "tariffs".
