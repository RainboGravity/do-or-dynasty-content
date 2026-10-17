# PickSix — Skins Package Format (v1)

A **skins package** decorates an imported roster with profile pictures — players, and
optionally coaches (`staff`) and owners (`owners`). It is a plain `.zip` containing a
manifest and WebP images. Faces show on player and staff profiles and the org chart;
anyone without a face keeps the standard text-only look. The format is additive by
design: `staff` and `owners` were added this way (an older game build warns about the
unknown sections and still loads the player faces), and `teams` (logos) was added the same
way, and future versions may add more art without breaking existing packages.

Load points:
- **New Game** — after a roster import is accepted, the flow offers a skins step.
- **Settings → Skins Package** — load, replace, or remove a package on an existing save.

The resolved face mapping is part of the save file; the images live on disk under the
game's app-data `skins/` directory. A save opened on a machine without the package files
simply shows the standard look until the package is re-loaded (same file → same result).

## Archive layout

```
my-skins.zip
├── manifest.json          (required, at the archive root)
├── faces/
│   ├── joe-burrow-cin-qb.webp
│   └── …
├── staff/                 (optional — coach/GM faces)
│   └── zac-taylor-cin-hc.webp
└── owners/                (optional — owner faces)
    └── mike-brown-cin.webp
```

Image paths are manifest-declared; the folder names are a convention, not a rule.
Only files the manifest references are ever installed — anything else in the archive is
ignored (reported as info).

## manifest.json

```jsonc
{
  "formatVersion": 1,
  "kind": "picksix-skins",
  "meta": {
    "title": "My League 2026 Faces",
    "author": "you",
    "forRoster": "My League 2026",  // free-text label of the roster this was authored for
    "notes": "optional"
  },
  "players": [
    { "name": "Joe Burrow", "team": "cin", "position": "QB", "face": "faces/joe-burrow-cin-qb.webp" }
  ],
  "staff": [                            // optional — coach/GM/scout faces
    { "name": "Zac Taylor", "team": "cin", "role": "HC", "face": "staff/zac-taylor-cin-hc.webp" }
  ],
  "owners": [                           // optional — owner faces
    { "name": "Mike Brown", "team": "cin", "face": "owners/mike-brown-cin.webp" }
  ]
}
```

| Field | Required | Notes |
|---|---|---|
| `formatVersion` | yes | Currently `1`. Newer versions are refused by older game builds. |
| `kind` | yes | Must be exactly `"picksix-skins"` — the discriminator that stops a roster zip (or any other archive) from being misread. |
| `meta` | no | Free-text provenance, shown in the report and in Settings. |
| `players[].name` | yes | Full display name, exactly as the roster file spells it. |
| `players[].team` | no | Roster-format lowercase abbreviation (`cin`, `kc`, …). Strongly recommended. |
| `players[].position` | no | Game position (`QB`, `EDGE`, `OT`, `MLB`, …). Strongly recommended. |
| `players[].face` | yes | Archive-relative image path. No `..`, no leading `/`, no drive letters, forward slashes only — violations are hard errors. |
| `staff[].name` | yes | Full display name, exactly as the roster file spells it. |
| `staff[].team` | no | Roster-format lowercase abbreviation. Strongly recommended. |
| `staff[].role` | no | Staff role id (`HC`, `OC`, `DC`, `STC`, `GM`, `AGM`, `QBC`, `RBC`, `WRC`, `OLC`, `DLC`, `LBC`, `DBC`, `SCT`, `TRN`). Strongly recommended. |
| `staff[].face` | yes | Archive-relative image path, same rules as player faces. |
| `owners[].name` | yes | Full display name — a **cross-check**, see below. |
| `owners[].team` | **yes** | The owner's identity: one owner per team. A missing team is a hard error. |
| `owners[].face` | yes | Archive-relative image path, same rules as player faces. |

There are **no ids** in the format, on purpose — the roster format has none either.
Identity is the author-visible triple (name, team, position/role), exactly like the
roster file itself. Owners are the exception in shape only: their identity IS the team.

## How matching works

Each entry is resolved against the loaded world's players in three narrowing passes:

