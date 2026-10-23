---
name: wincloud-adr-revpar
description: Use when a user asks about average daily rate (ADR), revenue per available room (RevPAR), pricing performance, which room types or rate plans earn the most, how prices have changed over time, or how current rates compare with what guests actually paid. Works from WinCloud's reservation analytics, tariffs and room counts.
---

# ADR, revenue and pricing performance

Follow `wincloud-report-format`.

## ADR from bookings

1. Call `get_reservation_analytics` with `report` = `bookings`, a date range and optionally `roomTypeId`. Read all pages.
2. Each row has rooms (booked plus availed, as `wincloud` explains) and an ADR, which is the reservation's tariff as WinCloud stores it. Average ADR = Σ(ADR × rooms) ÷ Σ(rooms). Don't take a plain average of the row ADRs. Cancelled and no-show reservations are already left out.
3. Group by arrival month, room type, or meal plan for a trend or breakdown. The range filter is the **booking date**, so say that the figures cover bookings made in the range, not stays in it.
4. Rows are per reservation, not per night, so ADR here is the booking's rate, not a night-weighted figure.

## Current pricing versus bookings

- `search_room_tariffs` gives the prices and restrictions set for dates. Use `get_rate_plans` first for the room type and rate type ids. Weekend tariffs are not included, so say the picture covers weekdays only.
- Compare the **rate on the calendar** with **ADR on the books** for the same room type and arrival month. A large gap suggests discounted channels, packages or old rates; say it is a possible reason, not a finding.

## RevPAR, and what can't be calculated

- RevPAR = room revenue ÷ available room-nights. The tools give neither room revenue nor room-nights directly, so **RevPAR cannot be exact here**.
- You may give an **indicative** figure: Occupancy on the books × ADR. Label it "indicative RevPAR, based on bookings on the books, not actual revenue". Take occupancy from `wincloud-occupancy-forecast` for future dates only.
- For actual revenue and RevPAR, tell the user to use the hotel's WinCloud reports.

## Show

- **Headline:** average ADR for the period, the change against the previous period, and the top and bottom room type.
- **Chart:** ADR by arrival month (line) or by room type (sorted horizontal bars).
- **Table:** room type, rooms, ADR, with the calendar rate beside it when you fetched it.
- **Caveats:** the hotel, the date field, pages read, the currency, and the RevPAR limit above. Analytics rows carry no currency: use the location's base currency from `list_companies`, or the currency `search_room_tariffs` returns, and say which.
