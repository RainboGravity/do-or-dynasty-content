# PickSix custom content

PickSix ships with a fictional league. You can play in any league you can describe: real
teams, a historical season, an all-time fantasy draft, a league you invented. The world
comes from a single roster file, and draft classes, faces, team marks and home fields are
optional files on top of it. Share the file and you share the exact world: identical bytes
produce the identical league on every machine.

**Start with the [Make Your Own guide](authoring-guide.md).** It has the two prompts to hand
an assistant, the merge step, and the three rules that let any league fit the game's 32
slots.

## Reference

| Page | For |
|---|---|
| [Make Your Own](authoring-guide.md) | The short road: prompts, merge, load, iterate |
| [Roster format](roster-format.md) | Every field and tier of a roster file |
| [Draft-class format](draft-class-format.md) | Prospects, rating bands, storylines |
| [Skins format](skins-format.md) | Faces, logos and fields: manifest, matching, sources and rights |
| [Validation glossary](validation-glossary.md) | What every error and warning in the load report means |
| Examples | Small runnable files: [minimal roster](examples/minimal-tier0.json), [editor-grade roster](examples/maximal-tier3.json), [a legendary coach](examples/famous-coach.json), [hall-of-fame legends](examples/legacy-legends.json), [a draft class](examples/draft-class-minimal.json) |

## The idea: facts, not internals

You state football facts a fan knows: a name, a team, a position, an overall rating. The
game derives everything else (attribute vectors, contracts, cap math, career arcs) through
the same systems it uses for its own players. You never touch internal schema. A player
fact says who the player is; the game figures out what he is.

The shortest possible roster file:

```json
{
  "formatVersion": 1,
  "players": [
    { "name": "Marcus Vale", "team": "cin", "position": "QB", "ovr": 88 }
  ]
}
```

Every team you don't mention keeps its shipped defaults. Every roster hole the game fills
for you, and the load report tells you how many.

## Loading a file

**New Game → Roster → Load .json file.** You'll see a validation report: errors block (fix
and reload), warnings proceed (they explain what the game filled in or adjusted), info
summarizes the league you're about to enter. If it's clean, start in that world and pick
your franchise.

## Versions

This is the guide for roster, draft-class and skins **format version 1**. The game's
**New Game → Make Your Own** panel links to the guide that matches the installed build.

PickSix is not affiliated with, endorsed by, or licensed by any league, team or player.
Files you make are yours; the skins guide's **Sources and rights** section says what a
file you share may carry.
