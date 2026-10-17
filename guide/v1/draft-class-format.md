# Draft-Class Format Guide (formatVersion 1)

Custom **draft classes** let you author the prospects for one or more upcoming drafts —
the real 2027 class, a fantasy class, a themed pack. A class can be loaded three ways:

1. **At New Game — its own step.** After the roster-import and skins steps you get a
   **"Load a Draft Class"** screen with its own file picker. This is the normal route: a
   class file is a *separate artifact* from a roster file, and this step appears whether or
   not you imported a roster. Skip it and classes generate as usual.
2. **At New Game — embedded in your roster file.** If your roster `.json` also carries a
   `draftClasses` block, those load too. Anything picked at the step is added on top (and
   wins on a year collision).
3. **During a career**, from the Scouting ▸ Draft Class tab's **LOAD CLASS** button
   (drop in next year's class after a draft concludes).

All three use the same schema below. A standalone class file needs nothing but
`formatVersion` + `draftClasses`.

## Shape

Add a `draftClasses` array to the top-level import file. Each entry is one draft **year**:

```jsonc
{
  "formatVersion": 1,
  "meta": { "title": "2027 Draft Class", "author": "you", "season": 2026 },
  "draftClasses": [
    {
      "year": 2027,
      "meta": { "title": "Consensus big board v1", "notes": "..." },
      "prospects": [ /* ProspectFact[] */ ]
    }
  ]
}
```

A **standalone class file** (for in-career loading) is just a file whose only content is
`draftClasses` — no teams/players/staff needed.

### `year` — which draft this class feeds

A calendar year. The game's first season is 2026 (season index 1), and a new game starts
**after** the 2026 draft, so the first draft you'll actually run is **2027**. Author
`"year": 2027` for the upcoming class, `2028` for the year after, and so on. A class for a
draft that has already happened is ignored (with a warning).

### `prospects` — the players

You do **not** need to author a full class. Author the prospects you care about (the top
names) and the game **fills the rest** with generated prospects to a complete, realistic
class shape (~342 players across all positions). Authored + generated together form the
class you scout and draft.

## ProspectFact fields

| Field | Req? | Notes |
|---|---|---|
| `name` | ✅ | Full name, "First Last". |
| `position` | ✅ | One of the game position codes (below). |
| `ovr` | ✅ | **Current** ability, 1–99 — a rookie's ability the day he's drafted (see scaling). |
| `pot` | | Ceiling, 1–99. Omit → derived from `ovr` + age. Always clamped ≥ `ovr`. |
| `age` | | Draft-day age (20–24; default ~21–23). |
| `jersey` | | 0–99. |
| `college` | | School name. |
| `collegeSize` | | `"Large"` \| `"Medium"` \| `"Small"`. |
| `attributes` | | Any subset of the attribute vocabulary; honored as hard constraints. |
| `traits` | | Array of trait strings. |
| `physical` | | `{ heightIn, weightLbs, wingspanIn, handSizeIn }` — any subset. |
| `prdPositional` | | 1–99 college-production signal (how productive he was vs his position). Omit → derived. |
| `projectedRound` | | 1–7. **Advisory only** — helps you organize; it does NOT reveal his draft stock in-game (you still scout to learn it). |
| `storyline` | | Backstory + media hooks (below). |

**No `team`, no `contract`, no `draft` record** — a prospect predates all three.

### Position codes

```
QB RB FB WR TE   OT OG C   EDGE DT MLB OLB   CB FS SS   K P
```

Map real positions: edge rusher/DE → `EDGE`, defensive tackle/NT → `DT`, off-ball
linebacker → `MLB`/`OLB`, strong safety → `SS`, free/deep safety → `FS`, guard → `OG`,
center → `C`, tackle → `OT`, all corners (incl. slot/nickel) → `CB`.

### Rating scaling — the one thing to get right

`ovr` is the prospect's ability **now**, as a rookie — not his ceiling. Rookies enter low
and grow. `pot` is the ceiling. Suggested bands:

| Prospect | `ovr` (current) | `pot` (ceiling) |
|---|---|---|
| Generational top-5 talent | 74–79 | 92–97 |
| First-round | 70–76 | 85–92 |
| Day two | 66–72 | 78–86 |
| Day three | 60–68 | 70–80 |

**Imported prospects are fully fogged.** The file states the truth, but in-game every
prospect starts unscouted — no OVR grade, no POT grade, nothing revealed. You scout them
exactly like generated prospects. (A file can't hand the player a pre-scouted board.)

## Storyline (for the news/media system)

An optional `storyline` block gives a prospect a backstory and machine-readable hooks the
game's news ticker / media system can use. It travels with the player — if he's drafted,
his storyline stays with him.

```jsonc
"storyline": {
  "blurb": "A three-year starter who returned from a torn ACL to lead his conference in sacks.",
  "headline": "The comeback nobody saw coming",
  "tags": ["injury-comeback", "character-questions"],
  "weight": 4
}
```

- `blurb` — free-text backstory prose.
- `headline` — a short ticker-ready line.
- `tags` — machine hooks (freeform strings), e.g. `"draft-day-slider"`, `"injury-comeback"`,
  `"small-school-riser"`, `"legacy-bloodline"`, `"character-questions"`.
- `weight` — 1–5 prominence (higher = the media leans on it harder).

All fields optional. Omit `storyline` entirely if the prospect has none.

## Validation

- `name`/`position`/`ovr` missing → error. `pot < ovr` → error. Unknown position → error.
- Two classes for the same `year` → error.
- `year` in the past → warning (loads, but that draft is unreachable).
- Storyline / `projectedRound` problems → warnings (never block).

See **validation-glossary.md** for every code.

## Authoring at scale

You don't have to hand-write 150 prospects. The reliable method mirrors the roster guide's
one-team-per-pass approach — here, **one position group per pass** — and the
[authoring guide](authoring-guide.md) has the exact prompt to hand an assistant for it.
Run one pass per position group, concatenate the arrays into one `prospects` array inside
a single `draftClasses` entry, and load it. You only need the notable names — the game
fills the rest of the class, and the **report screen tells you** how many it filled and
flags any problems.

