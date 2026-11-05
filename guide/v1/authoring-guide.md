# Make Your Own: rosters, draft classes, faces, logos and fields

Do or Dynasty ships with a fictional league. Everything else is a file you make: a roster for
any league and any season, a draft class, a pack of faces, team marks or home fields. This
guide is the short road. The format references next to it have every field.

You do not need to write JSON by hand. The prompts below describe each file format
exactly as this version of the game validates it. Paste one into any assistant, describe
what you want on the line after it, and load what comes back. The same prompts are in the game under **New Game →
Make Your Own → Copy**, which is the copy to use if you are unsure which version of this
page matches your build.

## What you can make, and where it loads

| File | What it is | Loads at |
|---|---|---|
| Roster `.json` | The league: teams, players, staff, owners, history | New Game → Roster → Load .json file |
| Draft class `.json` | One or more future draft classes | New Game → Draft Classes, or Scouting → Draft Class during a career |
| Skins `.zip` | Faces for players, prospects, coaches and owners | New Game → Faces, Logos and Fields, or Settings → Skins Package |
| Logo pack `.zip` | A team mark per slot | Same places as skins, **from your own disk only** |
| Field pack `.zip` | A home-field image per slot | Same places as skins |

Every load shows a report. Errors block and say exactly what to fix; warnings explain what
the game filled in. The [validation glossary](validation-glossary.md) decodes each one.

## The three rules that make any league fit

1. **The game has 32 slots, and the ids are fixed.** Any league with up to 32 franchises
   maps onto them. You rename and recolor a slot in the roster's `teams` block; you cannot
   add a 33rd or delete one. The ids:

   ```
   buf mia ne  nyj   bal cle pit cin   hou ind jax ten   den kc  lv  lac
   dal nyg phi was   chi det gb  min   atl car no  tb    ari lar sf  sea
   ```

2. **A franchise keeps one id through moves and renames.** Use the id of the city the
   franchise plays in today for every era, and put the era's city, nickname and colors in
   the `teams` override. A 1982 season has 28 franchises: author those 28 onto their modern
   ids, leave the other four unauthored (the game generates those clubs and the report
   says so) or name them as fictional expansion teams.

3. **Money is optional.** The game enforces a modern salary cap in every season. Authored
   contracts must fit under it; a player with no contract gets a cap-valid one generated.
   For a historical season, omit contracts entirely. For a current one, give deals to the
   ten or so players whose contracts define the team and leave the rest.

## Building a roster with an assistant

A full league is around 1,900 players. Asking for all of it in one answer is how you get
invented players. The reliable method is **one team per pass**: 32 small answers, then a
trivial merge. Each pass is small enough that the assistant gets the details right.

### Step 1: the prompt, one team at a time

Copy the block below verbatim. It ends by asking for a description, so on the next line
write the team you want, then run it. Do this once per team and save each answer as
`<teamid>.json`. A good description says:

| Say | For example |
|---|---|
| The league | A real league, a historical one, or one you invented |
| The year | The calendar year the file's first season is played in |
| The team and its slot id | The franchise as it was known that season, and its id from the list above |
| Anything else you want | Opening-day roster, end-of-season roster, an all-rookie depth chart |

