# REVISION 2: 30-LEVEL CAMPAIGN, REAL ART QUALITY, AND A FULL UI REDO

You revised the game after my last prompt. It's better and it doesn't look bad, but a lot is still wrong, and **some things I already asked you to fix are still broken**, like the signs hidden under the lodge. I've attached new screenshots.

The main problem is **awareness**: things get placed upside down, facing the wrong way, inside other objects, under buildings, or into the snow, and nobody notices. Most of this prompt is about fixing that, and about building **checks** so it can't keep happening.

I don't care how long this takes. Work in phases, keep notes, and check, double-check and triple-check your own work.

## 🧭 YOU ARE A NEW CHAT: GET ORIENTED FIRST

You have no memory of earlier sessions. This game (**Powder Rush**, an endless low-poly skiing game) was built over many earlier sessions by another Claude chat using Ropilot (Roblox Studio) and Blender. Before changing anything:

1. **Read the handoff document.** The full `PROJECT_SUMMARY.md` from the end of the last session is pasted at the bottom of this prompt (**Appendix A**). It covers where every file lives, the tooling quirks (Ropilot commit workflow, Blender axis mapping, edit-mode screenshots), the architecture, every module, the QA tools and the open items. Where it disagrees with older notes, the summary wins. **Where it disagrees with what I say in this prompt, I win.**
2. **Then read from disk:** `alpine-ski/PROJECT_SUMMARY.md` (in case it's newer than the appendix), `alpine-ski/PROGRESS.md` (the full pass-by-pass history), and `alpine-ski/GAME_SPEC.md` (the original design brief, which this prompt partly overrides; see section 3).
3. **Explore the live place and scripts** with Ropilot to confirm the summary matches reality. Don't trust it blindly (see the next section).
4. **Write a short orientation note** at the top of a new "Revision 2" section in `PROGRESS.md`: what you found, what doesn't match the summary, and your plan for phase 0.
5. Keep logging in `PROGRESS.md`. **Keep `PROJECT_SUMMARY.md` updated at the end of every phase**, so the next new chat can pick up from it the same way you did.

**Owner facts:**
- The game is published under my **Roblox group, "Jack's Motion."** Upload all animations (and any other assets that must be owned by the experience owner) **to the Jack's Motion group, not a personal account.** This is very likely why uploaded animations were "rejected for playback" before. If you can't upload to the group yourself, give me exact step-by-step instructions and I'll do it, then you wire the IDs in.
- **Desktop first, mobile later** (from earlier notes). The UI must still not cut off at any size.

### ⚠ The project folder has moved: use `~/Projects/alpine-ski/`

The summary says the project lives in `/private/var/folders/.../T/alpine-ski/`. That's the macOS temporary folder, which the system can clear. **I've already copied it to `~/Projects/alpine-ski/`** (`/Users/jackattack/Projects/alpine-ski/`). From now on, that's the only project folder. Never put it in `~/Documents` or `~/Desktop`: those sync to iCloud on my Mac.
- **Check the copy is complete:** compare the file count and total size of the temp original and `~/Projects/alpine-ski/` (especially `assets/resort/Resort.blend`, `checkpoints/` and `assets/`). Tell me if anything is missing. Don't delete the temp original; I'll do that once you confirm.
- Update every path in `PROJECT_SUMMARY.md` and `PROGRESS.md` to `~/Projects/alpine-ski/`.
- Blender: open `Resort.blend` from the new location from now on (`bpy.ops.wm.open_mainfile` with the new path).
- Leave Ropilot's own sync folder (`ropilot-src`) where Ropilot expects it, but make sure the latest script checkpoint is also copied into the permanent folder.

---

## ⚠ READ THIS FIRST: PASS 9 SAYS "DONE," BUT I CAN STILL SEE THESE BUGS

`PROJECT_SUMMARY.md` marks many things as fixed and verified. **In my own playtest, these are still broken:**
- The character still **jitters** while skiing.
- **Clipping** is still there.
- **Icicles** are still upside down / look wrong.
- **Snowmen** still face the wrong way: **both** the lobby snowmen **and** the slope (runtime-built) snowmen.
- Models still look weak, and the UI still looks bad.

So **treat every "verified" claim in the summary as unverified until you re-check it**, and figure out *why* your checks missed these. From reading your own notes, I think these are the reasons:

1. **You mostly verify with edit-mode screenshots and numeric/static audits, not with real play.** Your notes say live playtest screenshots time out and Studio runs at ~1–2 fps when its window is unfocused. That means you have **never actually seen** smooth or jittery skiing, the live countdown, or the runtime-built slope objects in motion. Static checks passing doesn't mean the game looks right.
2. **Runtime-built objects aren't the same as lobby objects.** Icicles, slope snowmen, ramps and set pieces are built by `WorldRenderer` at runtime. Fixing a lobby model or an asset in `Assets` doesn't prove the runtime version is correct. Screenshot **the runtime-built version** (e.g. with `SetPiecePreview`, or by building real chunks in edit mode) from player height.
3. **The Blender axis convention** (Blender +Y = model front = Roblox −Z; Blender Z+260 = Roblox Y) is a very likely source of the upside-down and backwards objects. Check every asset's real orientation in Roblox. Don't assume the export got it right.

**New verification rules for this revision:**
- **Every fix needs a screenshot of the actual in-game object** from the player's point of view, plus the audit numbers. An audit alone is not proof.
- **Write a `TEST_CHECKLIST.md` for me** at the end of every phase: a short list of exactly what I should check in a focused playtest ("ski into the ice cave in level 6 and look at the icicles"). **I'll do the live checks with Studio focused** and report back. Until I confirm something, mark it "awaiting owner check," not "done."
- For things you can measure, add **telemetry** I can read after a playtest (e.g. the jitter numbers below).

### Codebase-specific causes to investigate (don't skip these)

- **Jitter.** Pass 9 moved the skier to client ownership with an interpolated view and correction smoothing, but it still jitters. Check these specifically:
  - **Server corrections:** log how often a snapshot correction is applied and how large it is (studs). If the client prediction and the server simulation disagree often, every correction is a visible snap. Find out why they diverge (different inputs, different dt, float order, unsynced hazard timing) and fix the cause.
  - **Physics vs. CFrame fighting:** the root is unanchored, PlatformStand and network-owned by the client. If code also sets its CFrame every frame, physics and the CFrame writes fight each other. Pick one: either anchor it and drive it purely by CFrame on the client, or drive it purely with constraints.
  - **Poses via part CFrames:** your notes say poses are applied by rotating part CFrames because `Motor6D.C0` is read-only on generated rigs. Rewriting limb CFrames every frame against Motor6Ds can make limbs shake. Use `Motor6D.Transform` (that's what the Animator writes) in `PreSimulation`/`Stepped`, or real Animations (see below).
  - **Camera:** make sure the camera follows the *interpolated* view, updated in `BindToRenderStep` at camera priority, after the character is placed in the same frame.
  - **Hitches:** your telemetry showed build jobs up to ~11 ms (budget 4 ms), a recentre of ~36 ms and streaming peaks of ~42 ms. Those are visible stutters. Get every one under ~4 ms per frame.
  - Add telemetry for: correction count/size per minute, frame time spikes > 20 ms, and their cause. I'll run a focused playtest and send you the numbers.
- **Animations.** Your notes say uploaded animations were "rejected for playback," so poses are bundled as code. That usually happens because **an animation must be owned by the same account or group that owns the experience.** The experience is owned by the **"Jack's Motion" group**, so re-upload the animations under that group. If you can't, give me the exact steps and I'll do it. Real Animator-driven animations should look and blend much better than code poses.
- **Ramps.** Your notes say the visual ramp profile "matches the simulation's linear profile." A straight linear ramp barely launches you, which is exactly my complaint ("you go up, then straight back down"). Change the simulation and the visuals to a **curved kicker** with a real upward launch (see section 4A).
- **Borders.** Your notes say skiing more than 11 studs past the edge buries you in the bank (a crash), and the fences are only visual. So players can pass the fences. The **collision boundary must match the fence line**, and it should bump the player back, not crash them (see section 4B).
- **Mountain pop-in before Level 2.** Candidates: the backdrop that `WorldRenderer` re-centres on the camera, theme set dressing (`WorldRenderer:Extra`) spawning at the level change, a far chunk built inside the view distance, or StreamingEnabled loading a lobby/Summit mesh. Find the actual one and prove it's gone.
- **Lobby flicker (z-fighting).** `LobbyAudit` only checks overlaps (decor vs. structure). It **does not check coplanar faces**, which is what causes the flickering stud textures. Add that check (section 1).
- **Summit Express signs under the lodge.** Check the Chairlift and Lodge folders. Find every sign and screenshot it.
- **Quests.** `QuestService` gives 3 daily (4 VIP) and 2 weekly. Change this per section 7.
- **Backups.** Your notes say the newest **full place backup (.rbxl) is from Phase 1.** Before anything else, tell me to save one (File → Save to File), or save one yourself if you can. Then save one at the end of every phase.

---

## 0. HOW TO WORK ON THIS (READ FIRST)

1. **Keep a progress file.** Create `PROGRESS.md` in the project folder. After each phase, write down what you did, what you verified (with screenshot names), what's still broken, and what's next. If this session runs out of context or a new session starts, read `PROGRESS.md` first and continue from there. Never redo finished work, and never assume something is done without checking it.
2. **Checkpoint before each phase.** Save a copy of the place file before changing anything big.
3. **Inspect before building.** Before each phase, look at what currently exists (scripts, models, positions) and write a short plan.
4. **Self-review loop for EVERY change.** This is required:
   - Build it.
   - Playtest it as a player: actually ski and walk around.
   - Take screenshots from **several angles**: player height, from above, from the side, and up close.
   - Go through the "Awareness Checklist" (section 1) for every object in view.
   - Compare against reference photos (section 2).
   - Fix, then screenshot again. Only mark something done in `PROGRESS.md` when the after screenshot proves it.
5. **Never report something as fixed that you haven't seen fixed in a playtest.** If you couldn't fix something, say so clearly.
6. **Don't break working systems.** Data saving, currencies, crates logic, the inventory and the chunk generator should keep working. Change how things *look and play*, and restructure only where this prompt asks (the level system in section 3).

> **Things I want removed:** [ADD HERE — list anything you want deleted]
> **Things I want kept exactly as they are:** [ADD HERE]

---

## 1. AWARENESS CHECKLIST + AUDIT TOOLS (BUILD THESE FIRST)

Most bugs I keep reporting are the same kind of mistake. Build **automated audit commands** (Studio command bar scripts or a dev-only module) and run them after every phase. Report the counts in `PROGRESS.md`. They must reach **0** (or each remaining case explained).

**Audit scripts to build:**
- **Z-fighting / flicker audit:** Find parts that overlap with **coplanar faces**, meaning two surfaces at exactly the same height or position. This is what causes the flickering stud textures when I walk around the lobby. Fix by deleting duplicate parts, merging them, or offsetting one by ~0.02–0.05 studs. Also check that no two parts with the same texture share a face plane.
- **Overlap audit:** Find decorative or structural objects intersecting each other (trees in walls, crates in buildings, props in fences).
- **Hidden object audit:** Find signs, props and decorations that are **inside or underneath** other objects (like the *Summit Express* signs hidden under the lodge). Raycast from each object toward open sky and from the player walking area. If it can't be seen, it's in the wrong place.
- **Grounding audit:** Raycast down from every prop and tree. Report anything floating (> 0.2 studs above ground) or sunk too deep. **Trees:** most trees should have their trunk base *on* the snow surface, with a small snow mound around it. Only about 1 in 5 should be slightly buried, and only by a small amount, never to the branches.
- **Orientation audit:** For every object type that has a "front" or an "up," check that it faces the right way:
  - **Icicles:** wide end attached to the ceiling, sharp tip pointing **down**. They're currently upside down.
  - **Snowmen:** face **toward the path/player route**, not away from it or sideways.
  - **Signs:** readable text facing the direction players approach from.
  - **Skis and snowboards:** right side up, in racks or leaning naturally.
  - **Ramps:** lip faces downhill and kicks **up**.
- **UI audit:** Loop over every `TextLabel` / `TextButton` and check `TextFits`. Check every GUI element's `AbsolutePosition`/`AbsoluteSize` stays inside the screen. Run it at phone (e.g. 667×375), tablet (1024×768) and desktop (1920×1080) sizes. **Zero cut-off text is required.**

**Manual checks** (ask yourself these for every object in a screenshot):
- Is it the right way up? Is it facing the right way?
- Is it touching the ground correctly, not floating and not sunk?
- Is it inside or overlapping anything?
- Can players actually see it? Is anything hiding it?
- Would someone recognize what it is without being told?
- Does it match the reference photo and the rest of the art style?

---

## 2. ART QUALITY: USE REFERENCES

Many of the models you make don't look good. From now on:

1. **Find a reference photo before modeling anything.** Use web search if you have it, or describe the real-world object in detail from your knowledge first. Note its proportions, silhouette, key features and colors. Then model it and **compare a screenshot side by side** with the reference. If it doesn't clearly read as the same object, redo it.
2. **Style target: stylized low-poly with realism.** Low-poly is fine, but proportions, silhouettes and details must be believable. Use bevels, wedges, cylinders and real materials (Wood, WoodPlanks, Slate, Ice, Metal, Fabric), with subtle color variation. Avoid "a cube with a roof." Use Blender for anything complex if it's available.
3. **Using Toolbox / Creator Store models is allowed and encouraged** where they look better than what you'd build, especially **penguins**. Rules for free models:
   - **Delete every script inside** (free models can contain malicious scripts or backdoors). Check for hidden scripts, `require()` calls and weird modules.
   - Check the triangle count (budgets below), fix the scale, and make it match our art style and colors.
   - Pick models that look good and are consistent with each other. No mixing realistic and super-cartoony.
4. **Triangle budgets:** small props ≤ 500, medium (tree, penguin, snowman, rock) ≤ 1,500, large (car, train car, cabin) ≤ 4,000. Decorative meshes use `CollisionFidelity = Box` or no collision. Use `RenderFidelity = Automatic`.

---

## 3. RESTRUCTURE: 30-LEVEL HANDCRAFTED CAMPAIGN, THEN ENDLESS

**This changes the original design spec on purpose.** The original said "no numbered levels." I'm replacing that.

### 3A. Structure
- **30 handcrafted levels**, each **distinctly different**, together covering roughly **100,000m**. Levels get longer and harder as you go (early levels short, ~1,500–2,000m; late levels much longer). Tune lengths so each level takes about **1.5–4 minutes** at the expected speed.
- **Every level is EXACTLY the same every time you play it.** Same layout, same hazards, same timing, first play or hundredth. Players should be able to learn and master them.
- **After Level 30 → ENDLESS MODE.** Procedural, harder, and it combines every element and set piece from all 30 levels. Use the existing chunk generator and floating origin for this.
- **Checkpoints:** Beating a level unlocks it. Players can start from any level they've reached. **Endless mode unlocks after beating Level 30**, and players can then start directly in endless.
- **Leaderboards:** best time per level, campaign progress, and a **separate Endless distance leaderboard** that counts from where endless starts. Skipping to endless gives no unfair advantage.
- **Rewards:** First-time level completion gives coins, tickets and XP. Replays give reduced rewards. Optional **1–3 star rating** per level (finish / collect X coins / no crash or a time target).
- **Optional Daily Run:** one random endless layout that's the same for everyone that day, with its own daily leaderboard.

### 3B. How to build levels so they stay identical
- **Reuse what exists.** The course is already deterministic from `(seed, chunkIndex)`, and `Director`, `Sections`, `ChunkValidator`, `Path` and the themes already exist. Don't rewrite them. Instead:
  - The campaign uses a **fixed seed per level** and a **fixed theme per level** (no shuffled theme deck in the campaign; keep the shuffle for Endless).
  - Add a **level definition** for each level (a ModuleScript per level, or one `Levels` config): the exact ordered list of sections/set pieces for the Director to use instead of its random pacing plan, plus section parameters, length, theme, events (if any) and the level's exclusive obstacles.
  - Same definition + same seed = identical level, every time, for every player. Add a QA check that builds each level twice and compares the results.
  - Replace the current distance-based `Config.LevelAt` (levels 1–3 ≈ 806 m, 4+ ≈ 1.2 km) with the campaign's level boundaries. Keep the account level shown as **RANK**, so it doesn't get confused with campaign levels.
  - Endless mode is the existing endless generator, starting after Level 30 and tuned harder, using every section and set piece.
- Build a simple **dev level-preview tool** to fly through a level quickly for checking.
- Build and verify levels **in batches of 5**. Playtest each level start to finish before moving on, and record that in `PROGRESS.md`.

### 3C. Level design rules
- **Each level has a unique identity:** its own theme, scenery, signature obstacle and at least one memorable set piece. Some obstacles are **exclusive** to a level (e.g. only one level has the snowman field).
- **Alternate routes:** many levels have a **harder side route** for experienced players: narrower, riskier, more coins or a shortcut.
- **Boost-required moments:** some gaps or routes can only be cleared with boost. Players who saved boost can take them, and players who didn't must take the normal route. **The main route must always be possible without boost.** Mark boost routes clearly (sign, colored flags, glowing arrows).
- **Speed control moments:** some sections need you to **brake** (e.g. timing train crossings).
- **Every ramp has a purpose:** clearing a gap, getting over a blocker, reaching an alternate route, or flying through coin rings.
- **Pacing:** something interesting every ~150–300m. No long empty planes.

### 3D. The 30 levels (starting point; adjust if needed, but keep them distinct)
1. **Bunny Hill:** tutorial. Steering, coins, wide and gentle. Teach controls contextually.
2. **Pine Slalom:** trees and slalom flags. Teaches carving.
3. **First Air:** small ramps over small gaps. Teaches jumping and landing.
4. **Snowman Valley:** *exclusive* snowman fields. Smash some for bonuses, avoid the big ones. Varied snowmen (see 5D).
5. **Lift Line:** chairlift towers and low-swinging chairs. Duck or weave.
6. **Rolling Thunder:** snowballs rolling down side gullies.
7. **Frozen Pond:** cracking ice. Keep your speed up.
8. **Logging Road:** fallen logs, a sawmill, log piles, log trucks.
9. **Cannon Alley:** snow cannons firing across the slope.
10. **Polar Express:** a **train level, Subway Surfers style.** Railroad tracks cross the slope, and trains chug through with horns, steam and a crossing signal. You must **slow down or time it** to pass between trains. Includes a tunnel section and a station platform you can ramp over.
11. **Abandoned Highway:** snowed-in cars and trucks across the road. Ramp onto car roofs. A snowplow pushes snow across.
12. **Crevasse Fields:** ravines with ice bridges and required ramps.
13. **Moose Crossing:** wildlife herds crossing, forest cabins.
14. **Glacier Run:** sliding ice blocks and blue glacier walls.
15. **Avalanche!:** a chase level. The avalanche is behind you the whole time.
16. **Ice Caves:** falling icicles (done properly, see 5B) and glowing crystals.
17. **Mine Shaft:** minecart tracks, carts crossing, wooden supports, lanterns.
18. **Blizzard Pass:** whiteout with fair glowing markers and side wind gusts.
19. **Ghost Village:** an abandoned ski village. Weave between buildings, ride over roofs.
20. **Polar Bear Territory:** a bear chase.
21. **Half-pipe Canyon:** carve up the walls for coins and rails.
22. **Ski Jump Championship:** giant ski jumps, gold rings and a crowd.
23. **Frozen Waterfall:** ride across and down a frozen waterfall with ice ledges.
24. **The Ridge:** a narrow ridgeline with drops on both sides.
25. **Northern Lights:** a night level with aurora and glowing hazards.
26. **Radio Peak:** an observatory or radio-tower summit with satellite dishes, cables and a helicopter.
27. **Glacier Collapse:** the route breaks apart ahead of you.
28. **Yeti's Lair:** rare yeti scares, a cave of bones and ice.
29. **The Gauntlet:** a remix of everything at high difficulty.
30. **Summit Descent:** the grand finale. Big set pieces back to back, and a finish celebration that unlocks Endless.

---

## 4. SKIING FEEL, RAMPS, BORDERS, ANIMATION

### 4A. Ramps (several are broken)
- **Problem:** On some ramps you go up and then straight back down, with no launch. Some ramps make no sense where they are. Some ramps **kill you** because something blocks the landing.
- **Fix:**
  - Ramps need a real **kicker shape**: an upward curve at the lip (roughly 20–35°) so the player is **launched upward and forward**. Apply a launch impulse based on speed and lip angle. Test at low, medium and high speed.
  - **Landing validation:** for every ramp, simulate or raycast the jump trajectory at min/expected/max speed. The landing zone must be **clear** and slope downhill for a smooth landing. Nothing may block the flight path.
  - Remove pointless ramps. Every ramp must clear something or lead somewhere.
  - **Intentional "joke" trap ramps** are allowed **rarely** (max 1–2 in the whole campaign), but only if they're clearly signposted (e.g. a "⚠ DON'T" sign) so it reads as a joke, not a bug.

### 4B. Borders
- **Problem:** I can ski past the little side fences.
- **Fix:** The course edges must be **solid**. Use a proper boundary: snow banks rising at the sides plus fences or barriers with **invisible collision walls**. Hitting the edge should bump you back and slow you down (not kill you), with a snow-spray effect. The visual border and the collision border must be in the same place.

### 4C. Player animations
- **Problem:** The skiing animations aren't good, and the jump doesn't look like a jump.
- **Fix:** Create proper animations (Animation Editor or Blender rig), blended smoothly:
  - **Ski stance:** knees bent, leaning forward, poles held naturally.
  - **Carve left/right:** body leans into the turn, hips shift, skis edge.
  - **Tuck:** low crouch at high speed or boost.
  - **Jump:** crouch → **pop up** with legs extending → airborne tuck or grab (vary between a few grab poses) → spot the landing → **land with knee compression** and absorb.
  - **Hard landing** wobble, and a **crash/ragdoll** with tumbling.
  - Add procedural lean on top of the animations. Skis stay attached to the feet and aligned with the slope (IK or align). Poles don't clip through the body.
- Compare against reference videos or photos of real skiers. The pose should read as "skiing" instantly.

### 4D. Crash sound
- **Problem:** The crash sounds like **water**.
- **Fix:** Replace it with a layered crash: **body thud + snow crunch/powder burst + gear clatter** (skis and poles), plus a short grunt (optional). Pick sounds from the Creator Store by name and description. Then **list every sound ID you used in `PROGRESS.md` with what it's for**, so I can listen and verify them. You can't hear them, so tell me which ones you're least sure about. Review all other game sounds the same way (anything that sounds wrong for its action).

---

## 5. SPECIFIC FIXES

### 5A. Lobby sleds
- **Problem:** The sleds at spawn appear **out of the snow**, slide, then **sink back into the snow**, always at the **same constant speed**.
- **Fix:**
  - Sleds should **slide down the existing landscape**: follow the actual terrain surface (raycast or a spline along a real slope or sled run), tilting with the slope.
  - **Speed changes naturally**: speeds up on steeper parts, slows on flats. Each sled gets a slightly different speed, with random gaps between sleds.
  - **No popping in or out of the snow.** Spawn them out of view (behind a hill, inside a lift shed, or beyond the fog) and despawn them out of view the same way. Or give them a proper loop: a lift or conveyor carries them back up.
  - Add a rider (NPC skier/kid) and a snow spray trail. Animate on the **client** so they're smooth.

### 5B. Icicles (still look weird and are upside down)
- Flip them: the **thick base is attached to the ceiling** and the **sharp tip points down**.
- They must be visibly **connected** to the cave ceiling, with a small ice cluster where they attach.
- Vary sizes (0.6×–1.8×) and shapes. They shake before falling, a shadow grows on the ground, they fall with gravity acceleration (ease-in) and a slight wobble, and they shatter on impact.

### 5C. Penguins
- **Problem:** Cute, but they look weird: arms stuck at their sides and bad faces.
- **Fix:** **Find a good penguin model** in the Creator Store/Toolbox (or a clearly better custom one), following the free-model rules in section 2. It needs a proper penguin silhouette and flippers that move. Animate a waddle, flipper flaps, looking around, and a belly slide. Animations run on the client.

### 5D. Snowmen
- They **face the path** (toward the player's route).
- **Alternate them:** 4–6 variants (different hats, scarves, sizes, expressions, stick arms, some with a broom, a sign or a carrot nose falling off). Don't use the same one everywhere.

### 5E. Summit Express signs (I already asked for this)
- They're **hidden under the lodge**. Move every sign to a visible, sensible location: entrances, the start gate, the lift. Run the hidden-object audit and show me a screenshot of each sign.

### 5F. Mountain popping in before Level 2
- **Problem:** A big mountain **suddenly appears** right before Level 2 while I'm skiing.
- **Fix:** Find what's spawning it (chunk streaming, StreamingEnabled, LOD or a biome backdrop). Background mountains should be **persistent** (`ModelStreamingMode = Persistent`, or always loaded), or spawn far beyond the fog and render distance and fade in. Nothing large should ever pop into view.

### 5G. Barriers, buildings, ravines
- **Barriers:** some have random things hanging or floating on top. Rebuild them cleanly: proper ski-race netting, wooden fences, or orange safety fences with posts, all grounded.
- **Buildings:** no more "cube with a roof." Use real alpine architecture: timber frames, A-frames, stone foundations, balconies, framed windows with warm light, chimneys with smoke, roof overhangs and **snow on the roofs**. Every roof has to sit correctly on its building with no gaps or floating parts. Check every building from above and from the side.
- **Ravines:** they look bad. Rebuild with layered rock walls (strata), snow lips curling over the edges, icicles on the rims, depth fog or darkness at the bottom, and a clearly readable far edge with landing flags.

### 5H. More scenery
The slopes need more life, especially in the background (none of it should block gameplay):
- Distant mountain ranges, villages with lit windows, moving chairlifts and gondolas, smoke from chimneys.
- Birds, a helicopter, background NPC skiers, snowmobiles on side trails.
- Biome-specific detail: frozen lakes, waterfalls, cliffs, forests, ruins, observatories.
- Ambient particles: snowfall, wind-blown snow, sun glints.

---

## 6. LOBBY: FULL REDO IS OK

The lobby doesn't **blend** well: the edges, surfaces and objects don't flow together. I'm OK with you **redoing the whole lobby**.

- A believable mountaintop ski resort with stylized low-poly realism: a main lodge (proper architecture, see 5G), lift station, ski rental/locker hut, crate shrine area, quest board, leaderboard boards and a start gate.
- **Terrain blends:** paths of packed snow transition smoothly into soft snow, snow drifts against walls, snow on every roof and fence top. No hard cut-offs or flat slabs. The lobby edge transitions into hills and mountains with fog (no visible wall).
- **Zero z-fighting/flicker** (run the audit), zero clipping, zero hidden signs and props.
- Trees grounded correctly (see the grounding audit). Varied tree types.
- Small life details: string lights, lanterns, a fire pit with smoke, ski racks, benches, people (NPCs) and the new penguins.
- Screenshot the lobby from 6+ angles, including from above, and compare against reference photos of real ski resorts.

---

## 7. UI: FULL REDO

**Problem:** The UI looks "AI-made." Text cuts off. Icons look bad. Some panels scroll when they shouldn't. The wording sounds like AI ("Your style," "Same skill," "Save outfits," "Go back to the mountain").

**Target:** look like a **popular Roblox game's UI**, e.g. **Blox Fruits, Fisch, Pet Simulator 99, Grow a Garden**. Study their style (search for screenshots if you can) and match the *quality and feel*, not copy their assets:
- **Bold cartoony font** (e.g. Fredoka One / Luckiest Guy / Bangers style) with **thick dark text outlines** (`UIStroke` on text).
- **Chunky buttons** with a gradient, a darker bottom edge for a 3D look, rounded corners, an outline, and a press animation (squish).
- **Big, clear icons** with outlines, the same style everywhere. No emoji. No tiny flat shapes.
- **Bright saturated colors** with clear meaning: green = play/confirm, red = close, gold = coins, blue = tickets, rarity colors for items.
- **Very little text.** Short, plain labels.
- **Wording:** short, natural, game-like. Use "Play," "Outfits," "Save," "Back," "Equip," "Claim!" Never use marketing-speak or filler phrases. Read every string and ask: "Would a real game say this?"
- **Nothing cuts off. Ever.** Use scale sizing, `UIAspectRatioConstraint`, `UITextSizeConstraint`, AutomaticSize, and the UI audit from section 1 at phone, tablet and desktop sizes.
- **Panels don't scroll unless they really need to.** Use a grid that fits on screen, or tabs/pages.

**Quests:**
- The current quest board is a scrolling list with only **3 daily and 2 weekly** quests, and some are too easy. Redo it:
  - **6 daily quests** (2 easy, 2 medium, 2 hard) and **8 weekly quests** (bigger goals), with rewards scaled by difficulty.
  - **1 free reroll per day.**
  - **Level challenges:** each campaign level has its own star challenges.
  - Quests are about skill: reach a level, get a 10× combo, near misses, take an alternate route, use boost to clear a boost gap, beat a level without crashing, survive the train level, etc.
  - Layout: a **card grid that fits on one screen**, with tabs for Daily / Weekly / Levels. Each card has an icon, a short title, a progress bar, the reward and a claim button.

Also redo: the main menu/lobby HUD, Locker/Outfits, Shop, Crates, Daily Spin, Index, Leaderboards, the run HUD, the level-complete screen (stars, time, rewards, Next Level / Replay) and the crash screen. **All in the same style.**

---

## 8. PHASE ORDER

Do these in order. Finish, playtest, verify and update `PROGRESS.md` before moving on:

0. **Orient + backup + re-verify.** Read Appendix A and the docs on disk, confirm the project copy in `~/Projects/alpine-ski/` is complete and update the doc paths, and save a full `.rbxl` backup. Re-check pass 9's claims for the bugs I listed at the top, and write down which are actually broken and why your checks missed them.
1. **Audit tools** (section 1). Run them and record baseline counts.
2. **Quick fixes:** icicle orientation, snowmen facing, hidden signs, z-fighting, tree grounding, mountain pop-in, crash sound, borders.
3. **Ramps + player animations** (section 4).
4. **Level system restructure** (section 3A–3B), plus Levels 1–5. Then build and playtest levels in batches of 5 up to 30, then Endless.
5. **Art pass:** penguins, snowmen variants, sleds, buildings, barriers, ravines, scenery (sections 2, 5).
6. **Lobby redo** (section 6).
7. **UI redo + quests** (section 7).
8. **Final QA pass:** run all audits (all 0), play every level start to finish, test on phone/tablet/desktop sizes, check performance (FPS, no hitches), and check there are no console errors.

## 9. FINAL REPORT

When finished (or when you have to stop), give me:
- **Before/after screenshots** for every item in this prompt.
- Final audit counts (all should be 0).
- The list of sound IDs for me to check.
- Anything you couldn't fix, and why.
- What you recommend doing next.

---

# APPENDIX A: HANDOFF DOCUMENT FROM THE PREVIOUS CHAT (`alpine-ski/PROJECT_SUMMARY.md`, after pass 9)

> The previous Claude chat wrote this at the end of its last session. It's pasted here word for word so you can orient yourself. **Remember: several things it calls "fixed" or "verified" are still broken in my playtests** (see "READ THIS FIRST" above). Use it as a map of the project, not as proof that something works.

````markdown
# Powder Rush ("Ski As Far As You Can"): project summary and handoff

_Last updated 2026-10-02, after polish pass 9. This is the starting point for a new chat. Where it disagrees with older notes, this file wins._

**Game.** An endless, procedurally generated, low-poly downhill skiing game on Roblox. Players ski as far as they can on a fair, skill-based course. Between runs they spend the coins and tickets they earned on cosmetics (suits, skis, trails...), crates, a daily spin and quests in a resort lobby. Cosmetics never affect survival.

---

## 1. Where everything lives

| What | Path / place | Notes |
|---|---|---|
| **Live game (source of truth)** | The open Roblox Studio place (universe 10768673585) | All UI is built at runtime, so StarterGui is empty on purpose. |
| **Script source (synced)** | `/private/var/folders/rj/l53ft6t10ql9ty7jqt5hw1_00000gn/T/ropilot-src/140601517752326/src/` | Edit the files here, then commit back to Studio (see §2). |
| Project folder | `/private/var/folders/rj/l53ft6t10ql9ty7jqt5hw1_00000gn/T/alpine-ski/` | Docs, assets, QA, checkpoints |
| Original design brief | `alpine-ski/GAME_SPEC.md` | The full spec: movement, generation, hazards, economy, UI, monetization |
| Running log | `alpine-ski/PROGRESS.md` | History pass by pass. Sections 1–5 are old (pre-redesign); sections 6–9 are recent. |
| **Latest script checkpoint** | `alpine-ski/checkpoints/pass9-2026-10-02/src/` | A copy of all scripts as of now. Older checkpoints are in the same folder. |
| Blender sources | `alpine-ski/assets/resort/Resort.blend` plus `lib.py`, `terrain.py`, `nature.py`, `props.py`, `hazards.py`, `gear.py`, `icons.py`, `assets2.py`, `placements.py` | `lib.py` has the Builder class, the `ResortPalette` texture atlas and helpers |
| Painted UI images | `alpine-ski/assets/ui/` | Wheel face, pointer, glow, trail map (+ `route.npy`, `trail_marks.json`) |
| Suit textures | `alpine-ski/assets/suits/` | 25 suit prints |
| Before/after screenshots | `alpine-ski/qa/pass9_after/` | Named by the user's original screenshot number (01–17) |
| Stale source | `T/src/` | An early Phase 1 copy. **Never edit or restore from it.** |

---

## 2. Tooling and workflow (Ropilot plus Blender)

- **Editing scripts.** Edit the local file, then commit it (`ropilot_sourcecontrol_commitscript`).
  - To add a new script, use `createscript`. **Create-script overwrites the local file with a template**, so write or copy the content *after* creating it, then commit.
  - To fetch a fresh copy from Studio, use `requestscript`.
- **Edit-context Lua.** For builders and QA run from edit mode:
  - Clone a module before you `require` it, to dodge the require cache: `local c = M:Clone(); c.Parent = game.ServerStorage; require(c)`.
  - For the shared `ReplicatedStorage.Alpine` modules, clone the whole folder, swap it in, then restore the original afterwards.
  - Plugin context **can** set `ModuleScript.Source`.
- **Builders** live in `ServerStorage.ResortBuilder`, run as `require(Builder)(require(Kit), workspace.AlpineLobby, ...)`.
  - `Kit.model(name, parent)` destroys and recreates a model.
  - `Kit.asset` places a model from `ReplicatedStorage.Alpine.Assets`.
- **QA modules** are in `ServerStorage.PowderRushQA` (see §8).
- **Screenshots.**
  - `ropilot_studio_screenshot` takes edit mode only. To aim it, set `workspace.CurrentCamera.CameraType = Scriptable` and its CFrame, then restore.
  - Studio "front" views are often from behind the subject, so aim the camera by hand instead.
  - **The in-playtest `ropilot_screenshot` helper times out every time, and playtests seem throttled to about 1–2 fps when the Studio window is unfocused.** Timings measured in playtests and quick sequences such as the countdown are unreliable. Check visuals in edit mode with `UIPreview` / `SetPiecePreview`, and check logic with numeric samples.
- **Playtests** go through the `roblox-tester` subagent. Give it exact paths and steps. Afterwards, call `ropilot_is_playing` and stop the test if it's still running.
- **Blender** runs headless and restarts after 30 idle minutes, which empties the scene. To get back, run `bpy.ops.wm.open_mainfile(Resort.blend)`.
  - Axis mapping: Roblox X = Blender X, Roblox Y = Blender Z + 260 (terrain), Roblox Z = −Blender Y. Blender +Y is the model's front, which is Roblox −Z.
  - Imports: `ropilot_blender_import(collection, name, position = bottom centre)`.
- **Uploading images for UI.** Put each image on a textured plane in Blender, import it, read the MeshPart `TextureID`, then delete the import. IDs are listed in `UIComponents.Images`. New images may take a few minutes before they render.
- **Gotchas.**
  - A MeshPart with a `TextureID` ignores `Color`, so tint through `SurfaceAppearance.Color`, or clear `TextureID` with pcall.
  - Setting `CollisionFidelity = PreciseConvexDecomposition` from edit context once **disconnected Studio**. Avoid it.
  - `Motor6D.C0` is read-only on generated rigs, so pose by rotating the part CFrames.
  - EditableMesh is **disabled** in this experience's settings.
  - StreamingEnabled is **on** (see §6).
  - Summit meshes have CanQuery = false and convex-hull collision, so raycasts on them are wrong. Use the height functions instead.

---

## 3. Architecture (current)

**Deterministic, server-authoritative simulation with client prediction.**
- The client sends controls only (steer, brake, boost, hop).
- The server runs the shared `SkiSimulation` at 60 Hz, decides crashes and rewards, and sends snapshots.
- The course is generated from `(seed, chunkIndex)`, so client and server build identical courses.

### Shared: `ReplicatedStorage.Alpine`

| Module | Role |
|---|---|
| `Config` | All tuning (see the numbers below), plus the 14 themes, economy, audio IDs (`Audio.Beep` is the countdown beep), level maths and `Config.Speed(m)` |
| `Path` | Path coordinates `(s, x, h)` → world, plus half-width and curvature |
| `Course` | Chunk provider (`Course.New(seed)`) with chunk fields: hazards, ramps, rails, coins, platforms, gaps, props, cable, lake, mine, cave, `setPiece`, `safeRoutes` <br>`SurfaceHeight` includes platforms (with roof rise), `ProfileHeight` (half-pipe walls, mogul ellipsoids) and `EdgeLift` |
| `Director` | Plans 6-chunk segments. Slots 2 and 5 hold set pieces only, other slots never do. No set piece repeats within ~800 m and no archetype appears twice in a row. A fixed INTRO opening. |
| `Sections` | 39 generators: 17 set pieces, 2 recovery fillers, and the rest. Each builds a safe line plus scatter.<br>Builder helpers: `ring`, `smash`, `platform`, `ramp`, `clearBox`, `scatter` (keeps crossing lanes empty), `crossers` |
| `ChunkValidator` | Safe lane clear, turns survivable, ramps reach landings, ravine bypass, platform and bridge rules.<br>New in pass 9: **"obstacle in crossing lane"**. A failing chunk falls back to an empty chunk on both sides (currently 0 fallbacks). |
| `SkiSimulation` | Movement, ramps (launch only if velocity > 0), platforms (crash into the side, ride on top), lake speed rule (`FELL THROUGH THE ICE`), rings, smash, crossers, swings, strikes, near misses, adrenaline, boost |
| `HazardMotion` | Hazard poses: Snowball, Cannon, MovingIce, Icicle, Strike (Rock/Icicle with shake and ease-in fall), Vehicle, Crosser, Swing.<br>Crash reasons: e.g. `FALLEN GIANT`, `CABIN WALL` |
| `EventSchedule` | Mid-run events (Avalanche, Bear, Gold Rush, Whiteout, Snowman Attack, meteors...) |
| `Difficulty` | Distance bands |
| `Catalog` | Cosmetics (suits, skis, poles, helmets, goggles, trails, sleds, emotes, companions...)<br>**9 crates:** Snow, Alpine, Summit plus 6 themed (Volcano, Aurora, Candy, Haunted, Crystal, Clouds) with odds, pity and rarity colours. `Catalog.Tint`, `Catalog.Pool` |
| `QuestDefinitions`, `ProductConfig` | Quests. Product IDs are still **0**, so no real purchases happen. |
| `AnimationClips` / `AnimationSampler` | Rider poses bundled as code (uploaded animations were rejected) |
| `UIComponents` | Design system (§7): `Palette`, `Images`, `IconSheet`/`Icons`, `Panel`, `Button`, `IconButton`, `Label`, `Small`, `Round`, `Stroke`, `Gradient`, `TextStroke`, `Shadow`, `Pop`, `Number`, and **`Theme(root)`** |
| `Assets` (Folder) | Every model and mesh used by the game (§5) |

### Server: `ServerScriptService.AlpineServer`

| Module | Role |
|---|---|
| `RunManager` | Remotes (`Alpine.Net`: Command, Input, State, Meta).<br>Session phases Lobby → Countdown (3 s) → Running → Ended.<br>**The client owns the skier:** an unanchored root with PlatformStand and the player as network owner, which the server never moves during a run.<br>Publishes `AlpinePhase`, `AlpineDistance`, etc. |
| `DataService` | Session-locked profiles and rewards. Publishes `AlpineSuit`, `AlpineLevel` and `AlpineBest` attributes. `Data:Public(player)` builds the client profile. |
| `MetaService` | One RemoteFunction for Buy, Equip, Crate, Spin, ClaimQuest, ... Lobby ProximityPrompts with a `Menu` attribute open menus. |
| `InventoryService`, `QuestService`, `LeaderboardService`, `ProductService`, `CosmeticService` | As named |

### Client: `StarterPlayerScripts.AlpineClient`

| Module | Role |
|---|---|
| `Main` | Phase state machine.<br>Fixed 60 Hz prediction, rendered from an **interpolated `view`** with correction smoothing that decays over about 0.15 s.<br>Starts `SuitRenderer` and `NameTags`. Autopilot only runs in Studio, when `workspace:SetAttribute("PowderRushAutopilot", true)`. |
| `WorldRenderer` (~100 kB) | Budgeted chunk streaming: coroutines with a 4 ms/frame budget, nearest first, current chunk ±1 built at once.<br>**Floating origin:** recentres every 2,880 studs with `BulkMoveTo`, near chunks immediately and far chunks the next frame.<br>Builds all set-piece visuals, hazards and decoration (with the decoration validator, §4).<br>Telemetry attributes on `workspace.AlpineRunWorld` (MaxGenMs, MaxJobMs, RebaseMs...) |
| `CharacterController`, `CameraController` (air pull-back on big jumps), `EffectsController`, `AudioController`, `WeatherController`, `AnimationController`, `InputController`, `EmoteController` | As named |
| `UIController` | Lobby HUD, run HUD, **start-light countdown gantry** (§7), run-end screen. Responsive scale 0.6–1.1. |
| `MetaController` (~50 kB) | Menus (Locker, Shop, Crates, Quests, Index, Records, Settings, **Wheel**), the **3D crate opening**, item previews |
| `LobbyController` | Client lobby animation (§6) and trail-board updates |
| `SuitRenderer` | Suits on any avatar, plus the mannequin builder (§6) |
| `NameTags` | Overhead name tags (§6) |
| `Autopilot` | Studio test bot |

### Key numbers
- **Scale:** 1 stud = 0.28 m. A chunk is 240 studs; a Director segment is 6 chunks.
- **Speed:** 40 + 60·(1 − e^(−m/2600)), capped at 100 st/s (0 m: 40, 1.5 km: 66, 5 km: 91).
- **Levels:** levels 1–3 are about 806 m each, level 4+ about 1.2 km. After a fixed six-level opening, themes are dealt from a shuffled 14-theme deck with no repeats.
- **Economy:** 1 coin per 15 m. Pickup 5, ring 15, smash 3, near miss 5, ramp 10, clean landing 10, record 100. 1 ticket per 1,000 m. Milestone chests at 500/1k/2.5k/5k/7.5k m.

---

## 4. Slope content

- **39 sections:**

  | Group | Sections |
  |---|---|
  | Classics | Open Cruise, Forest Slalom, Rock Garden, Snowbank Chicane, Glacier Gates, Ravine Jump, Side Ravine, Rail Line, Trick Park, Ice Cave, Frozen River, Cliffside Trail, Resort Run, Snowball Alley, Cannon Alley, Falling Timber, Split Route, Hairpin, Crevasse, Fracture Field |
  | **Set pieces** | Abandoned Truck, Snowcat Wreck, Fallen Chairlift, Giant Log, Cabin Jump, Patrol Barricade, Snowmobile Crossing, Penguin Parade, Swinging Chairs, Frozen Lake, Snow Bridge, Half-pipe, Mogul Field, Ski Jump, Rockfall, Snowman Field, Mine Tunnel |
  | Fillers | Narrow Ridge, Powder Bowl |

- **14 themes (one per level):** Alpine Resort, Snow Forest, Glacier, Sunset Ridge, Blizzard, Ice Caves, Extreme Peaks, Aurora Valley, Frozen Lake, Crystal Canyon, Volcanic Ridge, Candy Cane Lane, Haunted Hollow, Cloud Summit.
  - Each sets its own colours, fog, weather, set dressing (crystals, lava cones, candy canes, pumpkins, ...) and section bias.
- **Every ramp has a purpose:** a drift onto the truck roof, kickers through rings, the ski-jump lip, ramps over gaps.
- **Icicles:** hang from the cave ceiling, then shake, trickle, crack and fall with an ease-in before shattering.
- **Rocks:** fall from the wall side with a growing ground marker.
- **Snowballs:** grow, leave a track and throw up powder.
- **Snowmobile crossings:** headlight and spray.
- **Decoration validator** (`WorldRenderer:Decorate`):
  - Spacing is measured in world XZ: footprint + 1 stud, at least 6 studs apart.
  - Keep-out discs around crossing lanes, cable towers and anchors, signs and roadside snowmen.
  - Every footprint stays inside its own chunk.
- **Assets:** 6 pine variants (A–F) plus a dead tree, 6 faceted rocks, 6 crystal clusters, 4 icicle sizes, all with tint and scale variation.

**Verification:**

| Check | Result |
|---|---|
| CourseV2 regression | 1,800 chunks, **0 fallbacks**, all 39 sections appear, all jumps land |
| Decoration overlaps | 0 in 96 chunks across 3 seeds |
| Crossing lanes | 0 objects in 61 lanes |
| Giant Log | 0/129 crashes in a deterministic autopilot simulation |

---

## 5. Art and asset pipeline

- **Blender collections** in `Resort.blend`, imported as `ReplicatedStorage.Alpine.Assets` (AssetOrigin part at the base, used as PrimaryPart):
  - PenguinV2: 6-part rig, 1,126 triangles
  - Crates: CrateSnow, CrateAlpine, CrateSummit, CrateThemed (lid parts are named `*Lid*`, panels `*Panels*`)
  - Props (PropSet): benches, lamp posts, signposts, ski racks, fire pit, picnic tables, Adirondack chair
  - Set pieces: car, snowcat, fallen chairlift, giant log, cabin, barricade, snowmobile, minecart, track, mine frame, gold ring
  - Nature: PinesV2, RocksV2, CrystalsV2, IciclesV2, SnowballV2
  - SuitV2: unit-space suit pieces with a `UnitSize` attribute
  - Helmet and goggles: AlpineHelmet / AlpineGoggles
  - `MannequinR15`: faceless matte R15 body
  - Summit terrain: SummitNW/NE/SW/SE
- **Triangle budgets are all met.** The largest object is the car (1,348). Props are 200–1,108; everything is under 5k.
- **Terrain** (`terrain.py` `summit_height`): **fixed in pass 9.**
  - The massif term used to floor the ground at −6, which built a 64–162-stud cliff and shelf beside the lobby (the user's "edge wall").
  - The base now blends from the side slope to −6 between z = 200 and z = 400. The peaks are unchanged.
  - `terrain_pre_pass9.py` is kept for reference, and `ResortBuilder.SummitDelta` holds the height delta.

---

## 6. Lobby (`workspace.AlpineLobby`)

- **Top-level folders:** Lodge, Chairlift, StartArea (StartGate, TrailMapBoard, Firepit, PracticeArea...), LockerHut, Market (CrateShed, SkiShop...), Plaza, Grounds, Props, PrizeWheelStation, Bounds (invisible guards), Summit (4 terrain meshes), Polish (CrateShrine, SuitGallery, Penguins, Drifts, FestoonLights, LevelsBoard), Forest (ForestV2), Edges (berms).
- About 8.1k parts. 3,192 shadow casters (berms, mounds) were turned off.
- **Builders:** Polish, ForestV2, Edges, PropSwap, TrailBoard, SummitHeights (height grid, now corrected), SummitDelta.
- **Overlap audit:** `PowderRushQA.LobbyAudit(fix)` reports **0** (it started at 15 real overlaps).
- **Fixed this pass:** pitched crate-shed roof with tier chests inside, ski racks instead of skis leaning on walls, the bin moved out of the shed corner, a lamp moved off the signpost, a tree removed from the trail board's sightline, and old berm wedges rotated so their slopes face out.
- **Streaming:** penguins are Persistent; chairs, shrine crates, showcases and the wheel are Atomic. `LobbyController` adopts models as they stream in and drops ones that leave.
- **Penguins:** client-side rig animation with one `BulkMoveTo` per penguin per frame: waddle with foot lift, head look-around, flipper flap, belly slide. Verified live.
- **Suit Gallery:** five R15 mannequins on three-tier rarity-lit pedestals. **Shrine:** tier chests that bob and glow.
- **Start gate:** the light bar was lowered off the banner. The lamps start dim and copy the countdown.
- **Trail board v2:** illustrated map, pulsing YOU ARE HERE, LV waypoints, a BEST flag placed along the route, and challenge cards. `LobbyController.Profile` fills it in.
- **SuitRenderer v2:**
  - Modelled suit pieces with the skin's SurfaceAppearance, mirrored trim stripes, gloves, boots, and a helmet and goggles sized to the head.
  - Hats, hair and face accessories are hidden locally while a suit is on.
  - Verified live: 26 welded parts.
- **NameTags:** BillboardGui `AlpineTag` on the head with level chip, display name, and `BEST n m` (or live distance).
  - StudsOffset (0, 2.75, 0), AlwaysOnTop, MaxDistance 90. Your own tag hides during the countdown and the run.

---

## 7. UI design system and screens

- **Style:**
  - Snowy panels: `C.Panel` is white→ice with a navy outline.
  - Text: FredokaOne for headings, Gotham for small text; navy on light surfaces.
  - Colours: orange or gold for primary buttons, gold for currency.
  - `C.Theme(panel)` recolours text on light surfaces. Mark a label with the `KeepColor` attribute to exempt it.
  - Menus run the theme pass automatically from `ClearContent`. Views without the category column use the full width.
- **Daily Spin:**
  - Painted wheel with icons and amounts on each wedge, and a pointer that flicks on each peg.
  - 5.2 s Quint ease-out with six full turns and a tick per peg, then a reward pop-up with confetti and an odds table. The lobby's 3D wheel spins along.
  - Verified live: landed on "400" and paid +400.
- **Crate opening:**
  - The tier chest drops into a ViewportFrame and shakes while rarity light leaks from the seam.
  - It unlatches with a hop and click, then the hinged lid opens to 108° with overshoot.
  - A core glow, rays, flash and sparks lead into the rarity reveal. Odds chips are shown, and you can tap to skip after 0.6 s.
  - Crate cards show the tier chest and a coloured odds line. Verified live.
- **Countdown:**
  - A gantry hangs below the HUD (offset ≈ 124 × HUD scale + 14).
  - Sequence: dim, 3 = red, 2 = red + amber, 1 = red + amber + amber, GO = all green, then reset. A beep sounds on each step (pitched up for GO) and the number pops in.
  - In edit-mode preview the 3/2/GO states looked right. In the throttled live test GO fired and the lamps reset; the intermediate 3/2/1 frames weren't caught.
- **HUD:** speed top-left, distance, level and biome top-centre, coins top-right, boost meter at the bottom.

---

## 8. QA tools (`ServerStorage.PowderRushQA`)

| Module | Use |
|---|---|
| `CourseV2()` | Generation regression across 3 seeds × 600 chunks (fallbacks, archetypes, jumps, rails, autopilot) |
| `LobbyAudit(fix?)` | Decor vs structure and decor vs decor overlap audit, with an automatic fix |
| `SetPiecePreview(name)` | Builds a section at x = 5000 with a `SetPieceFocus` part for screenshots. Call with `nil` to clean up. |
| `UIPreview(view)` | Mounts the real UI in StarterGui. Views: `"Wheel"`, `"Crates"`, `"Locker"`, `"Quests"`, `"Lobby"`, `"Countdown"` (pass n), `"Results"`. Call with `nil` to clean up. |
| `ThemePreview` / `ThemeRestore` | Preview a theme's lighting, and restore it afterwards (don't leave a preview on) |

---

## 9. History (what the user asked for, by pass)

1. **Original spec:** endless skiing game (`GAME_SPEC.md`).
2. **Phase 1–3:** core skiing, lobby, then the redesign (sections, art, UI).
3. **Pass 2:** fix clipping into the snow, a better lobby, truly endless levels with 14 themes, and a crate system with suit skins.
4. **Pass 9 ("Polish & content", with 17 screenshots).** Priority order: smoothness → slope variety → clipping → models → lobby → UI. All done, as described above.

---

## 10. Open items / next steps

1. **Real performance check.** Use a focused Studio window or a phone with the MicroProfiler.
   - Throttled telemetry showed build jobs up to about 11 ms (budget 4 ms), the world recentre at about 36 ms once per 2.9 km, and streaming peaks around 42 ms.
   - Consider splitting the recentre further, or capping job slices.
2. **Device emulator** (phone and tablet) not tested. The logic scales, but it hasn't been seen on those screens.
3. **Countdown 3/2/1 frames** are not confirmed live (the throttling issue). Run one focused manual test.
4. **Old Grounds.Berm wedges** still show blocky ends at the lobby sides. Replace them with Edges-style berms.
5. **Product IDs are 0** (no monetization), and DataStore saving in Studio is untested in production.
6. **EditableMesh is disabled.** Terrain changes need a Blender re-import.
7. **Autopilot** sometimes crashes in throttled playtests on timing obstacles; this is not a course issue.
8. **Backups.** No recent full `.rbxl` place backup; the newest full place file is from Phase 1. Save one: File → Save to File.
````
