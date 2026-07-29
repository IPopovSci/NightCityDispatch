# Night City Dispatch — Design Vision

## The missing element

Night City has no shortage of mods that make the world violent. Gangs fight,
traffic is dense, NPCs react, factions remember. What none of it has is
**consequence**. A six-man firefight can run for two minutes across from a
police checkpoint and nothing in the world acknowledges it happened.

NCPD in the base game is a function of the player's wanted level and nothing
else. Crime that isn't yours doesn't exist to them.

This mod adds the reactive layer: **violence in the world generates a police
response proportional to its scale, duration, and location** — independent of
whether V is involved at all.

The design goal is not "more cops." It is *plausibility*. A player should be
able to watch a shootout escalate, see NCPD arrive, and find the outcome
unsurprising.

---

## Core principles

**1. Response is proportional, not binary.**
A stabbing in an alley is not a MaxTac deployment. Most violence should produce
either nothing or a single cruiser slowing down as it passes.

**2. Location matters more than severity.**
This is the most Night City thing in the design. The same firefight produces a
gunship in Corpo Plaza and total silence in Pacifica. Response is a function of
*whose property is being damaged*, not of harm caused.

**3. Police are reluctant.**
NCPD is underfunded, understaffed, and its officers know it. They arrive
cautiously, hold perimeter, and wait for numbers before pushing in. They do not
charge into a superior force. Cowardice is characterful here, not a bug.

**4. Escalation is earned over time, not triggered by a number.**
MaxTac shows up because an incident has been running for ninety seconds, three
officers are down, and it isn't contained — not because a counter crossed 34.

**5. V is a bystander until V isn't.**
Responders are neutral to the player by default. They are not a wanted-level
system in disguise.

**6. Incidents end.**
Something that starts must resolve, get cleaned up, and leave a trace. Units
that never leave are worse than units that never arrive.

---

## District response doctrine

The single highest-value axis in the whole design. Each district gets a
response profile: how fast, how much, and whether at all.

| Zone | Response time | Ceiling | Character |
|---|---|---|---|
| **City Center** (Corpo Plaza, Downtown) | Very fast | MaxTac | Disproportionate. Corporate property. Arrives in force for things that would be ignored elsewhere. |
| **Westbrook** (Charter Hill, North Oak, Japantown) | Fast | Tactical | Wealth is protected. Japantown is Tyger Claw territory, so response is present but negotiated — slower, more cautious. |
| **Heywood** (Wellsprings, The Glen) | Moderate | Tactical | Government district. Competent, unremarkable. |
| **Heywood** (Vista del Rey) | Slow | Patrol+ | Valentino turf. NCPD shows up late and in numbers, or not at all. |
| **Watson** (Little China, Kabuki) | Slow | Patrol+ | Overstretched. Response exists but is thin and arrives after the fact. |
| **Watson** (Northside) | Very slow | Patrol | Maelstrom territory. Minimal presence. Often just a drive-by that doesn't stop. |
| **Santo Domingo** (Arroyo, Rancho Coronado) | Slow | Patrol+ | Industrial. Corporate security responds to corporate property; NCPD covers the rest badly. |
| **Pacifica** | None | — | Abandoned combat zone. NCPD does not enter. This is the point. |
| **Badlands** | None | — | Outside jurisdiction. Nomad and corpo problems only. |

**Pacifica having no response is a feature.** It's the clearest way the system
communicates its own logic to the player. Firefights there escalate forever and
nobody comes, exactly as the setting says.

**Corporate security** substitutes for NCPD on corporate property — Arasaka,
Militech, Biotechnica. Faster than police, less interested in bystanders, only
defends its own perimeter.

---

## Incident model

An **incident** is a cluster of violence in space and time.

**What feeds it:**

| Signal | Weight | Notes |
|---|---|---|
| NPC enters combat | Medium | Primary signal — the game telling us a fight started |
| NPC enters alerted | Low | Bystander fear. Volume matters more than individual events. |
| NPC death | High | |
| Officer death | Very high | The single strongest escalator |
| Explosion | High | |
| Cyberware/cyberpsycho indicators | Special | Routes to MaxTac regardless of scale |

**How it behaves:**
- Events within ~30 m merge into one incident
- Heat decays continuously; a fight that stops resolves on its own
- **Ongoing combat feeds sustained heat** — this is what separates a brief
  exchange from a prolonged battle, and it's the core of the MaxTac gate
- Heat is capped, so a saturated incident can't escalate infinitely
- Multiple concurrent incidents are supported; the world is not required to
  have only one thing happening

**What doesn't count:** melee brawls, single punches, NPCs fleeing without
violence. The city is rough. Police don't come for rough.

---

## Escalation ladder

Each tier has **entry conditions**, not just a threshold. Tiers gate on
district ceiling — a Northside incident can't reach tactical no matter how bad
it gets.

### Tier 0 — Noticed
A patrol cruiser already in the area slows, or diverts past. Does not stop, does
not engage. Cost: nothing. This is the most common outcome by far and it's what
makes the rest feel earned.

