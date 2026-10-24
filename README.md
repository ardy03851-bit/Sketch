# Sketch Brawl — iPad-Optimized Indie Multiplayer

A round-based PvP game where every player receives one randomly selected weapon per round.
Weapons have **one use maximum** — no abilities, no reloads. Guns shoot. Swords slice.
The map is destructible: craters, dents, and debris persist across rounds.

---

## Art & Style Guide

**Hand-drawn sketch aesthetic.** Every surface uses wobbly, slightly imperfect outlines.
Colors sometimes bleed a few pixels outside the shape — this is intentional and emulates
a marker-pen drawing.

- **Characters:** Maximum two primitive shapes (circle head + capsule body). No eyes, no mouth.
- **Weapons:** Rendered from `weapons/<name>/<name>.png`. If the PNG is missing, a
  fallback texture (crossed-out square) is shown automatically for testing.
- **Lighting:** Soft radial gradients. Ambient occlusion via a dark vignette.
- **VFX:** Craters are permanent. Debris particles use velocity stretching, OKLCH-style
  heat gradients (white → orange → cherry red), and floor bounce with restitution.
  No cartoonish stars or smiley faces — the particles are sharp, physical, and minimal.
- **Outlines:** Drawn with Rough.js-style jitter. Line width varies. Corners are never
  perfectly square.

---

## Controls (iPad-Optimized)

The game uses **unified Pointer Events** (`pointerdown`, `pointermove`, `pointerup`).

| Action | Input |
|--------|-------|
| Move | Drag the left half of the screen (virtual stick appears under your thumb) |
| Aim | Drag the right half of the screen |
| Fire / Attack | Tap the right half while aiming |
| Menu | Tap UI buttons (minimum 44×44 pt hit targets) |

**Performance notes:**
- `touch-action: none` is set on the canvas.
- `{ desynchronized: true }` is passed to `getContext` to reduce latency.
- Device pixel ratio is respected for Retina displays.
- Particle count is capped and object-pooled to avoid GC jank.

> **iPadOS tip:** For 120 fps, go to Settings → Safari → disable
> "Prefer Page Rendering Updates near 60fps". Ensure Low Power Mode is off.
> If Apple Pencil input feels inconsistent, disable Scribble under Settings → Apple Pencil.

---

## Multiplayer (Room System)

The game uses **PeerJS** (WebRTC data channels) with a **host-authoritative** model.

- **Host:** Opens the game, taps **Start Game**. A room is created and listed in the
  public room directory (backed by a lightweight Supabase table, or a fallback
  localStorage-based directory for local testing).
- **Join:** Open the game on any device, see the room appear in the server list,
  and tap to join.
- **Reconnection:** If a client disconnects, PeerJS automatically attempts to
  re-establish the connection using the same peer ID.

> **Important:** For internet play, the host's iPad must be reachable. PeerJS
> handles NAT traversal via STUN/TURN. If both peers are behind symmetric NATs,
> a TURN server may be required (configure in the `PEER_CONFIG` object).

---

## Weapon System (Auto-Discovery)

Weapons are **data-driven**. To add a weapon:

1. Create `weapons/myweapon/`.
2. Add `myweapon.png` (the icon/model).
3. Add `script.js` exporting a weapon definition:

```js
// weapons/myweapon/script.js
export default {
  name: "My Weapon",
  type: "gun",          // "gun" or "sword"
  damage: 25,
  projectileSpeed: 900, // px/s (guns only)
  projectileColor: "#ffdd55",
  useDuration: 0,       // ms before the weapon is consumed (0 = instant)
  cooldown: 0           // ms between uses (0 = single use)
};
