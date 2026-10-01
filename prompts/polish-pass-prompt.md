# POLISH & CONTENT PASS: "Ski As Far As You Can"

You built this game from my original design spec, and it has had many revisions since. The core loop works. The game does **not** yet look or feel like a polished public Roblox experience. I playtested it and attached 17 screenshots. Below is everything that's wrong, what I want instead, and how to tell when it's fixed.

The original spec still applies: low-poly, skill-based, endless downhill, fair hazards, and good performance on mobile. This pass is about **quality, feel and variety**, not new meta systems.

---

## 0. GROUND RULES FOR THIS PASS

1. **Inspect before you touch anything.** Scan the place and scripts first. Write a short list of the current systems and which ones each fix below will affect.
2. **Checkpoint first.** Save a copy of the place before each major section. Don't delete working systems (data, economy, crates logic, chunk streaming, floating origin). Fix or replace their *visuals and feel*, not their architecture, unless a fix needs that.
3. **Work in the priority order in section 1.** Finish and playtest each section before starting the next.
4. **Verify everything visually.** After each section, playtest, take screenshots from the same angles as mine, compare, and fix what still looks wrong. Don't tell me something is fixed unless you've seen it in a playtest.
5. **"Low-poly" does not mean "made of default blocks."** I want *stylized low-poly with some realism*: correct proportions, bevels and chamfers, wedges and cylinders instead of only cubes, believable materials, subtle color gradients, and objects that sit *in* the world (snow piled at their base, snow on top). Use Blender for anything that is currently a pile of stacked Parts.
6. **At the end, report back** with before/after screenshots for every item, a list of what changed, any performance numbers (FPS, triangle counts) and anything you couldn't fix.

---

## 1. PRIORITY ORDER

1. **Smoothness and jitter** (the game feels glitchy, which ruins everything else)
2. **Slope gameplay variety and new obstacles** (my main concern: runs feel the same)
3. **Clipping and grounding** (everywhere: lobby and slope)
4. **Model quality**: crates, penguins, props, trees, rocks, crystals, characters and suits
5. **Lobby**: roof, edge of the map, layout
6. **UI**: all of it, including the countdown lights and the trail board

---

## 2. SMOOTHNESS / JITTER: FIX THIS FIRST

**Problem:** Sliding down, the character visibly jitters or stutters instead of moving smoothly. Penguins moving around the lobby stutter. Mouse and camera movement feels choppy. The game feels "glitchy."

**Investigate these likely causes, measure, and fix them. Don't guess:**

