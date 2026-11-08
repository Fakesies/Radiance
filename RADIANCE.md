# Radiance

Radiance is Austin's RuneScape private server, built on OpenRune (OSRS revision 241).
Goal: play as close to current OSRS as possible, plus selected leagues, RS3 and
Ferox-style systems on the main world. There is no separate leagues world; leagues
systems are normal plugins that apply to everyone.

## Rules for working in this repo

- All Radiance code goes in `content/radiance/<feature>/`, package
  `org.rsmod.content.radiance.<feature>`.
- Don't edit OpenRune's own files. If something upstream needs to change, note it
  and prefer a hook, a config option, or a pull request to OpenRune. Keeping our
  changes separate keeps upstream merges painless.
- Propose changes and wait for Austin's go-ahead before editing files.
- Follow `AGENTS.md` for OpenRune's conventions (varbits/varps over attrs, gamevals
  instead of raw ids, interface design rules, tests).

## Setup notes

- Windows, IntelliJ IDEA, Java 21 (Temurin).
- Server: Gradle `install` once, then Gradle `run`. Success prints
  `Server ready` and `Bound to ports: 43594`.
- The `install` task ends with "Configuration cache problems found in this build"
  and BUILD FAILED. This is a quirk in OpenRune's build script (the final
  `game.yml` copy isn't configuration-cache compatible). Install still completes:
  `game.yml`, `.data/client.key` and `.data/cache` all get created.
- Client: RSProx with Client Type "Old School (RuneLite)" and the "Radiance" proxy
  target (`%USERPROFILE%\.rsprox\proxy-targets.yaml`, modulus from
  `.data/client.key`).
- Git remotes: `origin` = github.com/Fakesies/Radiance, `upstream` =
  github.com/OpenRune/OpenRune-Server.
- Austin tests in game and reports back; there's no automated in-game testing yet
  (see Open items).

## Roadmap

1. Setup: server running, logged in (done). Brand it (login message, world name),
   first plugin (`::starter`), commit and push.
2. Small wins: a few small custom features (shop, NPC with dialogue, command),
   practice pulling upstream updates.
3. Multi-character accounts with a character select screen.
4. Pet perks.
5. Leagues systems on the main world (which ones still to be decided).
6. RS3 and Ferox-style systems: RS3 mechanics with OSRS assets (bank presets,
   portables, area loot), weapon upgrades with random perks, custom interfaces.
7. Fill OSRS gaps: Fletching, Agility, Hunter, more bosses and minigames.
8. Big content: Chambers of Xeric, then ToB and ToA. Launcher and hosting once
   it's fun to play.

## Design notes

### Pet perks
- Each pet has 5 perk slots. A general perk pool, plus unique perks for some pets.
- Pet rating sets the base odds per slot (example, tier 50: S1 84%, S2 34%,
  S3 12%, S4 3%, S5 -34%, where negative means locked out).
- Rolling costs pet tokens. 1 token rolls at base odds; more tokens bump the odds
  slightly.
- Every unlocked slot rolls independently. Slots can be locked for tokens.
- No duplicate perks on one pet. Perks only apply while the pet is following.
- Draft roll logic exists (standalone Kotlin, not yet in the repo).

### Character select
- After login, the player is held in a lobby with a proper custom interface listing
  the account's characters (name, combat level, total level, location, last
  played) with Play, Create and Delete.
- Picking a character loads that character's save. OpenRune's in-place save swap
  (used for world types) is the likely starting point.
- Open questions: characters per account, anything shared across characters,
  switching without logging out, per-character modes (e.g. ironman).

## Open items

- OpenRune-DeveloperTools (the RuneLite plugin that lets an AI agent drive the
  client) can't load through RSProx's RuneLite: RuneLite only loads sideloaded
  plugins when started without its launcher, and RSProx always launches through
  one. Ask OpenRune which client they use with it.
