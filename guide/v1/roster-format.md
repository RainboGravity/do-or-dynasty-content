# Format Guide (formatVersion 1)

The complete field reference. This is a **frozen public contract** — field names and tier
definitions won't change without a `formatVersion` bump.

## Top-level shape

A single `.json` object. Only `formatVersion` is required; everything else is optional.

```jsonc
{
  "formatVersion": 1,
  "meta":       { "title": "...", "author": "...", "season": 2026, "notes": "...", "showMascots": true },
  "teams":      [ /* TeamFact[]    — optional per-team overrides */ ],
  "players":    [ /* PlayerFact[]  — rostered players */ ],
  "freeAgents": [ /* PlayerFact[]  — the free-agent pool (contract-less) */ ],
  "staff":      [ /* StaffFact[]   — coaches / front office */ ],
  "history":    [ /* HistoryFact[] — league championship history */ ],
  "legacy":     [ /* LegacyFact[]  — historic HOF coaches & executives */ ],
  "hallOfFame": [ /* HofPlayerFact[] — Hall of Fame PLAYERS (Canton + club halls) */ ],
  "draftClasses": [ /* DraftClassFact[] — custom draft classes, one per year */ ]
}
```

- **`draftClasses`** loads custom draft prospects for one or more draft years. See the
  dedicated **[draft-class-format.md](draft-class-format.md)**. It is fully
  additive: a file without it generates its classes exactly as before.

- **`meta.season`** is the calendar year of the league's first season (default `2026`).
  It's also the year contract `yearSigned` and history `season` are measured against.

## Team ids

Use the lowercase team id everywhere a `team` is required. The 32 ids:

```
buf mia ne  nyj   bal cle pit cin   hou ind jax ten   den kc  lv  lac
dal nyg phi was   chi det gb  min   atl car no  tb    ari lar sf  sea
```

You **cannot add or remove teams** (v1). A team you don't list keeps its shipped identity.

**Mapping a league onto the 32 slots.** The ids are slots, not a league. Any league with up
to 32 franchises maps onto them: override each slot's `city`, `nickname`, colors and
conference in `teams` (below) and the slot *is* that franchise. Three rules keep a mapping
clean:

- **A franchise keeps one id through relocations and renames.** Use the id of the city the
  franchise plays in *today* for every era: the slot is the franchise, the `city` override
  is the era. The published real-league file does exactly this for its history blocks.
- **An id you do not use is filled by the game.** For a season with fewer than 32 teams,
  leave the extra ids unauthored and the game generates those clubs (the report says how
  many), or override them as fictional expansion clubs if you prefer to name them.
- **Historical money does not have to be scaled.** The game enforces a modern cap. For a
  historical season either omit `contract` on everyone (the game generates cap-valid deals,
  which is the default for depth already) or scale every figure so each team's total lands
  under the cap. Omitting is the safer choice.

## Positions

```
QB RB FB WR TE   OT OG C   EDGE DT MLB OLB   CB FS SS   K P
```

## PlayerFact

`ovr` is the **only required anchor** — it's present at every tier. Everything else is
optional; how much you provide is the player's *fidelity tier* (see below).

