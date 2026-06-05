# Stand Test — Polish + Whitesnake/C-Moon Pass

## 1. Fix the build error
`Game.tsx` references `MiniMap`, `LegendDot`, and `ChangelogEntry` that were never defined. Add three small components at the top of `Game.tsx` (or in a new `src/components/game/` folder):
- `MiniMap` — uses existing world projection logic with player/NPC/item dots.
- `LegendDot` — `{ color, label }` swatch row.
- `ChangelogEntry` — `{ version, date, children }` styled block.

## 2. Autosave / resume
- On every meaningful state change (stand acquired, HP, position, inventory, gold, kills, unlocked codex) write a debounced snapshot to `localStorage` under `standtest.save.v2`.
- Also save on `visibilitychange` (hidden) and `beforeunload`.
- On boot, if a save exists, hydrate world: player position, HP, stand, inventory, gold, codex unlocks, changelog-seen flag.
- Add a small "Reset save" button in the inventory's Settings/How-to-Play tab.

## 3. Mini-map redesign
Replace pure dots with a small **schematic of the actual map**:
- Draw the real ground tiles/biome rects (city block, forest patch, water, roads) scaled down — reuse the same shape data the engine uses for rendering so the minimap reads as the map, not abstract circles.
- Player = triangle (facing dir), Neutral NPC = small green square, Hostile = red square, Items = tiny diamond colored by type.
- Keep the ~900-unit window clamp.
- Border + compass "N" marker for orientation.

## 4. Item-use confirmation
When the player clicks an inventory item that triggers a consume (Arrow, Requiem Arrow, Hat, Blue Pebble, Green Arrow, Green Baby, DISC):
- Open a small confirm popover: "Use {Item}? [Use] [Cancel]".
- Only consume on confirm. Hat / Arrows still warn about overwriting current stand.

## 5. Lock-on button
- Add a 🎯 button stacked **above the A2 slot** in the action bar.
- Toggles `w.lockedTargetId`. While locked: target reticle drawn over enemy, abilities that auto-aim (Eagle Shot, Triple Pebble, Life Beam, Improvisation, Star Finger) prefer this target; auto-aim cone code already exists, just bias toward the locked id.
- Auto-clear when target dies or leaves a generous range (~600).

## 6. M1 always available
- Currently M1 requires the stand to be summoned. Change: if `standId !== "none"` and stand is *un*summoned, M1 falls back to the player's own punch (same damage as the stand's M1 minus ~40%, uses player-arm animation). If `standId === "none"` it stays as the basic punch.

## 7. Hostile NPC punch range parity
- Give hostile NPCs the same `range`/`radius` as the player's basic punch (currently NPCs need to overlap). Add a brief punch-arm anim in `drawNpc` mirroring `drawPlayer`'s swing.

## 8. RHCP A3 rework
- Replace `Ground Bomber` AOE with **Lightning Pillar**: single-target, picks closest enemy in range (or locked target), 0.4s telegraph ring on ground, then a vertical cyan/yellow electric column for 0.35s. Damage ~14, small stun 0.6s, leaves a crater. Cooldown 6s.

## 9. Whitesnake (new stand) + green W.I.P. Arrow
- Add `whitesnake` to `StandId`, `STANDS`, codex.
- New ability kinds:
  - `ws_improvisation` — 3 timed bullets, 5 dmg each, last with knockback 120, cd 12.7.
  - `ws_disc_steal` / `ws_disc_implant` — A2 has two states tracked per target id; first tap steals DISC (slow 6.3s, 7.9 dmg, cd 4.9), tap-again-within-window on same target implants (3.4 dps for 10s, steamy/bubble VFX, cd 5.9).
  - `ws_acidic_stab` — 4.7 dmg + 1 dps for 7s, cd 16.
  - `ws_hypnotic_spit` — projectile, 0 dmg, stun 12s, cd 13.
- M1 dmg 2.6. Passive: see §11.
- **Green W.I.P. Arrow** item: rare spawn (~1/4 of normal arrow rate, own pickup model = green arrow). Using it always rolls Whitesnake.
- Add to inventory grid + confirm-use popover.

## 10. Green Baby item + C-Moon evolution
- New item `green_baby` (small plant-baby sprite, red eyes, gold tinge). Spawns very rarely (similar to Requiem Arrow cadence).
- Inventory: enabled only when `standId === "whitesnake"`; on confirm-use → grants `c_moon`, consumes baby.
- C-Moon stand:
  - M1: 3.5 dmg, can damage props/craters.
  - Passive `inversion`: 12.7% chance incoming hit reflects to attacker for same damage.
  - A1 `cmoon_punishment` — gravity-inversion bubble (radius ~140) for 8s, hostile NPCs inside float (disable AI move, lift sprite Y over time), on end any still-affected take 18 dmg. CD 21.
  - A2–A4: leave as `-` placeholders (W.I.P. consistent with prompt only specifying A1).

## 11. Prime-number heal passive (Whitesnake & C-Moon)
- While stand is equipped, cycle through the first 9 primes (2,3,5,7,11,13,17,19,23) as floating text above the player every ~1.6s; each tick heals a small flat amount (e.g. 0.4 HP) capped at max HP. Pauses when dead.

## 12. Stand mechanics / VFX polish (JJBA-accurate, scoped)
- **Stand aura**: replace current ring with a soft layered wave — two offset radial gradients pulsing at different frequencies in the stand's color, plus faint vertical "menacing" ゴ glyphs drifting up at low alpha when stand is summoned. Tuned per stand color.
- **Star Platinum / SPTW**: punch flurry trails use afterimage sprites (3 ghosts at decreasing alpha) instead of plain lines.
- **GER**: pink/gold particle motes orbit the player, life-beam gets a sparkle head.
- **Echoes text moves**: render the JP text larger with a slight rotation + drop shadow for impact.
- **Purple Haze**: capsule shards add a small purple gas puff on impact.
- **White Album**: ice stomp adds a brief frosted ring decal.
- **Hanged Man** mirror shards: thin specular flash when teleporting.
- **Items** model touch-ups (kept minimal): Arrow gets feathered tail; Requiem Arrow gets shimmer; Hat gets the gold palm box already added; Blue Pebble gets a subtle glow; Green Arrow has feather fletching tinted green; Green Baby has eyelash detail + gold cheek.

## 13. Changelog tab content
Add new entry for this pass summarizing items 1–12 so players see it in-game.

---

## Technical notes (for the build step)
- **Files touched**:
  - `src/game/stands.ts` — add `whitesnake`, `c_moon`, new ability kinds, primes-heal passive flag.
  - `src/game/types.ts` — extend `WorldState` with `lockedTargetId`, `discTargets: Map<string,{stage:'stolen'|'implanted', until:number}>`, `primeTickAt`, `primeIndex`, `gravityZones[]`, `saveDirty`.
  - `src/game/engine.ts` — implement new ability kinds, NPC punch range/anim, lightning pillar, gravity bubble, aura/VFX changes, ground-scar already exists.
  - `src/game/codex.ts` — Whitesnake, C-Moon, Green Arrow, Green Baby entries.
  - `src/components/Game.tsx` — define missing components (fixes TS errors), lock-on button, item confirm popover, autosave hooks, minimap redesign, changelog entry, M1-without-stand wiring.
  - New: `src/game/save.ts` (load/save/clear helpers).
- No backend changes; everything is client-side `localStorage`.
- Keep all new colors via existing hex palette in stand data (no design-token migration in this pass).