<!-- prompt:roster -->
```text
You are building a roster file for the football management game Do or Dynasty. Output ONLY a JSON object, no prose, no markdown fences.

Rules:
- Output shape: { "players": [ ... ], "staff": [ ... ], "teams": [ ... ] }. Build ONE team per pass, the team described at the bottom; I merge the files myself.
- The game has exactly 32 team slots with these ids: buf mia ne nyj bal cle pit cin hou ind jax ten den kc lv lac dal nyg phi was chi det gb min atl car no tb ari lar sf sea
  A franchise keeps ONE id through relocations and renames (use the id of the city it plays in today). "teams" holds a single override for this team's slot: { "id", "city", "nickname", "abbreviation", "primaryColor", "secondaryColor" } with the names and colors this franchise used that season. Never invent an id.
- Each player: { "name", "team" (the slot id), "position", "ovr", and optionally "age", "jersey", "pot", "college", "draft", "contract", "careerStats", "awards" }.
- "position" is one of: QB RB FB WR TE OT OG C EDGE DT MLB OLB CB FS SS K P
  Map real positions: edge rusher/DE -> EDGE, defensive tackle/NT -> DT, off-ball linebacker -> MLB or OLB, strong safety -> SS, free safety -> FS, guard -> OG, center -> C, tackle -> OT, every corner -> CB.
- "ovr" is 0-100 (stars ~88-99, starters ~78-87, depth ~62-77). Rate from how the player actually played that season; do not copy ratings from any other game or product.
- "draft": { "year", "round", "pick" } when known.
- "contract" (millions): { "years", "totalM", "guaranteedM", "yearSigned", "throughSeason" }.
  The game enforces a $280M hard cap for every season, so keep this team's total average annual value under $252M. Give deals ONLY to the ~10 best players and leave the rest with no contract; the game generates cap-valid deals for everyone without one. For a historical season, omit "contract" on everyone.
  If the deal was signed BEFORE the season described, you MUST include "throughSeason": the calendar year of the deal's FINAL season. An extension's advertised length counts only new money, which starts the year AFTER signing, so "years" + "yearSigned" alone places an extension 1-3 seasons early. Keep both truthful and add "throughSeason". You may omit it only for a rookie deal or a signing made in that season itself.
- "careerStats" (OPTIONAL, for stars and starters): one object per PAST season, each with "season" (a calendar year strictly before the season described), "team" (the id he played for that year), "gamesPlayed", and only the fields that fit his position, using these exact names:
    QB:  passingYards passingTDs interceptions completionPct passAttempts passCompletions
    run: rushingYards rushingTDs
    rec: receivingYards receivingTDs receptions
    def: tackles sacks forcedFumbles interceptionsDef passDeflections
    K:   fgMade fgAttempted fgPct
    P:   punts puntYards puntsAvg
  Per-season lines only, never career totals. Omit careerStats for depth players.
- "awards" (OPTIONAL): one object per honor per season, { "season", "award", "team"? }, where "award" is one of: MVP OPOY DPOY OROY DROY ALL_LEAGUE ALL_STAR TOP_100 CHAMPION
  (ALL_LEAGUE = first-team all-league, ALL_STAR = the all-star game, CHAMPION = won the title that season.) "season" is the year the honor was won FOR and is strictly before the season described. List ONLY honors you are certain of: these count toward the Hall of Fame.
- "staff": the head coach, coordinators and general manager as { "name", "team" (the slot id), "role" } with "role" one of: GM AGM HC OC DC STC QBC RBC WRC OLC DLC LBC DBC SCT TRN
  Add "careerRecord" for the head coach when you know it: one { "season", "team", "role", "wins", "losses", "playoffResult" } per season coached.
- Include 45-53 players across all position groups (at least 2 QB, 2 RB, 5 WR, 2 TE, 6 OL across OT/OG/C, 6 DL across EDGE/DT, 4 LB across MLB/OLB, 5 DB across CB/FS/SS, 1 K, 1 P). The game fills any gaps.
- State facts a fan knows. Do not invent attribute vectors, and omit any field you are not sure of rather than guessing.

The user will describe the type of roster they want below and should include any relevant league name, the year, and any other traits desired for the roster:
```
<!-- /prompt:roster -->

### Step 2: merge the files

Put all the per-team files in one folder and run this. It concatenates every file's arrays
and adds the required top-level fields. Save it as `merge.mjs`, then run
`node merge.mjs ./teams-folder my-league.json`.

```js
// merge.mjs: concatenate per-team roster files into one importable file.
import { readdirSync, readFileSync, writeFileSync } from "node:fs";
import { join } from "node:path";

const [, , dir = ".", out = "my-league.json"] = process.argv;
const merged = {
  formatVersion: 1,
  meta: { title: "My League", author: "Me", season: 2026 },
  teams: [], players: [], freeAgents: [], staff: [], history: [],
};

for (const file of readdirSync(dir).filter((f) => f.endsWith(".json") && f !== out)) {
  const part = JSON.parse(readFileSync(join(dir, file), "utf8"));
  for (const key of ["teams", "players", "freeAgents", "staff", "history"]) {
    if (Array.isArray(part[key])) merged[key].push(...part[key]);
  }
}

writeFileSync(out, JSON.stringify(merged, null, 2));
console.log(`Wrote ${out}: ${merged.players.length} players, ${merged.staff.length} staff, ${merged.teams.length} team overrides.`);
```

