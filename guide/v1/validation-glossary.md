# Validation Glossary

When you load a file, the game shows a report. **Errors block** the import (fix and
re-export). **Warnings proceed** — they tell you what the game filled in or adjusted.
**Info** summarizes the league. The report lists all errors at once, so you can fix a
whole batch per attempt.

## Errors (must fix)

| code | means | fix |
|---|---|---|
| `invalid-json` | The file isn't valid JSON. | Check for a trailing comma, unquoted key, or a broken merge. |
| `not-an-object` | The top level isn't a JSON object. | Wrap everything in `{ ... }`. |
| `missing-format-version` | No `formatVersion`. | Add `"formatVersion": 1`. |
| `unsupported-format-version` | A `formatVersion` this build doesn't read. | Use `1`. |
| `missing-required-field` | A player/staff is missing `name`, `team`, `position`, `ovr`, or a staff `role`. | Add the named field (the path points at it). |
| `bad-field-type` | A field has the wrong type (e.g. `ovr` as a string). | Use the type in the format guide. |
| `unknown-team` | A `team` / `championTeamId` isn't one of the 32 ids. | Use a lowercase team id. |
| `unknown-position` | A `position` isn't one of the 17. | Use a valid position code. |
| `unknown-role` | A staff `role` isn't one of the 15. | Use a valid role code. |
| `impossible-value` | A value is out of range — `ovr` > 100, `pot` < `ovr`, `guaranteedM` > `totalM`, draft round > 7, a team both champion and runner-up. | Correct the number. |
| `duplicate-player` | Two players share a name on the same team. | Rename one (or fix a merge that doubled a team). |
| `duplicate-team` | A team is overridden twice. | Keep one TeamFact per team. |
| `duplicate-staff-role` | Two staff hold the same role on one team (e.g. two HCs). | One of each role per team. |
| `forbidden-roster-shape` | A team has more than 90 players. | Trim to ≤ 90. |
| `cap-breaking` | A team's stated contracts exceed the $280M cap. | The report lists the top deals driving it — trim `totalM`, drop a contract, or spread it over more `years`. |
| `career-stat-future-season` | A `careerStats` line's `season` is at or after `meta.season`. | A career line is a past season — use a calendar year before `meta.season`. |
| `career-record-future-season` | A `careerRecord` line's `season` is at or after `meta.season`. | A coach's record is a past season — use a year before `meta.season`. |
| `record-contradicts-claim` | A `WON_SB` careerRecord names a champion the file already gave to a different team that season. | Fix the year, the team, or the conflicting champion claim. |
| `unknown-award` | An `award` claim isn't `COTY` or `EOTY`. | Use `COTY` (Coach of the Year) or `EOTY` (Executive of the Year). |
| `legacy-still-active` | A legacy entry's career ends at or after your file's season. | A legacy record is a RETIRED figure — seat a working coach with a `staff` entry instead. |
| `legacy-unknown-role` | A legacy `roles` value isn't `HC` or `GM`. | Those are the only two the legacy boards track. |
| `duplicate-legacy` | The same legacy name appears twice. | Keep one entry per legend. |

## Warnings (import still proceeds)