| field | type | notes |
|---|---|---|
| `name` | string | **required.** "First Last". |
| `team` | string \| null | **required.** Team id, or `null` for a free agent. |
| `position` | string | **required.** One of the 17 above. |
| `ovr` | number | **required.** 0–100. The anchor — attributes are generated to hit it. |
| `pot` | number | 0–100 potential ceiling. Must be ≥ `ovr` (raised to `ovr` with a warning if lower). Omit → derived from ovr + age. |
| `age` | number | Omit → derived. Or use `birthYear`. |
| `birthYear` | number | Alternative to `age`. |
| `jersey` | number | 0–99. |
| `college` | string | Free text. |
| `traits` | string[] | PLAY-AFFECTING traits (free text; flavor). See the note on retired temperament traits below. |
| `attributes` | object | Any subset of the attribute vocabulary (below). Stated values are **hard constraints**. |
| `draft` | object | `{ "year": 2021, "round": 1, "pick": 12, "team": "mia" }`. `pick` is the **overall** pick (1 = first off the board), not the pick's place inside its round. `team` is the club that MADE the selection — optional, and deliberately not the same field as his current `team`. |
| `udfa` | boolean | Undrafted. Mutually exclusive with `draft`. **Saying nothing is not the same as `udfa: true`** — a player with neither reads as "origin unrecorded" and his profile says "Entered the league 2019" rather than calling him undrafted. Set it where you know he went undrafted; leave both out where you simply do not know. |
| `contract` | object | `{ "years", "totalM", "guaranteedM", "yearSigned", "throughSeason" }` — money in **millions**. See the extension trap below. |
| `careerStats` | object[] | Per-season stat lines — the player's real statistical past (see below). |

### Attributes vocabulary (for Tier 2–3)

State only the ones you know — the burner's `speed`, the cannon `throwPower`. The game
fills the rest coherently around them. All are 0–100.

```
Physical:   speed acceleration strength agility stamina durability
Mental:     awareness playRecognition decisionMaking learningRate coachability
Ball:       vision ballSecurity elusiveness trucking contactBalance breakTackle
Passing:    throwPower shortAccuracy mediumAccuracy deepAccuracy throwOnRun playAction
Receiving:  catching catchInTraffic routeRunning separation
Blocking:   zoneBlocking powerBlocking blockingPass
Defense:    manCoverage zoneCoverage press powerMoves finesseMoves blockShedding pursuit hitPower tackling
Kicking:    kickPower kickAccuracy puntPower puntAccuracy
```

### The `ovr` / attributes relationship (feasibility)

`ovr` is the anchor. When you state **only** `ovr`, attributes are generated to land on it.
When you state attributes too, they're **honored exactly** and the game re-solves the rest
to hit your `ovr`. If your stated attributes make the `ovr` impossible (e.g. every core
skill pinned at 40 but `ovr: 95`), the honored attributes win and the report shows the
delta ("stated 95, materialized 88"). This is expected — trust your attributes or trust
your ovr, not both when they conflict.

## Contracts (`contract`)

```json
{ "years": 4, "totalM": 160, "guaranteedM": 110, "yearSigned": 2024, "throughSeason": 2029 }
```

### ⚠ The extension trap — read this before you write a single contract

**Any deal signed BEFORE your file's `meta.season` must state `throughSeason`** — the calendar
year of the deal's **final season**. It is the fact every contract site prints: *"under contract
through 2029."*

Here is why. When a player signs an **extension**, he is *already under contract* — so the new
money doesn't start until the following year. The advertised length counts only that new money.
Patrick Surtain signed a 4-year extension in 2025; it runs **2026–2029**. Write it the obvious
way and the game does the obvious arithmetic:

```json
{ "years": 4, "yearSigned": 2025 }     →  2025, 2026, 2027, 2028   ✗ a year short
{ "years": 4, "yearSigned": 2024 }     →  2024, 2025, 2026, 2027   ✗ TWO years short
```

Both are wrong, and nothing looks wrong about them. State the end instead and the deal lands
exactly where it belongs:

```json
{ "years": 4, "yearSigned": 2025, "throughSeason": 2029 }   →  2026 … 2029   ✓
```

`throughSeason` **anchors the deal from its end** (`start = throughSeason − years + 1`). It does
not change `years`, so the AAV and the cap hits stay exactly as you wrote them.

You can skip `throughSeason` on a **rookie deal or a free-agent signing made in the file's own
season** — those genuinely start the year they're signed, so `yearSigned` is already unambiguous.
When in doubt, state it anyway; it is never wrong to be explicit.

*(This is not hypothetical. The first full real-league file had **every one of its 145
pre-season extensions short by one to three years**, and six players carrying contracts that
had already expired.)*

