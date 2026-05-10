# Fantasy Football Analytics Dashboard

An interactive single-page dashboard that pulls live data from the Sleeper API and computes nine categories of fantasy football analytics for a 2025 league.

**Live data source:** Sleeper public API (no auth required for league data).  
**Tech stack:** Vanilla JavaScript, HTML, CSS — no build step, no framework, no dependencies.  
**Just open `index.html` in a browser** and the dashboard fetches the season's data and renders.

## What It Calculates

The dashboard goes beyond standard fantasy site stats — most leagues only show wins, losses, and total points. This breaks the season down across nine analytical lenses:

**Overview.** League champion, top scorer, best lineup manager, luckiest team, biggest blowout, closest matchup.

**Lineup Efficiency.** For each team, sums actual points scored vs the maximum possible points if they'd started their optimal lineup every week. Surfaces which managers are leaving points on the bench.

**Schedule Luck.** For each team, calculates an expected win total by comparing their weekly score against *every other team's* score that week — if a team would have won against 7 of the 11 other teams' scores, that week is worth 7/11 of an expected win. The gap between expected and actual wins quantifies schedule luck.

**Consistency.** Standard deviation of weekly scores, plus high/low/average. Identifies boom-bust teams vs steady producers.

**Trade Performance.** For every completed trade, sums the fantasy points each side's incoming players scored *after the trade date* minus points the outgoing players scored after the trade date. A net measure of who won each deal in retrospect.

**Waiver Wire ROI.** This was the hardest piece. Naive waiver analysis credits every team for every point a pickup scored all season — even after they were dropped. This implementation tracks player drop-weeks per roster (via the transactions endpoint) and only credits points scored during the actual window a player was on a given team. Surfaces real waiver hit rate and best individual pickups.

**Draft Analysis.** Compares the top 30 actual season scorers to the first 30 players drafted, identifies steals (late picks who outproduced their draft slot) and busts (early picks who didn't deliver).

**Playoffs.** Performance during playoff weeks (15-17), playoff consistency, best and worst single playoff weeks.

**Awards.** A computed end-of-season awards list: Champion, Top Scorer, Best Manager, Consistency King, Boom/Bust King, Dominance (most weekly #1 finishes), Cardiac Kid (most wins by <5 points), Point Miser (lowest winning score), League Taco (worst lineup efficiency), Best Record.

## Implementation Notes

The dashboard fetches data in parallel from several Sleeper endpoints:

- `/league/{id}` — league config and roster slot structure
- `/league/{id}/users` and `/rosters` — team identities
- `/league/{id}/matchups/{week}` — weekly scores including per-player breakdowns (17 weeks fetched in parallel via `Promise.all`)
- `/league/{id}/transactions/{week}` — every trade, waiver claim, and free agent add (18 weeks)
- `/league/{id}/winners_bracket` — playoff bracket for champion identification
- `/league/{id}/drafts` and `/draft/{id}/picks` — draft order
- `/players/nfl` — player database for name/position/team lookups

After fetching, the script builds three index structures: a player-week-points lookup, a player-drop-week-per-roster lookup, and roster/user maps. Every analytical view derives from those, so adding a new analysis is mostly a matter of writing a new render function — the data layer doesn't change.

The lineup efficiency calculation deserves a note: optimal lineup is approximated by taking the top N scorers from each roster's available players each week, where N is the number of starter slots from the league config. This slightly over-counts optimal in leagues with strict positional requirements (since the top scorers might not fit the slot structure), but the approximation is consistent across teams.

## Limitations

- **Single league hardcoded.** The Sleeper league ID is set as a constant at the top of the script. Adapting for any other league is a one-line change but isn't parameterized via UI.
- **Optimal lineup approximation.** As noted above, the lineup efficiency calc doesn't enforce positional slot rules.
- **Defense scoring.** Sleeper represents team defenses by team abbreviation rather than a player ID, which is handled via a fallback in `getPlayerName` but means the player database doesn't enrich defense entries.
- **Hardcoded draft steals fallback.** The "Draft Steals" tab has a manually curated list of standout late-round names for this specific league; a generic algorithmic fallback kicks in if those names aren't found in the data.

## How To Run

Clone or download the repo, open `index.html` in any modern browser. The dashboard takes 5-15 seconds to load while it parallel-fetches the season's data, then renders all nine tabs client-side.

To run for a different Sleeper league, change the `LEAGUE_ID` constant at the top of the `<script>` block.

## Author

S. Z. — a sports analytics side project covering my 2025 fantasy league. Companion to my NFL analysis work in [bears-draft-analysis](https://github.com/shzaidi1997-glitch/bears-draft-analysis) and [poles-era-audit](https://github.com/shzaidi1997-glitch/poles-era-audit).
