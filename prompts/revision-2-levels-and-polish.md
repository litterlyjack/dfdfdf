# REVISION 2: 30-LEVEL CAMPAIGN, REAL ART QUALITY, AND A FULL UI REDO

You revised the game after my last prompt. It's better and it doesn't look bad, but a lot is still wrong, and **some things I already asked you to fix are still broken**, like the signs hidden under the lodge. I've attached new screenshots.

The main problem is **awareness**: things get placed upside down, facing the wrong way, inside other objects, under buildings, or into the snow, and nobody notices. Most of this prompt is about fixing that, and about building **checks** so it can't keep happening.

I don't care how long this takes. Work in phases, keep notes, and check, double-check and triple-check your own work.

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
- Define each level as **data** (a ModuleScript per level): an ordered list of sections/set pieces with exact positions and parameters, plus a **fixed seed** for any filler decoration. Same data + same seed = identical level every time.
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