### The rest of it

Money is in **millions**. The game turns this into a real, cap-valid contract structure
(base + signing bonus + guarantees + proration) indistinguishable from a generated deal.
**Your team must fit under the $280M cap** — an over-cap file is a blocking error with a
per-team breakdown. Players without a contract fact get a market-value deal automatically.

Free agents (`freeAgents[]`, or `players` with `"team": null`) are **contract-less** — omit
`contract`; their asking price is derived at free-agency time like any other FA.

## Career stats (`careerStats`)

Give a player his real statistical past. `careerStats` is an array of **per-season lines**
— one object per season the player has played — that show up on his profile as ordinary
career rows, indistinguishable from seasons the game simulates.

```json
"careerStats": [
  { "season": 2024, "team": "cin", "gamesPlayed": 17,
    "passingYards": 4918, "passingTDs": 43, "interceptions": 9 },
  { "season": 2023, "team": "cin", "gamesPlayed": 10,
    "passingYards": 2309, "passingTDs": 15, "interceptions": 6 }
]
```

- **Per-season lines, not career totals.** The game adds the lines up for you; never state a
  total.
- **Any subset of seasons, any subset of fields.** Omit what you don't know. A bare
  `{ "season": 2022 }` is fine.
- **`season`** is the **calendar year** of that past season. It must be *before* `meta.season`
  (a career line is always in the past).
- **`team`** is the modern franchise id of the team he played for that year (a past team can
  differ from his current one). Omit → his current team.
- **`gamesPlayed`** is the games he appeared in that season.

### Stat vocabulary

The stat field names mirror the game's own season-stat model exactly. State only the ones
that matter for the position:

```
Passing:    passingYards passingTDs interceptions completionPct passAttempts passCompletions
Rushing:    rushingYards rushingTDs
Receiving:  receivingYards receivingTDs receptions
Defense:    tackles sacks forcedFumbles interceptionsDef passDeflections
Kicking:    fgMade fgAttempted fgPct
Punting:    punts puntYards puntsAvg
```

Rates are optional: give `passAttempts` + `passCompletions` (or `fgMade` + `fgAttempted`,
or `punts` + `puntYards`) and the game computes `completionPct` / `fgPct` / `puntsAvg` for
you — or state the rate directly.

### What the report tells you

- **Error (blocks):** a `season` at or after `meta.season` (a career line must be in the
  past); a negative or non-numeric stat.
- **Warning (imports anyway):** a `season` more than 30 years before `meta.season` (still
  displays); a stat that doesn't fit the position (a QB with `tackles`); an unknown stat name
  (typo — ignored); an unknown `team` id (shown as the id).

**`careerStats` is display-only.** Imported stats never touch legacy, Hall-of-Fame credit,
player progression, or AI valuation — they're the player's history, shown on his profile,
nothing more.

> **v1 files stay valid forever.** `careerStats` is an additive field: a file that omits it
> behaves exactly as before, no version bump required.

## Fidelity tiers

Fidelity is detected per player from what's present — mix freely (stars detailed, depth thin):

- **Tier 0** — `name` + `team` + `position` + `ovr`. The minimum.
- **Tier 1** — + `age`, `pot`, contract, draft origin. Career-shaped.
- **Tier 2** — + some `attributes`, `traits`, `college`, `jersey`. Flavor-accurate.
- **Tier 3** — the full attribute set. Editor-grade.

## Filler

You don't have to fill a roster. The game completes every team to a legal shape with
generated depth + camp bodies, each flagged `generated: true`, and the report tells you
exactly how many per team. A five-player team becomes a legal 90-man camp roster.

## TeamFact (optional overrides)

Rename/recolor a team or nudge its market. `id` is required; every override is optional.