Set `meta.season` to the same year you described, and `meta.title` and
`meta.author` to what you want shown on the load screen.

### Step 3: load it and iterate against the report

**New Game → Roster → Load .json file.** The report lists every problem at once. Paste the
error lines back to the assistant with "fix these", re-merge, reload. When it is clean,
start your career. The exact bytes of a finished file reproduce the exact league for anyone
you share it with.

Things worth adding once the basics load, all described in the
[roster format reference](roster-format.md): league championship history, a legendary
coach's accomplishments and career record, hall-of-fame players, coaching trees, traded
draft picks, conference labels and per-player tendencies.

## Building a draft class with an assistant

Same idea, **one position group per pass**. Under the prompt, describe the class you want:
the league, the draft year, the position group, and anything else (only the first round, a
deep quarterback class). Run it once each for QB, RB, WR, TE, OL (OT/OG/C), EDGE, DT, LB
(MLB/OLB), CB and S (FS/SS), concatenate the arrays into one `prospects` array, and wrap it:

```json
{
  "formatVersion": 1,
  "draftClasses": [
    { "year": 2027, "prospects": [ /* the concatenated arrays */ ] }
  ]
}
```

The first draft a new career runs is the year after `meta.season`, so a 2026 roster's
first custom class is `"year": 2027`. Only the notable names are needed; the game fills
the rest of the class. Prospects start fully unscouted whatever the file says, so a file
cannot hand you a pre-scouted board.

<!-- prompt:draft-class -->
```text
You are building a draft-class file for the football management game Do or Dynasty. Output ONLY a JSON array, no prose, no markdown fences.

Rules:
- Output shape: a JSON array of prospect objects. When the description at the bottom names a position group, cover only that group; I concatenate the arrays myself.
- Each prospect: { "name", "position", "ovr", "pot", and optionally "age", "jersey", "college", "collegeSize", "projectedRound", "storyline" }. No "team", no "contract", no "draft": a prospect predates all three.
- "position" is one of: QB RB FB WR TE OT OG C EDGE DT MLB OLB CB FS SS K P
  Map real positions: edge rusher/DE -> EDGE, defensive tackle/NT -> DT, off-ball linebacker -> MLB or OLB, strong safety -> SS, free safety -> FS, guard -> OG, center -> C, tackle -> OT, every corner -> CB.
- "ovr" is the prospect's ability NOW, as a rookie, and "pot" is his ceiling; "pot" must be >= "ovr". Use these bands:
    generational top-5 talent: ovr 74-79, pot 92-97
    first round:               ovr 70-76, pot 85-92
    day two:                   ovr 66-72, pot 78-86
    day three:                 ovr 60-68, pot 70-80
  Rate from the prospect's own college play and scouting consensus; do not copy ratings from any other game or product.
- "age" is draft-day age (20-24). "collegeSize" is "Large", "Medium" or "Small". "projectedRound" is 1-7 and only helps me organize; the game hides it.
- "storyline" (OPTIONAL, for the notable ones): { "blurb": one sentence of backstory, "headline": a short ticker line, "tags": [free-form strings], "weight": 1-5 }.
- Only the notable names are needed; the game fills the rest of the class with generated prospects. Omit any field you are not sure of rather than guessing.

The user will describe the type of draft class they want below and should include any relevant league name, the year, and any other traits desired for the draft class:
```
<!-- /prompt:draft-class -->

## Faces, logos and fields

A face, logo or field pack is a zip with a `manifest.json` at its root and WebP images
beside it. The [skins format reference](skins-format.md) has every field, the matching
rules and the image sizes. Three things to know before you start:

- **Names must match the roster file exactly.** The manifest names people the way the
  roster does, with `team` and `position` (or `role`) so matching never guesses. The load
  report lists every entry it could not match.
- **Coverage can be partial.** Anyone without a face keeps the game's drawn one; a team
  without a logo keeps its colored roundel; a team without a field plays on the standard
  turf. A pack that covers a hundred coaches and owners and nobody else is a good pack.
- **A pack you share is a publication.** Every image in it has to be one you may publish:
  your own art, public-domain images, or Commons images with a permissive license and an
  attribution table. Not headshots from a sports site, not team press photos, not real
  team marks. The skins reference's **Sources and rights** section says exactly where the
  line is. A pack you load only on your own machine is your own business.