### Tier 1 — Patrol response
**Trigger:** sustained violence, ~10-20 s, above minimum severity.
**Force:** 1 cruiser, 2 officers.
**Behavior:** arrive by road, stop at distance, exit, take cover, do not push.
They are outnumbered and they know it. If the fight is bigger than expected,
they call for backup rather than dying heroically.

### Tier 2 — Containment
**Trigger:** incident still active after first response, or first response takes
casualties.
**Force:** 2-3 more vehicles, 4-6 officers, arriving over ~20 s rather than at
once.
**Behavior:** establish perimeter. Block road access. **Contain rather than
assault.** This is the tier most incidents should end at — the gangs kill each
other, police watch the exits, then move in when it's quiet.

### Tier 3 — Tactical
**Trigger:** containment failing — combatants breaking the perimeter, officer
casualties mounting, heavy weapons, or a hostage/civilian situation.
**Force:** +4 tactical officers, better armed and armored.
**Behavior:** they actually push in. Coordinated entry, flanking.

### Tier 4 — MaxTac
See below. Deliberately hard to reach.

---

## MaxTac gating

MaxTac is a psycho squad. In the fiction they exist for one thing: people whose
cyberware has eaten them. They are not a SWAT escalation, they are a
specialist unit, and their arrival should be *an event the player remembers*.

**Any one of these qualifies:**
- **Cyberpsychosis indicators** — the canonical trigger. Heavy chrome, a single
  combatant with disproportionate kill count, sandevistan/berserk signatures.
- **Officer casualties: 3+** in one incident. Losing that many badges changes
  the department's posture entirely.
- **Prolonged uncontained incident** — Tier 3 active and still not resolving
  after ~90 seconds of continuous violence.
- **District override** — City Center. Corporate pressure produces
  disproportionate force. MaxTac for something that would get one cruiser in
  Santo Domingo is *correct* and says something about the city.

**Explicitly does not qualify:**
- Raw combatant count alone
- Raw heat alone
- Any single burst of violence, however intense, that resolves quickly

**Force:** 4-6 elite. Arrive last, arrive hard.

**Arrival:** the AV dropship is the correct fiction and the hardest thing in the
project. Phase it — ground arrival first, aerial insertion later, and only once
everything below it is stable.

---

## Other responders

**Trauma Team** — responds to *subscribers*, not to violence. If a Platinum
subscriber is injured in the area, TT arrives fast and heavily armed, extracts
their client, and leaves. They do not care who is fighting. They will kill
anyone who impedes extraction, including police.

This is a beautiful piece of Night City texture and it's mechanically simple:
a rare roll on NPC injury, an independent arrival, a single objective, and a
departure. It also creates genuinely interesting three-way scenes.

**Corporate security** — replaces NCPD on corporate property. Faster, better
equipped, jurisdiction ends at the property line.

---

## Arrival and staging

The failure mode that breaks immersion hardest is cops materialising out of
nothing at a jog.

- **Arrive by road, in vehicles**, from the direction of the nearest road access
- **Stage at distance** — stop 20-30 m out, exit, take cover behind the vehicle
- **Approach as a group**, not as individuals sprinting in
- **Response time is real** — 20-60 s by district. Delay is not a defect; it's
  the mechanism that lets the player watch a situation develop.
- Never spawn within player line of sight if avoidable

## Resolution and aftermath

- When violence stops, units sweep the area rather than vanishing
- Perimeter holds briefly, then releases
- Units leave the way they came, or despawn well out of sight
- Bodies remain; the world keeps the evidence
- The incident closes so the same location can generate a fresh one later

## The player's relationship to all this

- **Neutral by default.** Responders do not care about V.
- **V can be caught in crossfire** — stray rounds, not targeting
- **Shooting at responders makes V a target**, normally, via the vanilla wanted
  system. Nothing special.
- **Helping the police does not make them friendly.** They don't know who you
  are and they aren't grateful.
- **If V becomes wanted at a live incident**, see *Ownership transfer* below.
  The short version: V joins the incident, the incident does not disappear.

---

## Ownership transfer — when V gets stars mid-incident

The awkward case, and the one most likely to be hit in normal play: a gang
firefight is running, our units have arrived, V is a bystander, and a stray
round from V clips an officer. V now has stars.

**The wrong answer is shutting down.** The gang fight is still happening. If the
system stands aside, an incident with six combatants and two dead officers
silently stops existing because V made a mistake. The world's attention
collapses back onto the player, which is exactly the failure this mod was built
to fix.

**The wrong answer is also running both systems at full tilt.** Vanilla spawns
for V, we spawn for the incident, and the scene doubles in unit count with two
authorities disagreeing about who is fighting whom.

**The right answer: V becomes a combatant in the existing incident.**

Realistically, you don't get your own separate police response for shooting a
cop at a firefight. You become the police operation's second problem. The
incident is already open; V joins it.