```json
{ "id": "cin", "city": "Cincinnati", "nickname": "Bengals", "abbreviation": "CIN",
  "primaryColor": "#FB4F14", "secondaryColor": "#000000",
  "marketSize": "Medium", "fanLoyalty": 72, "mediaIntensity": 58,
  "conference": "CFC", "division": "CFC_NORTH",
  "owner": { "name": "Mike Brown", "personality": "Bottom Line", "patience": 62 } }
```

### Team names (`city`, `nickname`, `name`)

A team's name is two fields that compose:

| Field | Example | Shown |
|---|---|---|
| `city` | `"Cincinnati"` | always |
| `nickname` | `"Bengals"` | only when mascot names are on |

With mascot names **off** (the default) the game shows `Cincinnati`; with them **on** it
shows `Cincinnati Bengals`. Places that mean the *place* — market blurbs, travel lines —
always use the city alone.

⚠️ **`name` is not the mascot.** It predates `nickname` and has always been applied as the
**city**, and `city` overrides it when both are present. It is kept working exactly as it
was so older files keep their meaning, but it is the wrong field for a mascot: in the old
example above, `"name": "Bengals"` alongside `"city": "Cincinnati"` silently discarded
`Bengals`. Use `nickname`.

Nicknames longer than 40 characters warn (they crowd the rows that render them) but still
import.

**Turning them on.** A file that supplies any `nickname` switches mascot names on for that
career automatically. Set `"showMascots": false` in `meta` to ship nicknames without
forcing them on, or `true` to force them on regardless. The player can change it any time
in Settings → Team Names; the choice is saved with the career.

### Owners (`owner`)

Whoever signs the cheques and, eventually, fires you. The owner sets the season goal, drives
your approval rating, and decides when the hot seat gets warm, so who they are matters.

The owner's **reputation** is an archetype plus a fuse:

| `personality` | Reads as | Typical `patience` |
|---|---|---|
| `Win Now` | championship-or-bust | 25–45 |
| `Meddler` | in your draft room | 35–55 |
| `Bottom Line` | revenue first | 45–65 |
| `Loyalist` | sticks with the people in the building | 65–85 |
| `Patient Builder` | trusts the process | 65–85 |
| `Hands Off` | signs the cheques, stays away | 70–90 |

`patience` (0–100) is how much losing the owner tolerates before the firing. Both fields are
optional: give just a `name` and the rest is drawn for you, deterministically. Teams with no
`owner` block keep the owner the game generates for them.

`pronouns` (optional) is how the game's copy refers to the owner ("**he** has the standings
printed out", "**her** approval", "**they** want a word upstairs"):

| `pronouns` | Use for |
|---|---|
| `"he"` | one person who goes by he |
| `"she"` | one person who goes by she |
| `"they"` | a family, a trust, an ownership group, a publicly owned club, or a person who goes by they |

Omitted means `"they"`: the game assumes nothing about an owner you did not describe. Any other
value is a blocking error.

**If the owner is a real person, give only `name` and `pronouns`.** `personality` and
`patience` are a characterization, and a file you share should not put one on someone who
exists. Leave them out and the game draws them, deterministically, for everyone who loads the
file. Use them freely for fictional owners. Owners who are not `"he"` are shown with a team emblem instead of a
drawn face (the procedural face system only draws men), unless a skins package maps them a photo.

```json
{ "id": "ten", "owner": { "name": "Amy Adams Strunk", "pronouns": "she" } }
{ "id": "gb",  "owner": { "name": "Green Bay Packers, Inc.", "pronouns": "they" } }
```

## StaffFact

Coaches and front office. `name` + `team` + `role` are required; omitted fields are sampled.

```json
{ "name": "Hal Devereaux", "team": "cin", "role": "HC", "age": 54,
  "personality": "Innovator", "backstory": "coordinator-prodigy",
  "traits": ["mentor"], "schemePrefs": { "offense": "Air Raid", "defense": "Multiple" },
  "accomplishments": [ { "season": 2019, "championTeamId": "cin" }, { "season": 2015, "award": "COTY" } ],
  "careerRecord": [ { "season": 2019, "team": "cin", "role": "HC", "wins": 15, "losses": 2, "playoffResult": "WON_SB" } ] }
```

