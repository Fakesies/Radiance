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

1. Setup: server running, logged in, world name set to Radiance (done). Login
   message, first plugin (`::starter`), commit and push.
2. Account and XP modes (first custom feature, see design notes).
3. Multi-character accounts with a character select screen (mode is picked when
   creating a character).
4. Duo and group ironman: groups, invites, shared storage, shared HC lives.
5. Pet perks.
6. Leagues systems on the main world (which ones still to be decided).
7. RS3 and Ferox-style systems: RS3 mechanics with OSRS assets (bank presets,
   portables, area loot), weapon upgrades with random perks, prestige system,
   custom interfaces.
8. Lumbridge town square: an RS3-style town square built from OSRS objects with a
   map editor (RSPSi; first check its OSRS plugin supports rev 241).
9. Fill OSRS gaps: Fletching, Agility, Hunter, more bosses and minigames.
10. Big content: Chambers of Xeric, then ToB and ToA. Launcher and hosting once
   it's fun to play.

## Design notes

### Account and XP modes
Each character has one account mode and one XP mode, picked when the character is
created.

Account modes:

| Account mode | Allowed XP modes |
| --- | --- |
| Normal | Any |
| Ironman | Any |
| Hardcore Ironman | Any |
| Duo Ironman | Elite only |
| Group Ironman | Standard only |
| Hardcore Group Ironman | Standard only |

Ultimate Ironman is not offered (OpenRune supports it, Radiance doesn't use it).

XP modes (multipliers are combat / skilling):

| XP mode | Combat | Skilling | Drop rate boost |
| --- | --- | --- | --- |
| Relaxed | 50x | 30x | 0% |
| Standard | 25x | 18x | 5% |
| Elite | 12x | 8x | 10% |
| Cronto | 4x | 4x | 12% |

- Combat skills: Attack, Strength, Defence, Ranged, Magic, Hitpoints, Prayer.
  Everything else is skilling, including Slayer.
- Harder XP modes will also give more points in the prestige system (designed later).
- Changing XP mode: only toward easier (Cronto -> Elite -> Standard -> Relaxed),
  never harder. Duo and group ironman stay locked to their allowed mode.
- De-ironing: ironmen can de-iron and become a Normal account, keeping their
  current XP mode.
- Hardcore Ironman: an unsafe death demotes to Ironman (OpenRune already does this).
- Hardcore Group Ironman: shared group lives, same as OSRS (check the OSRS wiki for
  the exact rules).
- Group sizes: Duo is 2, Group is 2 to 5.
- OpenRune already has Normal, Ironman and Hardcore Ironman with restrictions
  enforced (see `docs/ironman.md`). Duo and group ironman are new.
- Open questions: leaving or disbanding a duo/group, and what de-ironing does to
  the group.

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
- Creating a character means choosing a name, an account mode and an XP mode, with
  the allowed combinations enforced (see Account and XP modes).
- Picking a character loads that character's save. OpenRune's in-place save swap
  (used for world types) is the likely starting point.
- Open questions: characters per account, anything shared across characters,
  switching without logging out.

## Open items

- OpenRune-DeveloperTools (the RuneLite plugin that lets an AI agent drive the
  client) can't load through RSProx's RuneLite: RuneLite only loads sideloaded
  plugins when started without its launcher, and RSProx always launches through
  one. Ask OpenRune which client they use with it.
