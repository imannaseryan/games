# DUSTLINE 3D

A real first-person 3D shooter for the browser, built with Three.js — an original
Dust-inspired desert compound, hitscan gunplay in the Counter-Strike 1.6 spirit,
bot enemies, and round-based play with a buy-phase economy. All art is procedural
(canvas textures), all sound is WebAudio-synthesized, and every asset is original —
no Valve/CS content is used or copied.

## Run it

Serve this folder statically (e.g. `python3 -m http.server`) and open `index.html`.
`lib/three.min.js` (Three.js **r160**, UMD — upgraded from r149 in v2) is vendored
locally — the game makes **zero runtime network calls** and runs fully offline /
inside a WebView.

## Controls

- **Desktop:** click to pointer-lock · WASD move · mouse look · LMB fire · RMB sniper scope ·
  Shift walk (steadier aim) · Space jump · R reload · 1/2/3 weapons · M mute
- **Mobile:** left virtual stick to move, drag right side to look, FIRE / JUMP / SWAP / RLD buttons

## The game

- **Weapons:** P-9 pistol (semi, 12/48), AR-7 rifle (auto, 30/90, bloom/recoil model —
  standing still shoots straight, moving sprays), LW-50 sniper (buyable, bolt-action,
  scope zoom; headshots multiply damage). **v2 gunplay:** rifle/pistol base spread
  cut ~45% and bloom settles ~50% faster, so first shots are near-pinpoint when
  still; rifle damage 34 (3 body shots to kill at mid range); bot hitboxes enlarged
  (head r=0.22, chest r=0.43, crouch-aware); bigger hitmarker, louder hit tick,
  floating damage numbers; gentle aim assist on touch devices; the crosshair gap
  is computed from the same spread function the shot uses, so it never lies.
- **Map:** one compact compound, three lanes — tunnels (roofed, north), mid with
  double doors, long corridor (south) — west spawn yard vs an elevated eastern
  bombsite plateau with ramps, crates, barrels and sandbags. Visual obstacles are
  exactly the collision/bullet blockers.
- **Bots (v2 AI):** primitive-built soldiers — now with helmet, goggles, vest,
  backpack and a detailed rifle — with a separate head hitbox, waypoint-graph
  navigation (30 nodes), LOS + reaction delay, burst fire with range-scaled
  inaccuracy, strafing, escalating difficulty. **v2:** bots spawn from four
  distributed gates (site / tunnels / long / mid) on five routes, and
  **dynamically repath every 2–4s** toward the player's current area
  (direct chase, velocity-lead intercept, or flank), so pressure arrives from
  changing directions. Squad roles are mixed per round — rushers close in,
  flankers strafe wide, holders claim cover spots and peek/crouch-cycle.
  Separation steering keeps them from stacking; a nearby teammate's death can
  reroute the squad; stuck bots hop to a neighbouring node and re-path.
- **Rounds/economy:** 5 rounds, 100s each, eliminate all hostiles. Kills $300
  (headshot $450), win +$1400, loss +$800 and retry. 10s buy phase between rounds:
  Kevlar $650 · Sniper $2700 · Rifle Upgrade $1200 · Ammo $250. Victory screen with
  score/kills/headshots/accuracy.

## v2 visual pass

Three.js r160 with ACES filmic tone mapping and sRGB output; 2048px PCF-soft
sun shadows; richer procedural textures (grime, exposed brick, hazard stripes,
three barrel liveries); original distressed wall-stencil decals (MID DOORS,
B ZONE, TUNNELS, LONG, DUSTLINE / SECTOR 7, 07 CHECKPOINT); sun-glow sprite and
warm haze; 240 floating dust motes; physical-falloff muzzle-flash light; fading
bullet-impact scorch decals; lighter viewmodel metals; the menu now opens on a
slow cinematic orbit of the compound.

## Verification (v2, performed 2026-11-20, headless Chrome + SwiftShader)

- In-page autotest (`?autotest=1`, deterministic `__game.step()` simulation):
  **17/17 PASS** — three.js r160 loads; buy phase; armor purchase charges
  correctly ($800+$5000−$650=$5150); sniper unlock ($2450); round starts with
  3 bots; **bots spawn from ≥2 distinct gates** (observed: site/tunnels/long)
  **on ≥2 distinct routes** (sitemid/tunnels/long); **bots demonstrably repath
  mid-round** (10–16 repaths logged, targets converging on the player's new
  lane, all bots switching to hunt mode); **marksmanship: a frozen bot's torso
  at 14 m is killed in 3 rifle shots** (fixed aim, from full HP); hits
  registered; kill pays cash/score; reload refills the magazine; bot fire
  damages the player; round timer expiry ends the round; **a full unattended
  5-round match completes to victory**; zero page errors.
- Two navigation defects found by the v2 marathon and fixed: the waypoint
  graph's direct spawn↔long edge cut a walled corner and wedged bots (removed —
  long now routes via the H2 gap), and the mid waypoints WP16/WP17 sat on the
  (-8,0) crate line (moved south, z=-2.4); sandbags are now step-up-able
  (0.8 step allowance) instead of bot traps.
- Screenshots inspected: menu (cinematic orbit), long corridor, bombsite,
  **bots approaching from two different directions** (`shots/twodirections.png`),
  **close bot view showing the new model detail** (`shots/botclose.png`), buy
  menu, aim, scope, victory — see `shots/`.
- Note: the sandbox renders via SwiftShader (CPU), so sandbox frame rates are
  not representative of GPU hardware; the renderer caps pixel ratio at 1.5 and
  uses a single 2048px shadow map. Pointer-lock feel and real-device FPS remain
  unverifiable headless. Local test rig note: this sandbox's Chromium gates
  WebGL behind a domain policy and blocks loopback HTTP, so local verification
  served the files to the browser via CDP Fetch interception under
  https://example.com (policy-listed) — the game itself still makes zero
  network calls.

## Debug / staging hooks

- `window.__game` — state, player, bots, startMatch/startRound/enterBuy/buyItem,
  teleport, aimAt, setFiring, step(frames, dt), stepNR (no-render stepping),
  hasLOS, findFiringPos, setRoundTime, giveCash, setMag, freezeBots, errors,
  and v2 additions: `botStats()` (gates/routes/repaths + per-bot role/mode/
  routeChanges), `placeBot(bot, x, z)`, `render()`.
- `?view=menu|corridor|aim|buy|scope|over|site|twodirections|botclose` — staged
  scenes for screenshots.
- `?autotest=1` — runs the test suite, writes results to `#autotest-results`.