| code | means |
|---|---|
| `filler-generation` | The game generated depth/camp players to complete thin rosters. The count is shown; every filler is flagged `generated: true`. |
| `feasibility-adjusted` | Your stated attributes made the stated `ovr` unreachable; the attributes were honored and OVR landed where they put it (delta shown). |
| `pot-clamped` | A stated `pot` below the materialized `ovr` was raised to `ovr` (potential can't sit under current ability). |
| `cap-approx-high` | A rough parse-time estimate flags a team as possibly over-cap. The exact check runs at import; trust the blocking `cap-breaking` error over this heads-up. |
| `chronology-conflict` | Two claims name different champions for one season; the first (in file order) wins. |
| `history-out-of-range` | A `history`/accomplishment season is outside the fillable 30-year window; the claim is ignored. |
| `history-blast-radius` | You included history/accomplishments — the whole league's fabricated past is reshaped, so teams you didn't author get different coaching histories than a default game. |
| `empty-history-fact` | A history fact names neither champion nor runner-up; it does nothing. |
| `fa-with-contract` | A free agent carried a contract; it's ignored (FAs are contract-less). |
| `fa-with-team` | A `freeAgents` entry named a team; it's treated as unsigned. |
| `draft-and-udfa` | A player is both drafted and UDFA; the draft origin is used. |
| `unknown-attribute` | An attribute key isn't recognized (likely a typo); it's ignored. |
| `unknown-personality` / `unknown-backstory` / `unknown-trait` | A staff value isn't in the catalog; it's sampled instead. |
| `unknown-market-size` / `unknown-conference` / `unknown-division` | A team override value isn't recognized; the default is kept. |
| `bad-color` | A team color isn't a `#RRGGBB` hex; the default is kept. |
| `bad-jersey` | A jersey isn't an integer 0–99. |
| `implausible-age` / `implausible-birthYear` | An unusual (but not impossible) age; double-check it. |
| `unknown-field` | An unrecognized field name (likely a typo — e.g. `positon`); it's ignored. |
| `career-stat-out-of-range` | A `careerStats` `season` is more than 30 years before `meta.season`; the line still displays. |
| `career-stat-unknown-team` | A `careerStats` line's `team` isn't one of the 32 ids; it's shown as the raw id. |
| `stats-vs-position` | A `careerStats` stat doesn't fit the player's position (a QB with `tackles`); the line is kept as authored. |
| `unknown-stat` | A `careerStats` stat name isn't recognized (likely a typo); it's ignored. |
| `career-record-out-of-range` | A `careerRecord` `season` is more than 30 years before `meta.season`; the line still displays. |
| `career-record-unknown-team` / `career-record-unknown-role` | A `careerRecord` `team`/`role` isn't recognized; shown as-is / his current role is used. |
| `record-length-mismatch` | A `careerRecord` W+L+T is far from a 17-game season; double-check it. |
| `unknown-playoff-result` | A `careerRecord` `playoffResult` isn't one of `MISSED WILDCARD DIVISIONAL CONF LOST_SB WON_SB`; treated as `MISSED`. |
| `award-on-history` | An `award` was put on top-level `history`; awards belong on a coach's `accomplishments`, so it's ignored there. |
| `contract-expired-at-import` | A rostered player's contract already ran out **before** your file's season. He loads anyway, with zero years left, and hits free agency at the next roll. Almost always the extension trap — add `throughSeason`. |
| `contract-starts-before-signing` | `throughSeason` minus `years` puts the deal's start *earlier* than `yearSigned`. One of the three numbers is off. |
| `legacy-unknown-team` | A legacy entry names a team that doesn't exist; it's shown as the id. |
| `legacy-accomplishment-outside-career` | A title/award season falls outside the stated `careerYears`, so it can't be stamped on his record. |
| `legacy-hof-before-retirement` | `hofYear` precedes the end of his career. |
| `legacy-claim-conflict` | A legend claims a title or award season that an ACTIVE imported coach already claimed. The active coach takes precedence; the legend's claim is dropped. |
| `unknown-tendency` | A `tendencies` id the game does not know. | Use an id from the format guide's Tendencies table. |
| `tendency-wrong-group` | A `tendencies` id that belongs to another position group (a `pressBias` on a QB). | Use his own group's ids; the stray one is dropped. |
| `bad-tendency-level` | A `tendencies` value that is not an integer from 1 to 5. | Levels are 1 (Rarely) to 5 (Constantly). |

## Info (just so you know)

Teams covered, players authored, free agents, staff imported, history facts applied, the
per-team fidelity-tier profile, your league's **OVR distribution vs. the default** — so you
know at a glance whether your league runs hot, cold, or on par — and a **contract summary**
(how many deals you authored, how many state `throughSeason`, and how many expire in the
file's own season). That last number is the expiry-cliff detector: if most of your league is
walking into free agency at once, you have hit the extension trap.
