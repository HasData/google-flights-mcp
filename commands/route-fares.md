---
description: Fares for a route and date, ranked with the emissions and the price level
---

Price a route.

Ask me for the origin, the destination and the dates if I have not given them. Origin and destination must be IATA codes, so convert a city name to one and tell me which airport you picked.

Then:

1. Call `hasdata_google_travel_flights_getGoogleFlights` with `departureId`, `arrivalId`, `outboundDate` and an explicit `type`. Send `oneWay` for a one-way trip, because the default is `roundTrip` and that fails validation without a `returnDate`. Add `sortBy: price` unless I ask for another order, plus `travelClass`, `stops` or the passenger counts if I gave them.
2. If both itinerary arrays come back empty, tell me nothing flies that day. That is a successful call, not an error, and guessing a nearby date costs another one.
3. List five itineraries: price, total duration written as hours and minutes rather than raw minutes, the number of legs, the airlines, and the connections from `layovers`, which is present on multi-leg itineraries.
4. Read `priceInsights` and say where today's fare sits against `typicalPriceRange` and what `priceLevel` reads. That answers whether to book now better than the cheapest row does.
5. Report `carbonEmissions.thisFlight` in kilograms, converting from the grams the payload gives, and say how it compares to `typicalForThisRoute`.

Stamp the answer with the date you pulled it, because fares move constantly.