1. **name + team + position** — the exact match.
2. **name + position** — catches players who changed teams after the package was made.
3. **name alone** — only when the name is unique in the whole player pool.

Names are normalized on both sides before comparison: case-insensitive, diacritics
stripped, apostrophes and periods removed (`Ja'Marr` = `JaMarr`, `D.J.` = `DJ`),
hyphens treated as spaces (`Amon-Ra` = `Amon Ra`), and generational suffixes (`Jr.`,
`III`, …) dropped.

**Ambiguity is never guessed.** If a pass finds two candidates (two Josh Allens), the
entry is reported unmatched with reason `ambiguous` — fix it by adding `team` and
`position` to the entry. The first entry to claim a player wins; later duplicates are
reported. Unmatched entries are harmless: those players just keep the standard look.

Free agents match too (passes 2–3; they have no team). Generated filler players have no
entries and correctly stay text-only — no likeness exists.

**Draft prospects match too**, which is how a custom draft class gets faces — see
"Draft-class packs" below.

**Staff match the same way**, with `role` in place of `position`: name+team+role, then
name+role (a coach the carousel moved lands here — his face follows him), then unique
name. Coaches sitting in the unemployed pool match like free agents. Staff hired or
generated in later seasons have no entries and stay text-only, like draft classes.

**Legacy legends match through the `staff` section too.** Retired records — imported
historic legends (roster spec §13e) and coaches who retire during play — join the
staff candidate pool anchored at their **final** team (a legend's last team in the
roster file's `legacy[].teams` list) with their primary role. Author a legend entry
as `{ "name": "Vince Lombardi", "team": "was", "role": "HC", … }` and his face shows
on the Legacy board profile. A legend with no team resolves by unique name.

**Hall of Fame players match through the `players` section.** §13m hall records
(roster spec) join the player candidate pool anchored at their **first** team — the club
an author lists first, which is the one a hall of famer is identified with (Rice is a
49er, not a Seahawk). Author the entry as
`{ "name": "Jim Brown", "team": "cle", "position": "RB", … }` and his portrait shows on
the League → Hall of Fame and Front Office → Team HOF boards. Note the asymmetry with
legacy coaches, who anchor at their **last** team: a coach's story ends where he
finished, a player's is where he starred. Passes 2 and 3 catch a differently-anchored
entry either way.

**Owners resolve by team**, since each team has exactly one: the entry's `team` picks
the owner, and the `name` must then match the in-game owner (same normalization). A
mismatch means the world's owner is somebody else — the face is reported unmatched
(`name-mismatch`), never applied as a wrong likeness.

Coach and owner faces are only meaningful alongside a roster that **imports** its staff
(`staff` facts, spec §13d) and owners (`teams[].owner`, spec §13f) — a generated world
draws random names no author can anticipate. The published real-league roster imports both.

## Draft-class packs

A pack can give faces to the prospects in a **custom draft class**
(see [draft-class-format.md](draft-class-format.md)). Author the entries in
the normal `players` section, but **omit `team`** — a prospect isn't on a roster yet:

```jsonc
"players": [
  { "name": "Arch Manning", "position": "QB", "face": "faces/arch-manning-qb.webp" }
]
```

They resolve on pass 2 (name + position) or pass 3 (unique name), the same route free
agents take. Include `position` — a class has ~342 prospects and it's what keeps two
similar names apart.

**The face follows him after the draft.** A prospect keeps his internal id when he's
drafted, so once a face is claimed it stays with him for his whole career — no second
pack, no re-match.

Load points for a class pack:
- **New Game** — the skins step matches against the class you loaded at the draft-class
  step (and any future years queued from the same file).
- **Settings → Skins Package** — the route for a class imported mid-career: import the
  class first, then load its pack.

⚠️ **Coverage is inherently low for a class you intend to share.** Prospects are college
athletes whose team and press photos are copyrighted, so the only images a shared pack can
carry are ones you have the rights to (see **Sources and rights** below). Anyone without a
face keeps the game's generated face, which is the designed fallback.

## Images

- **Format: WebP** (the packaged file is rendered as-is; WebP keeps a 1,200-face
  package around 10–20 MB).
- **Size: 256×256** recommended, cover-cropped from the top (headshots keep the face).
- The profile modal renders faces at 64px; anything ≥128px square looks fine.

## Team logos (`teams`)

An optional section mapping team ids to logo images.

```jsonc
"teams": [
  { "team": "cin", "logo": "logos/cin.webp" }
]
```

Unlike a player, a team needs no matching — the id **is** the identity, so there is no
name to cross-check and nothing to guess. An entry for a team the league does not have is
dropped with a warning; the rest of the package still installs.

- **Format: WebP with transparency**, since logos are composited over turf, photos and
  panels rather than sitting in a box.
- **Size: 512×512** recommended. The largest use is the midfield mark in the play-by-play
  field, which scales to the width of the field.
- **One entry per team.** A duplicate is reported and the first wins.

Where they show up: painted at midfield in the play-by-play field, in the matchup intel
header, on the team-select screen, and as a faint watermark behind the game. Anywhere a
team has no logo simply keeps the colored mark it has today.

### Building a logo pack

A logo pack is a zip with a `manifest.json` carrying only a `teams` section and one
image per team. Any zip tool builds one. Keep images at 512×512 with transparency,
letterboxed rather than cropped (a logo cropped square loses the edges of the mark).

A logo pack **stacks** with a face pack — load both; they cover different subjects.

**Logo packs load from your own disk only** — **New Game → skins step → Add .zip file**
or **Settings → Skins Package**. The content catalog refuses to install a downloaded
package that carries a `teams` section, and PickSix does not list, host or distribute
logo packs. Team marks are the part of a league's identity most likely to be a registered
trademark, and a pack you build for your own machine is your act in a way a listed
download is not. If you want a league to have marks without that question, draw your
own: a plain roundel in each team's colors with its abbreviation is what the game shows
by default, and original art is yours to share.

## Home fields (`fields`)

An optional section mapping team ids to a home-field image, painted as the turf in the
play-by-play field for that team's home games.

```jsonc
"fields": [
  { "team": "cin", "image": "fields/cin.webp", "left": 0.02, "right": 0.98 }
]
```

The image is top-down and horizontal, end zone to end zone. `left` and `right` say where
the back line of each end zone sits as a fraction of the image width (defaults `0` and
`1`, meaning the image starts and ends exactly at the back lines); `top` and `bottom`
trim vertically the same way. The same rules as logos otherwise: the id is the identity,
one entry per team, an unknown team is dropped with a warning. WebP, around 2400×1000. A
photograph of a real stadium is the photographer's copyright; a field you paint or render
yourself is yours.

## Sources and rights

A pack you load on your own machine is your business. A pack you **share** is a
publication, and every image in it needs to be one you are allowed to publish. The safe
sources, in order:

1. **Images you made.** Your own art, renders, drawings or photographs. Always fine.
2. **Public-domain images.** Fine, with a note in `meta.notes` saying where they came from.
3. **Wikimedia Commons images with a permissive license** (CC0, CC BY, CC BY-SA). Check the
   license on each file's own Commons page — a Wikipedia article's lead image is sometimes a
   non-free file used under fair use, and those may **not** be redistributed. Keep an
   attribution table (person, source page, author, license) in the zip as `ATTRIBUTION.md`
   and summarize it in `meta.notes`. CC BY-SA is share-alike: say so on the pack.

What a shared pack must **not** carry: headshots pulled from a sports site's or a league's
image servers, team or college press photos, broadcast stills, or agency photographs. Those
are owned by whoever shot them, and no site's public URL makes them yours to republish.
The same goes for real team marks, for the reason given under logos.

Coverage will be partial by design. The game draws a face for anyone a pack does not
cover, so a pack that gives faces to a hundred coaches and owners and nobody else is a
perfectly good pack.

## Building a pack

The manifest is plain JSON and the images are WebP, so any scripting language and any zip
tool will do. The one rule that matters: spell each `name` **exactly as the roster file
does**, and include `team` and `position` (or `role`) so matching never has to guess. The
game's own validator runs on load and the report names every entry it could not match.