### How the art prompts work

An assistant writes the **manifest**; **you supply the images**. Each prompt below
returns one `manifest.json`. Under the prompt, describe the pack: the league and year, the
people or teams to cover, and the names of the image files you have (or say you will name
them the way the prompt suggests). Then:

1. Save the answer as `manifest.json`.
2. Put your images in the folders the manifest points at (`faces/`, `staff/`, `owners/`,
   `logos/` or `fields/`), with the same file names.
3. Zip `manifest.json` and those folders together, with `manifest.json` at the top level
   of the zip, not inside a folder.
4. Load the zip under **New Game → Faces, Logos and Fields → Add .zip file**, or
   **Settings → Skins Package** for a career already running. The report lists anything
   that did not match; paste those lines back to the assistant with "fix these".

Faces, logos and fields can be separate packs or one zip with several sections. Separate
packs stack.

### Faces

Faces cover players, coaches and owners, and prospects in a custom draft class (leave
`team` off a prospect). Paste the names from your roster file into your description so the
assistant spells them the way the roster does. Square WebP, 256×256.

<!-- prompt:faces -->
```text
You are building the manifest for a faces pack for the football management game Do or Dynasty. Output ONLY a JSON object (the file manifest.json), no prose, no markdown fences.

Rules:
- Output shape: { "formatVersion": 1, "kind": "do-or-dynasty-skins", "meta": { "title", "author", "forRoster", "notes" }, "players": [ ... ], "staff": [ ... ], "owners": [ ... ] }. "players" is always present, even as an empty array; leave out "staff" or "owners" when they have no entries.
- The game has exactly 32 team slots with these ids: buf mia ne nyj bal cle pit cin hou ind jax ten den kc lv lac dal nyg phi was chi det gb min atl car no tb ari lar sf sea
  A franchise keeps ONE id through relocations and renames (use the id of the city it plays in today). Never invent an id.
- Each player: { "name", "team" (the slot id), "position", "face" }. Spell "name" EXACTLY as the roster file does; matching depends on it. Omit "team" for a draft prospect or a free agent.
- "position" is one of: QB RB FB WR TE OT OG C EDGE DT MLB OLB CB FS SS K P
- Each staff member: { "name", "team" (the slot id), "role", "face" } with "role" one of: GM AGM HC OC DC STC QBC RBC WRC OLC DLC LBC DBC SCT TRN
- Each owner: { "name", "team" (the slot id), "face" }. "team" is required: an owner is identified by the team.
- "face" paths: faces/<name>-<team>-<position>.webp for players, staff/<name>-<team>-<role>.webp for staff, owners/<name>-<team>.webp for owners. Paths are inside the zip: forward slashes, lowercase, hyphens for spaces, no leading "/", no "..". Use the file names the user gives when they give them.
- Images are WebP, square, 256x256 recommended, cropped from the top so the face stays in frame.
- List only people the user has an image for. Anyone left out keeps the game's drawn face, so partial coverage is fine.
- Every image must be one the user may use: their own art or photos, public-domain images, or Wikimedia Commons images under CC0, CC BY or CC BY-SA. Never suggest headshots from a sports site, a league, a team, a college or a photo agency. Summarize each image's source and license in "meta.notes".

The user will describe the faces pack they want below and should include any relevant league name, the year, the people and image files to include, and any other traits desired for the pack:
```
<!-- /prompt:faces -->

### Logos

One mark per team slot, WebP with a transparent background, 512×512, letterboxed rather
than cropped. For a historical or invented league, name each franchise and the assistant
maps it onto its slot id.

Logo packs load from your own disk only. The game's online catalog does not list or
install them, by design: real team marks are often registered trademarks. A plain roundel
in each team's colors, or any mark you draw yourself, is yours to share.

