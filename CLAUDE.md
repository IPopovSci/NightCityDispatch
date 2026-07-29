# CLAUDE.md — Cyberpunk 2077 redscript project

## Hard rules

**Never invent an API name.** Every function, type, field, and enum value must
be traceable to one of:

1. `./decompiled/` — the decompiled game scripts
2. NativeDB (https://nativedb.red4ext.com/) — has more types than the scripts
3. A working mod's source in `./reference/`
4. Official Codeware docs (https://github.com/psiberx/cp2077-codeware/wiki)

If you cannot point to the source, do not write the call. Say so instead.
A wrong name is not a compile warning — it takes down every redscript mod in
the user's load order until it's fixed.

**Prefer type inference over annotation** for anything returned by a game
system. `let x = GameInstance.GetFoo();` cannot be wrong about the type name;
`let x: ref<FooSystem> = ...` can, and has been.

**Grep before writing.** `rg "FunctionName" decompiled/ reference/` first.
Reading how a working mod calls something beats reconstructing it from memory.

**Existence is not usability.** A TweakDB record existing does not mean it can
be spawned cold. Quest-scene records (`sq*`, `mq*` prefixes) carry scene
dependencies and crash natively. Only use records the game itself spawns
dynamically, and say which source proves it.

## Build gate

Run the compile check after every change to a `.reds` file. Do not report a
change as done before it compiles.

```
# fill in the verified invocation for the installed redscript CLI version
./scripts/check.sh
```

Non-zero exit means broken. Read the error, fix, re-run. Errors name the file
and line.

## Semantics that are not obvious from names

These cost real debugging time. Check the docs before assuming a flag's
meaning.

- `DynamicEntitySpec.spawnInView = false` does not skip the spawn — it **queues
  it indefinitely** until the player looks away. `CreateEntity` still returns a
  valid EntityID.
- A valid EntityID from `CreateEntity` means **the request was accepted**, not
  that an entity exists. Confirm via the `Entity/Attached` callback.
- `DynamicEntitySystem.GetTagged()` has been observed returning 0 while the
  attach callback fires for those same tags. Do not use it as a population
  count. Track spawned IDs yourself.
- `ScriptableSystem.OnPlayerAttach` fires on a **new game only**. Loading a save
  goes through `OnRestored`. Register callbacks in `OnAttach`.
- Hold game systems as `wref`, not `ref`. Strong refs interfere with session
  disposal and can crash.
- Always clean up dynamic entities and cancel pending callbacks on
  `Session/BeforeEnd` and `OnDetach`.

## Diagnostics

There is no debugger attached by default and no log the user can reliably read
mid-session. `FTLog` does not surface in the CET console on all setups, and
`redscript_rCURRENT.log` is **compile-time only**.

Use on-screen output for runtime telemetry:

```reds
let msg: SimpleScreenMessage;
msg.isShown = true;
msg.duration = 4.0;
msg.message = text;
GameInstance.GetBlackboardSystem(gi).Get(GetAllBlackboardDefs().UI_Notifications)
  .SetVariant(GetAllBlackboardDefs().UI_Notifications.OnscreenMessage, ToVariant(msg), true);
```

`LogChannel` does **not** exist. Do not use it.

**Instrument every stage boundary separately.** Collapsing two stages into one
counter hides bugs. Requested / accepted / attached / alive were four distinct
numbers and three of them lied at different times.

**Any feature flag that disables work must be visible in the telemetry.** A
switched-off subsystem and a broken one produce identical output otherwise.

## Config

All tunables live in one config file. It is overwritten on every install, so
never silently change a default — if a default changes, say so explicitly in
the changelog and in the response to the user.

## Style

- Two-space indent, matching the `.redscript` formatter config
- Comments explain *why*, especially where a value is defensive against a
  specific known failure
- Every version gets a changelog entry naming what was wrong, not just what
  changed