- **`role`** — one of the 15: `GM AGM HC OC DC STC QBC RBC WRC OLC DLC LBC DBC SCT TRN`.
  One of each per team.
- **`personality`** — one of the 12: `Motivator, Drill Sergeant, Professor, Scheme Purist,
  Players' Coach, Old School, Innovator, Developer, Competitor, Politician, Stoic, Leader`.
- **`accomplishments`** — championship years for this coach's team, and/or explicit
  **award claims** `{ "season": 2015, "award": "COTY" }` (`COTY` = Coach of the Year,
  `EOTY` = Executive of the Year). Championship claims become league history, place him at
  the champion those years, and price his all-time legacy. An award claim fills that year's
  award slot for him (winning over the champion-derived pick; first claim wins on a tie).
  State three rings, he's a three-ring legend with the tree to match.

### Career record (`careerRecord`)

Give a coach his real season-by-season W/L past. It shows on his profile as a career
ladder (beside the seasons the game fabricates and the ones it simulates), and — for
head coaches, GMs, and coordinators — it **prices his tree depth**: the game replays his
record through its own income model, so a coach who won a lot has a deep tree, exactly as
if he'd earned it in-game.

```json
"careerRecord": [
  { "season": 2023, "team": "kc", "role": "HC", "wins": 11, "losses": 6, "playoffResult": "WON_SB" },
  { "season": 2022, "team": "kc", "role": "HC", "wins": 14, "losses": 3, "playoffResult": "LOST_SB" }
]
```

- **Per-season lines**, one per season. `wins` + `losses` are required; `ties` optional.
- **`season`** is a calendar year before `meta.season`. **`team`/`role`** name where he
  coached that year (omit → his current seat).
- **`playoffResult`** (optional) is one of `MISSED WILDCARD DIVISIONAL CONF LOST_SB WON_SB`.
  A `WON_SB` line makes him the champion that year (consistent with the chronology — it
  even counts as a ring), so you can state a title with just the record.
- **Report:** a season not in the past is an error; outside the 30-year window is a warning
  (still shown); an unusual W+L+T (a season isn't 17 games) warns; a `WON_SB` that
  contradicts a different champion claim for that year is an error (fails loudly).

> **v1 files stay valid forever.** `careerRecord` and `award` are additive — a file that
> omits them behaves exactly as before, no version bump.

## HistoryFact

League championship history — "who won it, and when".

```json
{ "season": 2019, "championTeamId": "cin", "runnerUpTeamId": "buf" }
```

Claims fill their year slots; the game fills the surrounding 30 years around them.
Two claims for the same year? The first wins, with a warning. **Including any history
reshapes the whole league's fabricated past** — teams you didn't author will have
different coaching histories than a default game (the report flags this).

## Legacy records (`legacy`) — the coaches in the Hall

Historic Hall of Fame coaches and executives. They appear on the **League → Legacy** board
(All-Time scope), with their rings, their score, and the HOF badge.

```json
"legacy": [
  {
    "name": "Vince Lombardi",
    "roles": ["HC"],
    "careerYears": [1959, 1969],
    "teams": ["gb", "was"],
    "accomplishments": [
      { "season": 1961, "championTeamId": "gb", "award": "COTY" },
      { "season": 1966, "championTeamId": "gb" },
      { "season": 1967, "championTeamId": "gb" }
    ],
    "hofYear": 1971
  }
]
```

**They are records, not people.** A legacy entry never becomes a coach: no seat, no job, no
carousel, and you can never take one over. He exists to have *already existed*.

- `roles` — `HC` or `GM`, the two the legacy boards track. First is his primary.
- `careerYears` — `[first, last]`. Must end **before** your file's season (he's retired; a
  coach still working belongs in `staff`, not here).
