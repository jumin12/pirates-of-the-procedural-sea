Original prompt: Using three.js as a single index.html file with amazing graphics create a multiplayer pirate game. The world should be randomly generated as the players explore it, the world should be mostly ocean with islands. The player can customize and build their own pirate ships with parts and items they find, the ships should be large enough to have a crew and there should be a crew with the camera being RTS like in a fixed position.

## Completed

- **Server (server.js)**: WebSocket multiplayer server with HTTP static file serving, player state sync at 20Hz, world seed sharing, ship customization broadcast, chat, cannonball sync
- **Water Shader**: Custom Gerstner wave vertex shader with 5 wave layers, Fresnel reflections, subsurface scattering, foam, specular highlights
- **Sky System**: Procedural sky dome with sun, clouds, horizon gradient
- **Procedural World**: Chunk-based generation using simplex noise, islands with multi-layer terrain (sand/grass/rock), palm trees with curved trunks and drooping leaves, rocks, collectible crates
- **Ship Builder**: 3 hull types (Sloop, Brigantine, Galleon), customizable hull material, sails, cannons, figurehead with iron cost system
- **Ship Rendering**: Extruded hull shape, deck, rails, stern cabin, animated sails with wind billow, pirate flag, figurehead
- **Crew System**: Crew members on deck with role-based positioning (captain at helm, gunner at cannons, sailor at sails), idle wandering animation
- **RTS Camera**: Fixed elevated angle, WASD ship control, arrow keys/middle-mouse pan, scroll zoom, camera gently returns to ship
- **Combat**: Cannon firing (spacebar), projectile physics with gravity, water splash particles on impact
- **Multiplayer**: WebSocket connection, other player ships rendered with interpolated position, chat system, cannon sync
- **UI**: HUD (coords, speed, health, wind), minimap with island/player markers, ship builder panel, inventory panel, crew panel, chat box, notifications
- **World Items**: Collectible crates on islands containing wood/cloth/iron/gold, proximity pickup

## TODOs / Suggestions for next agent

- Add ship-to-ship collision damage
- Island docking mechanic for better item collection
- Enemy NPC ships with AI patrol routes
- Trading system at island ports
- Ship health/repair system using collected materials
- More crew roles and crew recruitment at islands
- Weather system (storms, fog, rain)
- Day/night cycle
- Sound effects and ambient ocean audio
- Ship wake/trail particles behind moving ships

---

New prompt: As a single index.html file create a game that is similar to spore and the curious expedition mixed. It should be just the space stage of spore but focused around exploration and relic collection and the badges. It should be expanded upon and there should be planetary exploration that is like the curious expedition but the galaxy view and planet view should be 3d like spore. The planets and plants and buildings and everything should be procedurally generated similar to spore. Do this as a single index.html file

## Relic Spiral build notes

- Replaced the prior pirate game with a new single-file Three.js game in `index.html`.
- Added procedural 3D galaxy view with star selection, ship travel, fuel costs, planet orbit displays, scanning, refueling, archive selling, and landing.
- Added procedural 3D planet expedition view with axial hex movement, terrain costs, supplies, relic sites, ruins, hazards, supply pockets, outposts, and generated flora/buildings.
- Added badge progression for scans, relic recovery, expedition steps, flora cataloguing, ruins, system visits, and hazards survived.
- Added `window.render_game_to_text()` and `window.advanceTime(ms)` hooks for automated game testing.

## TODOs / Suggestions for next agent

- Add save/load slots using localStorage.
- Add more ship upgrades that affect fuel, scanner range, and expedition supply cost.
- Add animated landing and takeoff transitions between galaxy and planet view.

---

## 2026-09-07 networking / AI / gameplay hotfix

Fixed dedicated-server and multiplayer bugs that broke combat, quests, and ship sync:

- Merchant AI no longer throws on undefined `mRel`/`mWarJ` (could halt the entire NPC tick).
- Kill awards now carry hunt name + victim faction so letters of marque, hunt contracts, and standing apply on dedicated server / non-host clients.
- Death respawn no longer fights anticheat (server marks a respawn snap; client skips reconcile for 6s).
- Boarding updates survive rate-limits; first grapple frame uses boarding jump limits; only the sim authority mirrors PvP boarding.
- Server NPC AI: return fire on the shooting captain, aligned pirate/patrol hostility, `loading_home` coastal approach, NPC-vs-NPC broadside damage, unstuck/teleport, hull separation.
- Other: `npc_sync` tick dedupe, docked reconnect restore, PvP ram reports, softer false desync overlay.

## TODOs / Suggestions for next agent

- Authoritative PvP cannon hits (still victim-simulated).
- Share reef occupancy with the Node NPC sim so dedicated-server hulls cannot clip reefs.
- Port merchant stockpile economy onto the server when `npcSimFromServer` is on.

---

## 2026-09-07 follow-up: remaining dedicated-server / PvP gaps

- PvP cannon hits are now shooter-claimed and server-validated (`pvp_hit_claim` / `pvp_hit_apply`); victims no longer apply incoming peer balls locally while connected.
- Dedicated-server NPC nav uses the same reef occupancy walk as the client and can wreck on reefs.
- Merchant load/unload updates server town stockpiles and broadcasts `town_stockpiles` / `politics_snap.ts`.

## 2026-09-07 leftovers pass (harbor / reefs / troops / ping / sync)

- Harbor PvP truce is server-enforced on `pvp_hit_claim` and `pvp_ram_report` (both hulls within 50u of a dock).
- Reef collision discs now consume the same boulder + jitter RNG as the client (no more estimated radius).
- Dedicated-server merchants can spawn as troop transports; dock_dest runs `applyTroopLanding` and broadcasts `politics_snap` (garrison / controller flip).
- Remote hull interp scales with RTT (up to 180ms). Server rewinds victim/attacker history using reported RTT before accepting cannon claims.
- `npc_sync` is 20 Hz (every 3 ticks). Player `state` stays 60 Hz. Default `STATE_AOI_RADIUS` is 3200.
- Playtest: new voyage → dedicated-server NPC roster (troop transports present) → dock at Île de Ré Downs (harbor truce + town stock) → reconnect stays docked. Asset URLs with spaces (`3d models`) are now decoded so local GLBs can be served.

## TODOs / Suggestions for next agent

- Smoothness still depends on machine, player count, and GLB load. No full load-test at 20+ captains.
- High ping can still look wrong; rewind + wider interp reduces “I hit / you didn’t,” it does not remove lag.
- Full live PvP/boarding/quest session still needs a second human client. Automated playtest covers sail / dock-proximity / reconnect hooks, not two-player combat.
- If `buildSeaReefSpireGroup` fails on the client, that reef is skipped visually but the server still has a disc.

## 2026-09-07 performance pass

- Default graphics is **Medium** (Settings → Graphics). High is opt-in.
- Medium/Low: no shadow pass, lower pixel ratio, coarser water, shorter view distance.
- Per-frame: only nearby NPC sails/crew animate, chunk rebuilds only on chunk change, net drain capped, sailing updates send at 20 Hz, minimap every 4 frames, skip a WebGL frame when already under 30 fps.