<!-- prompt:logos -->
```text
You are building the manifest for a team logo pack for the football management game Do or Dynasty. Output ONLY a JSON object (the file manifest.json), no prose, no markdown fences.

Rules:
- Output shape: { "formatVersion": 1, "kind": "do-or-dynasty-skins", "meta": { "title", "author", "forRoster", "notes" }, "players": [], "teams": [ ... ] }. Keep "players" as an empty array (the game requires it) and add no other section.
- The game has exactly 32 team slots with these ids: buf mia ne nyj bal cle pit cin hou ind jax ten den kc lv lac dal nyg phi was chi det gb min atl car no tb ari lar sf sea
  A franchise keeps ONE id through relocations and renames (use the id of the city it plays in today). Map every team the user names onto its slot. Never invent an id.
- Each entry: { "team" (the slot id), "logo" }. One entry per team.
- "logo" paths: logos/<teamid>.webp. Paths are inside the zip: forward slashes, lowercase, hyphens for spaces, no leading "/", no "..". Use the file names the user gives when they give them.
- Images are WebP with a transparent background, 512x512, letterboxed rather than cropped so the edges of the mark survive.
- Teams left out keep the game's colored roundel, so partial coverage is fine.
- Real team marks are often registered trademarks. A logo pack loads from the user's own disk only and is never listed in the game's catalog. For a pack the user means to share, recommend original marks they draw themselves, and summarize each image's source in "meta.notes".

The user will describe the logo pack they want below and should include any relevant league name, the year, the teams and image files to include, and any other traits desired for the pack:
```
<!-- /prompt:logos -->

### Fields

One top-down home field per team, horizontal from end zone to end zone, WebP around
2400×1000. If your image has margin past the end zones, say so and the assistant sets
`left` and `right` to where the back lines sit. A field you paint or render yourself is
yours to share; a photograph of a real stadium belongs to whoever took it.

<!-- prompt:fields -->
```text
You are building the manifest for a home field pack for the football management game Do or Dynasty. Output ONLY a JSON object (the file manifest.json), no prose, no markdown fences.

Rules:
- Output shape: { "formatVersion": 1, "kind": "do-or-dynasty-skins", "meta": { "title", "author", "forRoster", "notes" }, "players": [], "fields": [ ... ] }. Keep "players" as an empty array (the game requires it) and add no other section.
- The game has exactly 32 team slots with these ids: buf mia ne nyj bal cle pit cin hou ind jax ten den kc lv lac dal nyg phi was chi det gb min atl car no tb ari lar sf sea
  A franchise keeps ONE id through relocations and renames (use the id of the city it plays in today). Map every team the user names onto its slot. Never invent an id.
- Each entry: { "team" (the slot id), "image", and optionally "left", "right", "top", "bottom" }. One entry per team.
- "image" paths: fields/<teamid>.webp. Paths are inside the zip: forward slashes, lowercase, hyphens for spaces, no leading "/", no "..". Use the file names the user gives when they give them.
- Each image is a top-down view of the field, horizontal, end zone to end zone. WebP, around 2400x1000.
- "left" and "right" are where the back line of each end zone sits, as a fraction of the image width (defaults 0 and 1). "top" and "bottom" trim the image vertically the same way (defaults 0 and 1). Each pair must satisfy 0 <= first < second <= 1. Omit them when the image starts and ends exactly at the back lines.
- Teams left out play on the game's standard turf, so partial coverage is fine.
- A photograph of a real stadium belongs to whoever took it. Fields the user paints or renders themselves are theirs to share. Summarize each image's source in "meta.notes".

The user will describe the field pack they want below and should include any relevant league name, the year, the teams and image files to include, and any other traits desired for the pack:
```
<!-- /prompt:fields -->

## Sharing what you made

Post the file wherever your community lives. A roster or draft class is facts about a
sport and is yours to share. If you want a roster or draft class listed in the game's
online catalog so other players can download it in one click, the catalog's own guide says
how a package is submitted and what it must carry. Faces, logos and fields are never
listed; they travel as files.

## Reference

- [Roster format](roster-format.md): every field, every tier, the contract rules.
- [Draft-class format](draft-class-format.md): prospects, rating bands, storylines.
- [Skins format](skins-format.md): manifest, matching, logos, fields, sources and rights.
- [Validation glossary](validation-glossary.md): what every error and warning means.
- Examples, small runnable files: the [minimal roster](examples/minimal-tier0.json), an
  [editor-grade roster](examples/maximal-tier3.json), [a legendary coach](examples/famous-coach.json),
  [hall-of-fame legends](examples/legacy-legends.json) and [a draft class](examples/draft-class-minimal.json).