- **Network ownership:** The skier must be simulated on the **client** (the player's own network ownership), not moved by the server. If the server sets the character's CFrame or velocity every tick, that causes exactly this stutter. The server should only *validate* (distance and speed sanity checks), never drive the movement.
- **Wrong update step:** Camera code must run in `RunService:BindToRenderStep` at camera priority (or `RenderStepped`), not `Heartbeat` or `Stepped`. Movement forces belong in `PreSimulation`/`Stepped`. Mixing these up causes judder.
- **Setting CFrame against physics:** Don't set the character's CFrame directly each frame while physics is also moving it. Use constraints (`LinearVelocity`, `AlignOrientation`, `VectorForce`) or one consistent method.
- **Floating-origin recentering:** Confirm the shift happens in one frame and moves the camera with everything else. If there's a visible "pop" or hitch when it triggers, fix it.
- **Chunk spawning hitches:** Spreading chunk generation over several frames (task.defer / budgeted generation) and pooling objects instead of `Instance.new` + `Destroy` should remove periodic frame spikes.
- **NPC movement (penguins):** Animate or tween them **on the client**, with interpolation, not by stepping CFrame on the server in a loop.
- **Mesh and collision cost:** Check triangle counts. Decorative meshes need `CollisionFidelity = Box` or `CanCollide/CanQuery/CanTouch = false`. Hazards should use simple hitboxes. Nothing decorative should cost thousands of triangles (see the budget in section 5).
- **Too many Heartbeat loops:** Merge per-object loops into one manager loop per system.

**Done when:** A 3+ minute run at high speed is visually smooth. The MicroProfiler shows no regular spikes from chunk spawning, recentering or NPCs. Penguins move smoothly. The camera is smooth when turning with the mouse.

---

## 3. SLOPE GAMEPLAY: MORE ACTION, MORE VARIETY (MAIN CONCERN)

**Problem:** Runs feel very similar. Most of the slope is an open, empty white plane with scattered identical ice crystals, gray rock blobs, coin lines and the odd orange pole (see screenshots 9–12). Ramps don't have a *purpose*. Many objects clip into the snow. There's no sense of "oh no, what's that ahead?"

### 3A. Design rule: "set pieces" and rhythm

- The run should be paced in **beats**. Roughly every **150–300m** the player meets a **set piece**: a memorable, readable obstacle with a clear solution (go around, go over with a ramp, go under, pick a route). Between set pieces, use filler (trees, rocks, moguls, coins) with **variable density**. No more long empty planes.
- **Ramps must have a purpose.** Every ramp should do one of these: clear a gap or ravine, get over a blocker (car, fallen tree, cabin), reach a higher route with better rewards, or launch through a coin or gold ring. Ramps on flat empty snow with nothing to clear are pointless.
- **Split routes:** Often offer 2–3 lanes with different risk and reward: a safe slow lane, a risky lane with more coins and near misses, and a hidden or ramp-only lane with a big reward.
- **No repetition:** Don't use the same set piece twice within ~800m, or the same chunk archetype twice in a row.
- **Slope shape variety:** The terrain itself should vary: banked turns, rolling hills, moguls, narrow ridges with drops on both sides, wide bowls, steep drops that boost speed, gentle flats before big set pieces. Not one flat tilted plane.

### 3B. New obstacles and set pieces (build as many as possible, all modular in the hazard system)

**Blockers you need a ramp or route choice for:**
- **Abandoned car / snowed-in truck.** A half-buried rusty car or pickup across the main lane, with snow piled on the roof and around the wheels. A snow ramp lets you jump onto the roof and off the far side, or you squeeze through a narrow gap at the edge.
- **Crashed snowcat / snow-groomer** on its side across the slope. Its tread forms a ramp you can ride up.
- **Fallen chairlift.** Chairs and a snapped cable across the path. Duck through the gap under the cable (tuck), or jump over the lowest chair with a ramp.
- **Fallen giant pine log** across the slope. Low enough to hop at low difficulty. Higher logs need a ramp. Some logs form a natural bridge over a ravine.
- **Snow-covered cabin** in the path. Go around it, or hit the snow drift against its wall and ride over the roof.
- **Ski-patrol barricade** (orange net fences with signs) forcing a route change. These already sort of exist, so make them look good.

**Moving hazards:**
- **Rolling snowballs** that look good: actually round, with snow texture, rolling with real rotation, leaving a trail of compressed snow and small powder puffs. Some **grow as they roll**. Early ones are small and slow. Later ones are big, come down side gullies, and bounce off terrain.
- **Snowmobile / ski-patrol crossing.** A snowmobile crosses the slope left to right, with headlight and engine sound warning you before it appears.
- **Wildlife crossing.** A moose or elk herd, or a group of penguins sliding on their bellies across the run (funny and readable).
- **Swinging chairlift chairs.** Chairs hang low over the run and sway in the wind. Time your pass.
- **Snow plow / avalanche-control truck** pushing a wall of snow across.

**Terrain hazards:**
- **Frozen lake.** The ice visibly cracks under you (cracks spread from your path). Keep moving or fall through. Speed boosts can help.
- **Collapsing snow bridge over a crevasse.** The bridge crumbles section by section behind and ahead of you.
- **Ravine with a required ramp**, with a visible far edge and a landing marked by flags so the player knows it's possible.
- **Half-pipe / ice chute sections:** curved walls you can carve up for coins.
- **Mogul field:** bumpy terrain that slightly bounces the player and rewards clean lines.
- **Cliff drop-offs** with a safe line marked by flags or poles.

**Projectile and falling hazards:**
- **Snow cannons** (resort snowmaking guns) that visibly swivel and charge, then fire snowballs across the slope.
- **Falling icicles in caves** (see section 4D for how these must work).
- **Rockfall** from a cliff wall: a shadow plus small debris warns you first.
- **Tree falling across the slope** as you approach (it creaks and tilts, then falls).

**Rewards and fun:**
- **Gold rings** in the air after ramps: fly through for a coin and adrenaline bonus.
- **Rails / grind boxes:** fallen fences or pipes you can slide along for combo points.
- **Snowman fields:** smash through snowmen for small bonuses, with a fun burst VFX. They're harmless, just satisfying.
- **Ski-jump set piece:** occasionally a proper large ski-jump ramp with a long flight, a camera moment and a landing zone.
- **Ski-through tunnels:** a snow tunnel or an old mine tunnel in the ice-cave biome with lanterns, and maybe minecart tracks with a cart crossing.

**Biome-specific set pieces** (make each biome feel different, not just recolored):
- **Alpine Resort:** chairlift towers, snow cannons, ski-patrol huts, slalom flags, abandoned cars near the road.
- **Snow Forest:** dense tree slaloms, fallen logs, wildlife, cabins, rolling snowballs from gullies.
- **Glacier:** crevasses, ice bridges, sliding ice blocks, frozen lakes.
- **Blizzard:** reduced visibility *with fair warning* (glowing hazard markers, flags), wind gusts pushing you sideways.
- **Ice Caves:** stalactites, falling icicles, narrow tunnels, glowing crystals, minecart remains.
- **Extreme Peaks:** narrow ridges, cliff drops, rockfall, big ski jumps.

### 3C. Grounding on the slope

- **Nothing may float or be half-sunk at random.** Every prop and hazard must be raycast to the terrain surface, aligned to the slope normal where that makes sense (rocks, cars, logs), and given a **snow skirt/drift** mesh at its base so it looks settled in snow. Trees stay vertical but get a snow mound at the trunk.
- Rocks and crystals need **variety**: at least 4–6 shape variants each, random scale (0.7–1.5×), random rotation and slight color variation. Identical clones in a row (screenshot: crystal clusters) look cheap.

**Done when:** I can do three runs to 3,000m and each one feels different. I meet at least 10 different set pieces. Every ramp clears something. Nothing floats or clips into the snow.

---

## 4. SPECIFIC FIXES FROM MY SCREENSHOTS

### 4A. Clipping everywhere (lobby and slope)
**Problem:** Trees clip into ledges, walls, fences and buildings. Props sit on top of each other. A crate clips into the lobby building. Trees go into random things.

**Fix:**
- Write a **one-time Studio command / audit script** that lists every overlapping pair of decorative and structural parts in the lobby (using `GetPartsInPart` / bounding-box overlap). Fix each one by moving it, not by turning off collisions.
- Keep trees **≥ 6 studs** from walls, fences, buildings, paths and each other. Trees should be placed by a spacing rule (Poisson-disk style), not purely random positions.
- Add the same overlap check to the **chunk validator** so procedural decorations never spawn inside hazards, ramps, coins or each other.

### 4B. Lobby edge / "wall" and no roof
**Problem:** The lobby ends in a hard flat slab or wall (screenshot 3). It looks cut off, and trees poke into it. One building has **no roof**, and a crate glitches into its wall (screenshot 8).

**Fix:**
- Replace the hard edge with a **natural boundary**: sloped snow terrain or hills rising away from the lobby, rock outcrops, a tree line set back from the edge, and distant mountains. Use `Atmosphere` haze and fog so the distance fades softly. Use an invisible collision wall if needed, but the player should never *see* where the map ends.
- Give **every building a proper roof**: pitched roof with snow on top, overhang, chimney with smoke on the chalet, warm window light. Check every building from above.
- Move the crate out of the building and fix the crate area layout (see 4E).

### 4C. Penguins
**Problem:** They don't look like penguins (screenshot 2). The body is faceted and glitchy, the scarf is a floating stack of blocks, a flipper is detached, and the white belly is jagged patches.

**Fix:** Rebuild in Blender as **one clean low-poly mesh** (~500–1,500 triangles):
- Smooth teardrop/egg-shaped black body with a **clean oval white belly**, rounded head, small shiny eyes, orange beak and webbed orange feet, flippers *attached* at the shoulders.
- Scarf as part of the mesh or a properly fitted accessory, not loose blocks.
- Rig it simply (body, head, flippers, feet) and give it **waddle, idle look-around, flap and belly-slide** animations. Run animation on the client.
- Fix whatever is causing the body to "glitch" (likely overlapping parts, bad welds or a server-side CFrame loop).

### 4D. Falling crystals / icicles in the ice cave
**Problem:** Crystals don't actually drop *from* the ceiling, aren't connected to it, all look the same size, and fall linearly (screenshot 13).

**Fix:**
- Every icicle must **hang from the actual cave ceiling**: raycast up to find the ceiling, attach the icicle's base to it, with a small ice cluster at the base so the join looks natural.
- **Sizes vary:** 0.6×–1.8× scale, different lengths and thicknesses. Small ones are fast and harmless-looking. Big ones are slower to drop but deadlier.
- **Telegraph:** the icicle **shakes/wiggles**, small ice particles trickle down, a crack sound plays, and a **growing shadow/decal** appears on the floor where it will land. Give at least ~0.8–1.2s of warning at current speed.
- **Falling motion:** accelerate like gravity (ease-in, e.g. `Quad`/`Cubic` In, or real physics), with a slight tilt or wobble as it falls. No linear movement.
- **Impact:** shatter into ice shards with VFX and sound. The shards fade after a moment.

### 4E. Crates
**Problem:** The crates look nothing like crates (screenshots 7–8). They're flat boxes with colored straps that look like gifts. I like the **shrine/pedestal idea**, so keep that. The crates themselves need to be much higher quality.

**Fix:**
- Model real crates in Blender, one per tier, each clearly more premium than the last:
  - **Snow Crate:** wooden plank crate with visible planks, metal corner brackets, rope handles, a frosty snow lid.
  - **Alpine Crate:** reinforced dark wood with iron bands, a lock/latch and a carved mountain emblem. Blue accent glow.
  - **Summit Crate:** ornate chest with gold trim and glowing ice crystals growing from the lid. Purple/gold glow, floating slightly with sparkle particles.
- Keep the **shrine**: stone or ice pedestal, rarity-colored light, a nameplate with crate name and ticket cost, and slow rotation or bob.
- **Opening sequence:** the lid shakes, unlatches, opens on a hinge, light bursts out, and the item reveal is colored by rarity. Show odds clearly on the shrine UI.
- Make sure nothing in the crate area clips into anything else.

### 4F. Props that don't read as what they are
**Problem:** The chair (screenshot 5) is a few stacked red blocks. Some items face the wrong way entirely: the **snowboards/skis** leaning against the wall are rotated wrong and clip into snow mounds (screenshot 6).

**Fix:**
- Rebuild lobby props in Blender so they're recognizable at a glance: **Adirondack chair** (slatted, angled back, wide armrests, real wood material), benches, lamp posts, signposts, ski racks, fire pit, picnic tables.
- **Snowboards and skis** go in a proper **ski rack** or lean against a wall at a believable angle, with their bindings facing out and their bottoms against the wall. Check the orientation of *every* prop.
- Add a **"Does it read?" check:** if someone would need to be told what an object is, rebuild it.

### 4G. Characters, ski suits and name tags
**Problem:** The suits look ugly (screenshot 14). The display mannequins are blocky R6 bodies with default smiley faces, and the suits look like flat textures on blocks. The player character is also forced into a blocky look. **Name tags clip into the ground**, so you can't read them.

**Fix:**
- **Respect the player's own avatar** (R15). Don't force a blocky rig.
- Rebuild suits as proper **layered clothing / accessories** (jacket, pants, helmet, goggles), with good colors, seams, zippers and color-blocking like real ski gear. Think real ski brands, stylized.
- The suit display should use proper R15 mannequins on nice pedestals, with no default smiley faces. Use a face-covering helmet + goggles or a neutral mannequin head.
- **Name tags:** BillboardGui on the **head**, `StudsOffset ≈ (0, 2.5–3, 0)`, `AlwaysOnTop = true`, sensible `MaxDistance`. They must never go below the ground or clip into the player.

### 4H. Trees, rocks, crystals
- Trees: at least **4–6 pine variants** (different heights, layer counts, snow amounts), plus some dead and bare trees in later biomes. Some snow on the branches. Don't let identical clones line up.
- Rocks (screenshot 10): currently gray blobs. Make them faceted with **snow caps on top**, color variation, and moss or ice in some biomes.
- Crystals (screenshot 9): more variation in cluster shapes, sizes, colors and glow. Fewer copy-pasted rows.

---

## 5. ART QUALITY STANDARD (APPLIES TO EVERYTHING)

- **Style target:** stylized low-poly with **some realism**. Correct proportions, bevels on hard edges, believable materials, flat-shaded faces with subtle gradients. Think "polished indie low-poly," not "default Roblox parts."
- **Triangle budgets** (keep the game fast):
  - Small props (chair, sign, coin): **≤ 500 tris**
  - Medium props (crate, penguin, rock, tree): **≤ 1,500 tris**
  - Large set pieces (car, snowcat, cabin, chairlift tower): **≤ 4,000 tris**
  - Nothing decorative should be over ~5k. Use LOD / `RenderFidelity = Automatic`.
- **Collision:** Decorative meshes use `CollisionFidelity = Box` or no collision. Hazards use simple invisible hitboxes.
- **Grounding:** Everything sits *in* the snow with a drift or skirt at its base. Nothing floats, sinks randomly or clips.
- **Consistency:** One shared color palette, consistent material use and one lighting setup per biome.

---

## 6. UI: REDESIGN ALL OF IT

**Problem:** The UI doesn't look good. Specifics:
- **Daily Spin** (screenshot 1): the title "DAILY SPIN" is **clipped off the left edge**, text is tiny, and it's a flat list on a dark panel. **There's no actual wheel.** The "Spin the Wheel!" button is cut off.
- **Trail & Challenges board** (screenshot 15): the "YOU ARE HERE" mountain is a pixelated staircase with crude colored lines, and the challenge text ("Ski to unlock challenge…") is cut off.
- **Countdown lights** (screenshot 17): all three lights are lit at once. They should light up *in sequence*. The housing is plain squares. The title text behind them overlaps.
- HUD elements (Best Run, Trail Map prompt) are inconsistent with each other.

**Fix: create one UI design system and apply it everywhere:**
- **Style:** rounded and chunky (`UICorner`), soft `UIStroke` outlines, a subtle drop shadow, one font family (e.g. `GothamBold`/`Montserrat` for headers and `Gotham` for body), and a consistent snowy palette: white/ice-blue panels, navy text, and bright accents (orange for primary actions, gold for currency, rarity colors).
- **Scale-based sizing** + `UIAspectRatioConstraint` + `UIScale` so nothing clips on any screen. **Test on phone, tablet and desktop resolutions** in the device emulator and screenshot each.
- **Animations:** buttons scale on hover/press, panels slide or pop in with easing, numbers count up.
- **Daily Spin:** build an **actual spinning wheel**. It should have colored segments with icons, a pointer, spin with ease-out deceleration, tick sounds, a reward pop-up, and odds listed below. The title must be fully visible.
- **Trail & Challenges board:** redesign as a clean illustrated low-poly mountain map with a smooth route line, a pulsing "You are here" marker, and readable challenge cards with an icon, description, progress bar and reward. No cut-off text.
- **Countdown lights:** a proper start-light gantry (physical in the world and/or UI). The lights start **dark/dim**. **Red lights at 3, yellow at 2, green at GO**, each with a glow (Neon + PointLight / UI glow), a beep, and a big pop-in number. All lights reset when the run starts. No overlapping title text.
- **HUD:** keep it minimal and consistent: distance + best at top center, speed on the left, coins on the right, combo messages at center, boost meter at the bottom. Use the same style as the lobby UI.

---

## 7. ACCEPTANCE CHECKLIST (verify each in a playtest, with screenshots)

- [ ] Skiing is smooth at all speeds. No stutter from recentering, chunk spawns or NPCs. The camera is smooth.
- [ ] 3 different runs to 3,000m feel clearly different. At least 10 distinct set pieces have been met.
- [ ] Every ramp has a purpose. There's at least one abandoned-vehicle ramp set piece.
- [ ] Snowballs are round, roll properly, look good and are telegraphed.
- [ ] Ice-cave icicles hang from the ceiling, vary in size, shake before falling, cast a shadow, fall with acceleration and shatter.
- [ ] Zero overlapping or clipping objects in the lobby (audit script output = 0). Nothing floats or sinks on the slope.
- [ ] The lobby has a natural edge with no visible wall. Every building has a roof.
- [ ] Penguins look like penguins, are one clean mesh and animate smoothly.
- [ ] Crates look like real tiered crates on shrines and have an opening animation.
- [ ] The chair and all lobby props are recognizable. Snowboards and skis face the right way.
- [ ] Players keep their R15 avatar. Suits look like real stylized ski gear. Name tags are above the head and readable.
- [ ] The Daily Spin has a real wheel. No UI text is clipped on phone, tablet or desktop.
- [ ] Countdown lights light up in sequence red → yellow → green.
- [ ] The trail board is clean and readable.
- [ ] Triangle budgets are respected. There are no console errors. FPS is stable.

When you're done, send me before/after screenshots for each item, a summary of the changes, performance numbers, and anything that still needs work.
