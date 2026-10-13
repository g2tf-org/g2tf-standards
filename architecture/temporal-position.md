# Temporal Position: Where Grid 2 Sits in Time

**Status:** Draft — seeking input · **Editors:** TBD · **Source:** [RFC #1](../rfcs/grid-2-rfc-1.pdf), §Proposed Scope, Figure 4

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are to be interpreted as described in RFC 2119 / RFC 8174.

## The one-minute window

Grid 2 operates in **near real-time dispatch windows of one minute**, deliberately positioned between Grid 1's two existing control regimes:

```
~5 min    Grid 1 — Market Dispatch Domain
          nodal pricing, forward dispatch scheduling
────────────────────────────────────────────────────────
 1 min    Grid 2 — DESP  (service classification & prioritization)
          Grid 2 — GCAP  (fair service allocation & scheduling)
          Grid 2 — G2P   (addressing, service configuration, response)
────────────────────────────────────────────────────────
seconds   Grid 1 — Fast Operational Domain
          frequency response, AGC, reserves
```

## Normative requirements

- Grid 2 allocations MUST apply to whole one-minute intervals; sub-minute response is out of scope for the orchestration plane.
- Grid 2 elements MUST NOT interfere with, substitute for, or assume the function of sub-second primary controls (governor response, AGC, protection).
- Grid 2 clearing MUST complete within the interval such that Members receive commit-% with time to self-dispatch before the interval opens. *(Exact timing budget: open question — see Discussion: Engineering.)*
- Grid 2 SHOULD consume, not contradict, forward information from the market dispatch domain; where a Grid 1 dispatch instruction conflicts with a Grid 2 allocation, the Grid 1 instruction prevails.

## Rationale

- **Bet on current grid physics.** By committing only above the timescales where Grid 1's primary controls operate, Grid 2 lets Grid 1's ancillary services absorb errors without modification. NERC CPS1/BAL metrics remain the measures of balancing performance; the size of ancillary services can be adjusted up or down over time to match observed need.
- **Coexist with markets.** Sitting below the five-minute clearing domain means Grid 2 can be adopted voluntarily by select participants in a domain without any market rule change, and lets stakeholders in ERCOT and PJM run these control schemes so they coexist and reinforce each other (PCLR, IRAS/PRD compliance in the minutes timeframe).

## Dispatch and settlement clocks (informative)

The figure above places the one-minute window against the market's dispatch clock.
Energy is paid for on a second clock, the settlement interval, which is often longer.
[Overview](overview.md) keeps commercial settlement on Grid 1's slower clock; this section names that clock in three markets, since it is the interval on which any effect of Grid 2 on settlement appears.

**ERCOT.**
SCED calculates LMPs "approximately every five minutes, or more frequently if necessary" ([Nodal Protocols][ercot-protocols] §6.5.7.3(1)).
Real-time energy settles on 15-minute Settlement Intervals, so each holds fifteen Grid 2 windows.
An interval's Settlement Point Price is a time-weighted average over the SCED intervals that cover it, each weighted by the seconds it spends inside the interval (§6.6.1).
SCED intervals vary in length, so the fixed grid in ERCOT's real-time market is the settlement grid, not the dispatch clock.
Energy imbalance at a Load Zone is computed from interval totals at that price (§6.6.3.2): the formula sees a load's metered energy for the interval, not how it was spread across the minutes.
Storage charging at a Resource Node is the exception: its price weights each SCED interval by its duration and by the resource's own charging Base Point (§6.6.3.1(3)).

**Great Britain.**
Settlement under the Balancing and Settlement Code runs on Settlement Periods, "a period of 30 minutes beginning on the hour or the half-hour" ([BSC definition][elexon-settlement-period]), each holding thirty windows.

**Singapore (NEMS).**
The dispatch period and the settlement interval are both defined as "a thirty-minute time interval beginning on the hour or the half-hour" ([Market Rules][sg-market-rules], Chapter 8, 1.1.75 and 1.1.291).
EMC begins computing each real-time dispatch schedule five minutes before its period starts, every 30 minutes (Appendix 6A).
Dispatch and settlement share one clock, so the market's dispatch clock is thirty Grid 2 windows long, not the five drawn in the figure.

## Clock-change days (informative)

In all three markets the market day is a local civil day, so where the clocks change, two days a year are not 24 hours long.
ERCOT's Operating Day runs from hour ending 0100 to 2400 in Central Prevailing Time, which follows the clocks in Austin, Texas ([Nodal Protocols][ercot-protocols] §2).
When the clocks go back, a local hour occurs twice: the day has 25 hours, 100 Settlement Intervals and 1,500 one-minute windows.
When they go forward, a local hour is skipped: 23 hours, 92 Settlement Intervals and 1,380 windows.
Great Britain's Settlement Day has 50 Settlement Periods when its clocks go back and 46 when they go forward ([Elexon][elexon-settlement-period]).
Singapore has kept UTC+8 since 1982 ([tz database][tz], zone Asia/Singapore), so every NEMS trading day has 48 periods.
On those long and short days a local clock label alone is ambiguous: when the clocks go back, a label such as 01:30 names two windows an hour apart, and when they go forward, a label inside the skipped hour names none.
A UTC instant, or a local time written with its UTC offset, names each window exactly once.
ERCOT's interfaces mark the repeated hour in several ways: a `DSTFlag` column in its price reports ([api-specs][ercot-api-specs]), an `R` suffix in EWS transaction IDs ([`2R`][ercot-mrid]), `2*` in EWS report payloads, and in submitted offers only the UTC offset, `01:00:00-05:00` for the first 01:00 and `01:00:00-06:00` for the second ([Appendix H][ercot-appendix-h]).
Of these, only the offset can be read without the market's own calendar rules.

## Open items

- Interval phase alignment across federated domains (common epoch vs. per-domain clocks)
- Late-arrival and clock-skew tolerances for Member broadcasts
- Window identifiers across clock changes (UTC start instant vs. local label; see Clock-change days)

[ercot-protocols]: https://www.ercot.com/mktrules/nprotocols/current
[ercot-mrid]: https://developer.ercot.com/applications/ews/Services%20Organization/#management-and-use-of-transaction-ids-mrids
[ercot-appendix-h]: https://developer.ercot.com/applications/ews/Appendices/Appendix%20H_%20DST%20XML%20Examples/
[ercot-api-specs]: https://github.com/ercot/api-specs
[elexon-settlement-period]: https://www.elexon.co.uk/bsc/glossary/settlement-period/
[sg-market-rules]: https://www.home.emcsg.com/about-the-market/rules/market-rules
[tz]: https://www.iana.org/time-zones
