# CP2077 redscript dev environment

Written after building Night City Dispatch the hard way. Every failure in that
build was preventable with the tooling below.

## What went wrong without tooling

| Failure | Cost | What would have caught it |
|---|---|---|
| `LogChannel` doesn't exist | 1 launch, broken script stack | LSP diagnostics |
| `SystemRequestsHandler` unresolved type | 1 launch | LSP diagnostics |
| `Character.sq023_policeman` is a quest NPC | hard crash, lost session | TweakDB browser |
| `spawnInView = false` queues forever | 3 test cycles | decompiled scripts / docs |
| Callback registered in `OnPlayerAttach` | 2 test cycles | decompiled scripts |
| `spawnEnabled` shipped false | 2 test cycles | compile-time config assertion |

Roughly a dozen game launches at 2-3 min each, plus two crashes. The static
half of that is a sub-second check.

---

## 1. Language server — the highest-value single install

`redscript-ide` gives error and warning diagnostics, autocompletion for methods
and fields, hover for function definitions and types, go-to-definition,
workspace symbols, formatting, and a debugger.

**The critical part is `game_dir`.** Point it at your Cyberpunk install and it
indexes the game's entire compiled script bundle. Unresolved functions and
types are flagged as you type. `LogChannel` would have been underlined.

- Repo: https://github.com/jac3km4/redscript-ide
- VSCode: install the `Redscript IDE` extension by jac3km4
- Zed / Neovim / Helix / IntelliJ configs are in that README

Workspace config — a `.redscript` TOML file in the project root:

```toml
source_roots = [".", "../red4ext/plugins"]

[format]
indent = 2
max_width = 100
```

Add the reference mods you're learning from to `source_roots` and
go-to-definition works across them too.

## 2. Offline compile gate

`redscript` ships a CLI (`redscript-cli`) and `scc`, described as a drop-in
replacement for CDPR's own compiler. That means **you can compile-check without
launching the game.**

- Repo: https://github.com/jac3km4/redscript
- Releases: https://github.com/jac3km4/redscript/releases/latest

This is the thing to wire into Claude Code as a build command. Verify the exact
invocation for your version — the CLI surface has changed across releases — but
the shape is a compile against the game's script bundle plus your source dir,
exiting non-zero on error.

## 3. Decompiled game scripts

**Pre-decompiled, no processing needed:**
https://codeberg.org/adamsmasher/cyberpunk/src/branch/master

Clone it. Open in an IDE that can search across all files at once. This is how
you answer "what does `PreventionSystem` actually do" without guessing.

**Or generate them yourself**, matched to your exact game version — better,
since the public dump may lag patches:

```
redscript-cli decompile -i 'Cyberpunk 2077/r6/cache/final.redscripts.bk' -o dump.reds
```

Use `final.redscripts.bk` (the clean backup redscript makes on install), not
`final.redscripts.modded`, or you'll get every installed mod mixed in.

## 4. NativeDB

https://nativedb.red4ext.com/

All classes, functions, fields, enums, and inheritance. Worth knowing:
**NativeDB has more types than the decompiled scripts** — native functions
exposed to script but not defined in it only appear here. Search covers classes
and enums; functions and fields are visible once you're inside a class page.

Rule of thumb: decompiled scripts for *how something works*, NativeDB for
*does this exist and what's the signature*.

## 5. TweakDB browsing

Cyber Engine Tweaks ships a **TweakDB editor** in its overlay. This is the
answer to "is `Character.foo` real and is it safe to spawn."

Existence is not the same as spawnability. Quest NPCs
(`sq023_*`, `mq010_*`) exist in TweakDB and hard-crash when instantiated cold.
Cross-reference against records the game itself spawns dynamically — vehicle
passenger tables are a good source of known-safe character records.

For bulk/offline work: TweakDump + TweakDB-Edit expand the whole database to
JSON (https://github.com/AlpyneDreams/TweakDB-Edit — flagged outdated, verify
before relying on it).

## 6. Debugger

`redscript-dap`: https://github.com/jac3km4/redscript-dap — breakpoints and
inspection, driven through the language server. Replaces the print-to-screen
telemetry approach entirely once set up.

## 7. Fast runtime iteration

CET's Lua console executes against the live game with no recompile and no
restart. Prototype logic there, port to redscript once the shape is right.
psiberx's `cp2077-cet-kit` (https://github.com/psiberx/cp2077-cet-kit) has
helpers for game-state tracking and timers.

---

## Testing approach

There is no unit test harness for redscript, and no headless mode. Be honest
about that and build the layers that do exist.

**Layer 1 — static, sub-second, every save.** LSP diagnostics.

**Layer 2 — static, seconds, every commit.** CLI compile gate. This alone would
have caught two of our six failures and cost nothing.

**Layer 3 — assertions on config.** Most of our lost cycles were a config value
being wrong, not logic being wrong. A startup self-check that logs every config
value once, loudly, at session start would have caught `spawnEnabled = false`
immediately. Cheap and worth it.

**Layer 4 — a clean test profile.** A separate MO2 profile with only the
required core mods (redscript, RED4ext, Codeware, TweakXL, ArchiveXL, CET) plus
the mod under test. Loads faster, isolates conflicts, and rules out the
"is it my mod or Much Better AI" question in one launch. Keep a dedicated test
save in a target-rich area.

**Layer 5 — in-game telemetry.** What we built by hand: a probe printing
counters on screen every few seconds. Formalize it as a reusable harness rather
than rewriting per project. Key lesson: **instrument every stage boundary
separately.** `spawn accepted` vs `attached` vs `alive` were three different
numbers, and collapsing any two of them hid a bug.

---

## Install order

1. Clone the decompiled scripts (codeberg link above)
2. Install redscript-ide + VSCode extension, set `game_dir`
3. Add `.redscript` to the project root
4. Download redscript CLI, confirm a compile-check invocation
5. Bookmark NativeDB
6. Make the clean MO2 test profile
7. Optional but high value: redscript-dap

Steps 1-4 are maybe an hour and change the loop from
*edit → launch → crash → guess* to *edit → red squiggle → fix*.