- `teams` — where he coached, in order. A season he claims a title for is a season he was at
  *that* team, whatever the order says.
- `accomplishments` — the same shape as `history`: titles, runner-up finishes, `COTY`/`EOTY`.
- `hofYear` — **the year he was inducted. State it and he is in the Hall, full stop.** The
  game still computes his legacy score (that's what ranks him on the board), but it will not
  argue with you about whether Lombardi is a Hall of Famer.

His W-L record is filled in for you — plausible, strong in his title years. Nobody remembers a
1962 regular-season record; the titles are the facts, and the titles are what you state.

**Claims:** if a legend claims a title or an award that one of your *active* imported coaches
also claims, the active coach wins and the legend's claim is dropped, with a warning telling
you who took it. Historic seasons (outside the game's 30-year fabricated chronology) never
collide with anything — they simply stamp his own record.

See `examples/legacy-legends.json`.

## Hall of Fame players (`hallOfFame`) — the players in the halls

The player mirror of `legacy`. Canton, and the club halls (rings of honor, halls of
honor, honor rolls). They fill **League → Hall of Fame** and **Front Office → Team HOF**,
which are otherwise empty on day one of a save: the game's halls only know the players
its own seasons retired.

```json
"hallOfFame": [
  {
    "name": "Jim Brown",
    "position": "RB",
    "careerYears": [1957, 1965],
    "teams": ["cle"],
    "hofYear": 1971,
    "teamHof": [{ "team": "cle", "year": 2010 }],
    "awards": [
      { "season": 1957, "award": "OROY" },
      { "season": 1958, "award": "MVP" },
      { "season": 1964, "award": "CHAMPION" }
    ]
  }
]
```

**They are records, not people.** A `hallOfFame` entry never becomes a player: no roster
spot, no free agency, no draft, no contract, no coaching career afterward. He exists to
have *already existed*. His id is namespaced (`hofp_import_...`) and can never be
inherited by a generated player.

- `name`, `position`, `careerYears`, `teams` — required. `careerYears` is `[first, last]`
  and must end **before** your file's season (he is retired; a player still playing
  belongs in `players`). `teams` is his clubs in career order, and the League → Hall of
  Fame team filter matches **any** of them.
- `hofYear` — **the year the LEAGUE hall took him. State it and he is in it, full stop.**
- `teamHof` — the **club** halls that took him, `[{ "team": "cle", "year": 2010 }]`. The
  `year` may be omitted when you know the induction but not its year. A club he never
  played for is allowed (honorary inductions happen) but warns, because it is usually a
  typo.
- **He must be in something.** State `hofYear`, or a non-empty `teamHof`, or both. A
  record in neither hall would import and then render nowhere, so it is an error.
- `awards` — the same shape and vocabulary as a player's `awards` (below). They fill his
  plaque and price his legacy score.
- `jersey` — optional.

**Induction is a fact; the score is computed.** The game will not demote a real Hall of
Famer because a 2,200-point scale tuned against a modern 25-season arc prices a 1957-65
career below its bar. It still derives a legacy score from the honors you state, so his
line is comparable to a simulated player's on the same board. A `teamHof` entry works the
same way: it is an induction, not a score that cleared the club bar.

**The two halls are separate inductions.** A man can be in Canton and in three club halls,
in four different years, and each board shows its own. Nothing is inferred: stating
`hofYear` does **not** put him in his old club's hall, because real clubs make that call
themselves.

## Injury history (`injuryHistory`) — a player's real medical past

Give a player his real injury past. `injuryHistory` is an array on a `PlayerFact`, each entry
a single past injury:

```jsonc
"injuryHistory": [
  { "injuryName": "Torn ACL", "part": "Knee (ACL)", "severity": "Season-ending", "season": 2020, "weeksOut": 12 },
  { "injuryName": "High-ankle sprain", "part": "Ankle", "side": "L", "severity": "Moderate", "season": 2022 }
]
```

- `injuryName`, `part`, `severity`, `season` are required. `week`, `weeksOut`, `side` (`L`/`R`),
  `careerEnding` are optional.
- `severity` is one of: `Questionable`, `Minor`, `Moderate`, `Major`, `Severe`, `Season-ending`,
  `Career-ending`.
- `season` is a calendar year and must be **before** your file's season (it's a *past* injury).