### Rules

**1. The incident stays open and keeps escalating.** Heat continues to
accumulate. V's violence feeds it exactly like any other third party's.

**2. Attitude re-evaluates on wanted-state change — for existing units, not
just new ones.** When V acquires stars at an incident, the friendly override is
dropped from every unit we have on scene, and vanilla attitude takes over. A
unit that was ignoring V thirty seconds ago is now a normal hostile cop. This
must be an active transition, not a policy applied at spawn.

**3. Spawn authority passes to vanilla.** While vanilla prevention is actively
chasing V, our dispatcher stops *adding* units. It keeps tracking, keeps
escalating, keeps the incident alive — it just stops being the thing that
spawns. This avoids stacking without discarding the incident.

**4. Officer deaths caused by V count toward escalation, including the MaxTac
gate.** This is the good consequence. Get sloppy at someone else's firefight,
kill three cops, and you can legitimately pull MaxTac onto a scene you were
only ever a bystander at. That is a *great* Night City outcome and it should not
be special-cased away.

**5. When V's stars clear and the incident is still hot, control returns.**
Friendly override is restored after a short cooldown, spawn authority comes
back, and the system resumes. The fight was never ours to end.

### Degrees of guilt

Vanilla already models this and we should not duplicate it. A stray round in
crossfire produces a brief low-level reaction; deliberately engaging police
produces a real chase. Let vanilla's heat stages express the difference and
read from them rather than inventing a parallel scale.

### Identification (later phase)

Realistically, officers arriving *after* the shot don't know V fired it. Only
those with line of sight should react. Detection-overhaul mods already model
identification and line-of-sight, so this is texture to add once the core
transfer works — not something to build from scratch.

### What this looks like in play

You're watching Valentinos and 6th Street trade fire from behind a dumpster.
Two cruisers arrive and hold the street. You lean out, snap at a Valentino,
clip an officer instead. Every cop on that street turns. Vanilla gives you two
stars and its own units start arriving. The gang fight is *still going* — heat
is still climbing, the incident is still escalating around you. You break line
of sight, lose the stars two blocks away, come back, and the perimeter is still
there, still holding, because it was never about you.

---

## Anti-goals

Things that would make this feel wrong, listed so they don't creep in:

- Cops appearing from thin air near the player
- Response identical in every district
- Unbounded spawning; police as a tide
- Responders beelining for V
- MaxTac as a generic top tier rather than a specialist unit
- Units that never leave
- Any behavior that makes the player feel *targeted* rather than *present*
- An incident silently ceasing to exist because V acquired a wanted level
- Units that were forced friendly staying friendly after V shoots at them

---

## Incident lifecycle

The spine of the whole system:

```
DORMANT
   │  violence detected, above district minimum
   ▼
REPORTED ──────► (decays out) ──► CLOSED
   │  sustained past response delay
   ▼
RESPONDING  ── Tier 1 arriving
   │  still active / casualties
   ▼
CONTAINING  ── Tier 2, perimeter established
   │           └──► violence stops ──► SWEEPING
   │  perimeter failing / casualties
   ▼
ASSAULTING  ── Tier 3 pushing in
   │  90 s uncontained OR 3 officers down OR cyberpsycho
   ▼
SPECIALIST  ── Tier 4 MaxTac
   │  violence stops
   ▼
SWEEPING ──► units clear area ──► WITHDRAWING ──► CLOSED
```

Every tier can terminate straight to SWEEPING. Most incidents should never get
past REPORTED.

---

## Phasing

**Phase 1 — foundation.** Detection, incident clustering, escalation logic,
on-foot spawning, neutral-to-V attitude, cleanup. *(current)*

**Phase 1.5 — ownership transfer.** Attitude re-evaluation on wanted-state
change, spawn authority handoff to vanilla, restoration on clear. Small, and it
closes the most likely hole in normal play.

**Phase 2 — plausible arrival.** Vehicles, road-based spawn points, staging at
distance, response delay.

**Phase 3 — the Night City layer.** District response profiles. This is where
the mod stops being "cops respond" and starts being *Night City*.

**Phase 4 — texture.** Trauma Team, corporate security, sweep and withdrawal
behavior, cordons.

**Phase 5 — spectacle.** MaxTac AV insertion.

Phase 3 is where the design payoff is concentrated, and it's mostly data rather
than engineering. Worth reaching sooner than the ordering suggests.

---

## Out of scope

Named so nobody wastes a weekend on them:

- **Off-screen simulation.** Incidents exist near the player only. The engine
  streams NPCs aggressively; there is nothing to police 300 m away. Simulating
  abstract incidents and materialising them on approach is a separate project.
- **Arrests, de-escalation, surrender.** Fights end when one side is down.
- **Persistent world state.** Incidents don't survive a reload.
- **Rebuilding vanilla prevention.** It's a singleton with one heat stage and
  one chase target. It cannot represent three simultaneous incidents. This
  system runs alongside it, not through it.