**Display-only.** It fills the roster injury panel and nothing else — it never lowers durability,
never touches the sim, and never forces a retirement. (`careerEnding` on an active player is
ignored for exactly that reason.) An omitted `injuryHistory` behaves identically to before.

## Awards (`awards`) — a player's real trophy case

Give a player the honors he actually won. `awards` is an array on a `PlayerFact`, each entry
one honor in one season:

```jsonc
"awards": [
  { "season": 2019, "award": "CHAMPION", "team": "kc" },
  { "season": 2022, "award": "ALL_LEAGUE" },
  { "season": 2022, "award": "ALL_STAR" }
]
```

- `season` and `award` are required; `team` is optional.
- `award` is one of: `MVP`, `OPOY`, `DPOY`, `OROY`, `DROY`, `ALL_LEAGUE` (All-Pro first team),
  `ALL_STAR` (Pro Bowl), `TOP_100`, `CHAMPION` (won the title).
- `season` is a calendar year and must be **before** your file's season, and it is the season
  the honor was won FOR, not the year the trophy was handed out.
- `team` is the club he won it **with**. Omit it and he is credited under his current team, so
  always state it on a ring or an honor he won somewhere else.
- One honor per kind per season. A duplicate is dropped rather than counted twice.

**These count.** Unlike `careerStats` and `injuryHistory`, awards are not display-only: they
fill the player's trophy case, they show their years on hover, and they price into his legacy
score and his Hall of Fame progress exactly like an award the game hands out. A player with a
first-ballot résumé imports with the bar already full. An omitted `awards` behaves identically
to before.

## Traded draft picks (`draftPickTrades`) — who owns which future pick

Teams trade future picks. Record the ones that have already changed hands:

```jsonc
"draftPickTrades": [
  { "season": 2028, "round": 1, "originalTeam": "ne", "currentOwner": "phi" }
]
```

The pick's slot stays with `originalTeam` (its record still earns the selection); `currentOwner`
is who actually makes the pick. `season` is a calendar year and must sit within a few years of
your file's season (the tradeable window); a pick outside it is warned and skipped. A pick you
don't list stays with its original team.

## Conference names (`conferences`) — your league's labels

The game's two conferences are internally `CFC` and `FFC`. Override just their **display** names:

```jsonc
"conferences": {
  "CFC": { "abbr": "AFC", "name": "American Football Conference" },
  "FFC": { "abbr": "NFC", "name": "National Football Conference" }
}
```

Division labels follow automatically (`CFC_EAST` → "AFC East"). This changes only what the UI
shows — standings, brackets, team select, the division-rivals widget. Routing and scheduling are
untouched. Omit the block and everything reads as the shipped `CFC`/`FFC`.

## Coaching tree (`mentor`) — author the real lineage

Give a coach his real mentor and the coaching tree draws itself. Add `mentor` (the exact
NAME of another coach in the file — a `staff` member or a `legacy` legend) to a `staff`
or `legacy` entry:

```jsonc
"staff":  [ { "name": "Andy Reid", "team": "kc", "role": "HC", "mentor": "Mike Holmgren" } ],
"legacy": [ { "name": "Mike Holmgren", "roles": ["HC"], "mentor": "Bill Walsh", … },
            { "name": "Bill Belichick", "roles": ["HC"], "mentor": "Bill Parcells", … } ]
```

An authored `mentor` **overrides** the game's automatic (seniority-based) tree link for that
coach; coaches you don't give a `mentor` are still linked automatically, so you can author as
much or as little of the tree as you like. A `mentor` that doesn't name a coach in the file,
a coach listed as his own mentor, or a loop (A mentors B mentors A) is warned and the bad
link is simply dropped — it never blocks the import.

## Retired temperament traits (PERS C5)

Three names that used to be valid `traits` entries are now **personality axes** rather than
traits:

| Retired trait | Now expressed as |
|---|---|
| `Hot Head` | Emotional Stability at its low pole |
| `Natural Leader` | Leadership at its high pole |
| `Locker Room Cancer` | Teamwork at its low pole |

**Files using them still load.** On import (and on loading any save taken from such an
import) each name is folded into the matching axis and stripped from `traits`. The
absorption is applied as a bound rather than an assignment, so it never pulls an axis back
from a more extreme value, and it is idempotent.

Prefer leaving these out of new files. Authoring the axes directly is a v2 format want;
until then a player's personality is generated deterministically from his id.

## Tendencies (`tendencies`) — how he plays, not how good he is

Every player and prospect carries a set of **behavior magnitudes** for his position group:
how often he does one named thing on the field, on a 1 to 5 scale (Rarely, Seldom,
Sometimes, Often, Constantly). Level 3 is typical for the position. **No level is better**:
"Takes Off: Constantly" describes a quarterback who scrambles at the first crack of pressure,
and whether that is good depends on the scheme. Quality lives in `attributes`.

Add a `tendencies` object to a player or a prospect. Any subset of his group's ids; integers
1 to 5. Absent ids are generated from his attributes and build, so a file can author one
tendency or all of them.

```jsonc
{ "name": "Lamar Jackson", "team": "bal", "position": "QB", "ovr": 98,
  "tendencies": { "escapeInstinct": 5, "quickTrigger": 3, "readProgression": 3,
                  "downfieldAppetite": 4, "keepOnRead": 5, "offScript": 5 } }
```

| Group (positions) | Ids |
|---|---|
| qb (QB) | `escapeInstinct` takes off · `quickTrigger` gets the ball out · `readProgression` works the progression · `downfieldAppetite` pushes it downfield · `keepOnRead` keeps it on reads · `offScript` plays off script |
| rb (RB, FB) | `contactSeeking` · `bounceTendency` bounces outside · `receivingAppetite` releases into routes · `homeRunHunting` goes for the house · `passProtectWillingness` stays in to block · `workloadAppetite` wants the load |
| wr (WR, TE) | `verticalBias` wins downfield · `contestedAppetite` attacks the catch point · `yacHunting` fights for yards · `blockingAppetite` · `freelancing` · `ballDemand` |
| ol (OT, OG, C) | `powerBias` drives and displaces · `aggressiveSet` jump sets · `finishAggression` through the whistle · `pullAppetite` · `cutBlockUsage` |
| dl (EDGE, DT) | `speedRushBias` around the edge · `passFirstRead` pins his ears back · `gapShooting` · `insideCounterUsage` · `stuntAppetite` |
| lb (MLB, OLB) | `triggerSpeed` fires downhill · `coverageBias` drops into coverage · `blitzAppetite` · `bigHitSeeking` · `roaming` |
| db (CB, FS, SS) | `pressBias` presses at the line · `ballHawking` jumps routes · `runSupportAppetite` · `downfieldContact` gets handsy · `backfieldPeeking` |
| st (K, P) | `kickoffDepthBias` kicks for depth / hangs it · `directionalBias` angles it · `placementOverPower` places it · `rangeAggression` wants the long try |

Author the behavior you have watched, not the grade you would give him. An id from another
group, a non-integer, or a value outside 1 to 5 is warned (`unknown-tendency`,
`tendency-wrong-group`, `bad-tendency-level`) and dropped; the rest of the object is kept.
Tendencies never touch `ovr` or the attributes, and a file without them loads exactly as
before.
