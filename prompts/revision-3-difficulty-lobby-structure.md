# REVISION 3: REAL DIFFICULTY, LEVEL STRUCTURE, A NEW LODGE, AND QUALITY THAT HOLDS UP IN PLAY

## 0. ORIENTATION (you may be a new chat)

1. Read, in this order: `~/Projects/alpine-ski/PROJECT_SUMMARY.md`, the "REVISION 2" section of `PROGRESS.md`, `REVISION_2_PROMPT.md` (its rules on audits, verification and art standards **still apply**), and `TEST_CHECKLIST.md`.
2. Where this prompt disagrees with any of those, **this prompt wins.**
3. Log this work as a new "REVISION 3" section in `PROGRESS.md`, and update `PROJECT_SUMMARY.md` at the end of every phase so the next chat can continue.
4. **Backups:** before each phase, ask me to save `~/Projects/alpine-ski/checkpoints/Rev3 Phase<N>.rbxl` (you can't save .rbxl yourself), and zip the scripts and assets to iCloud as you did in Revision 2.
5. My screenshots for this revision are in `~/Projects/alpine-ski/qa/owner-screenshots/rev3/`. Look at all of them before you start.

---

## 1. WHAT I FOUND WHEN I ACTUALLY PLAYED IT

I played through the game myself after Revision 2. The good news: **every level is unique, and the set pieces (Cannon Alley and others) are cool.** The bad news: your report said every audit was at 0, but in real play I still found many visual bugs, the lobby looks bad, and **the game is far too easy.**

### Why the audits said 0 but the game still looks wrong
The audits check *correctness* (overlaps, flicker, facing, grounding of trees and listed props). They don't check:
- **whether parts of a model are attached to each other** (snowman noses, penguin flippers, Christmas lights, stand supports, hanging tags),
- **whether runtime set-piece vehicles and objects touch the ground** (the logging truck and logs float),
- **pop-in of moving set pieces** (the swinging chairs),
- **camera clipping into event objects** (the avalanche),
- **quality.** "No errors" is not the same as "looks good."

**Add these audits now (section 6), and from now on judge every visual change against a real reference photo, not only against audit numbers.**

### Bugs I saw while playing (fix all of them)

**Level 5 (Lift Line):**
1. **Fallen Chairlift:** the cable/rope hangs down in a way that looks wrong. Make the fallen chairlift read clearly as a *wrecked chairlift*: a snapped cable lying on the snow or draped believably between a toppled tower and a standing one, with chairs scattered and half-buried. The cable must never hang through the air at a weird angle or float.
2. **Swinging chairs:** they **pop in** at a cut-off point (spawn visibly). Fix: they must exist before they come into view. I also want **two parts** in this section:
   - **(a) Swinging chairs:** chairs swaying back and forth on a still cable.
   - **(b) Moving chairs:** the cable runs, so chairs **travel across/along the slope** while swinging, and you have to time your pass.
   The cable, towers and chairs should look like a real chairlift (bullwheel, tower crossarms, sheaves, chair with a safety bar).

**Ramps:**
3. **Some ramps push you DOWN instead of launching you up**, on Level 1 and Level 5. The wooden ramps work, but the others don't (probably the snow/drift ramps). Every ramp type must launch you up like the wooden ones. Test every ramp type at low, medium and high speed in a real run, and log the launch height for each.
4. **The ramp with tires on it kills you.** If that's an intentional troll ramp, it isn't readable: it just feels like a bug. Rule: **no ramp kills you unless it's clearly signposted as a joke ramp** (max 1–2 in the whole campaign). Otherwise make the landing clear.

**Barriers:**
5. **Red "notes" (tags/flags) hang in the air above some barriers.** Find what these are and either attach them properly to the barrier (e.g. a sign zip-tied to the netting) or remove them.

**Level 6 and caves:**
6. **Icicles look like a model "hanging out,"** not part of the cave. They need to **grow out of the ceiling/rock**: the base blends into the ceiling with ice buildup, there are clusters of different sizes, and some are thin, some thick, some broken. I want **much more diverse icicles** everywhere (caves, frozen pond, edges).
7. **Rockfall (Level 6):** right now rocks are *thrown*. I want a **cave-in**: you're inside a cave or under a cliff, and the ceiling/wall **crumbles**, with dust trickling first, cracks, small pebbles, then rocks breaking off and falling. Replace the sound too: I want a deep rumble plus a rock crack and debris, not what's there now.
8. **The rocks in Level 6 in general "hang off"**: they look like they're floating or sticking out of walls unnaturally. Ground them into the walls and floor, with partly buried bases.

**Avalanche:**
9. **The camera clips into the avalanche** sometimes, even though it's supposed to stay behind you. Keep the camera out of the avalanche volume (raycast or clamp the camera, or keep the avalanche's visual front a minimum distance behind the camera) while it still looks threatening.

**Frozen pond / frozen river:**
10. **The frozen pond looks out of place.** It should **blend into the terrain**: snowy banks that melt into the ice edge, reeds or rocks frozen at the shoreline, cracks and air bubbles in the ice, a slight blue-gray tint, and snow dusting at the edges. The icicles around it have the same problem as in 6. (The "you can't go slow on it" rule is fine. Keep it.)
11. **Frozen river: the warning barriers** (that tell you the ground will drop) **look ugly**, and **the whole frozen river section looks ugly**. Redo it with reference photos: a real frozen river with banks, rocks, overhanging snow, and warning markers that look like real things (e.g. orange "thin ice" signs on poles, a rope line with flags).

**Penguin crossing:**
12. The sliding penguins **lag** (stutter) when crossing. Animate them on the client with smooth interpolation.
13. Make their routes **varied**: some slide straight across, some slide **down the trail toward you**, some go back and forth, some zig-zag. **It shouldn't be the same pattern every time** within a level. (In the fixed campaign levels each penguin's pattern is fixed, but the patterns differ from one penguin to the next.)

**Logging Road:**
14. **The truck floats**: its wheels are in the air, and the logs float too. Every runtime vehicle and object must touch the ground. The wheels sit on the snow (slightly sunk into it), and logs rest on the ground or on each other. Add this to the grounding audit (section 6).

**Background skiers:**
15. The skiers on the side look ugly. **Make them real Roblox characters** (R15 rigs with HumanoidDescriptions: real shirts, pants, accessories, faces) wearing our ski suits and gear, with the same ski animations as the player. A few different outfits.

**Candy Cane Lane (Christmas level):**
16. **More diverse snowmen** (hats, scarves, sizes, poses, expressions, some with presents or lights).
17. **The Christmas lights on the trees aren't attached.** The lights must wrap around the tree on its surface, following the branches, with bulbs on a wire.

**Snowmen in general:**
18. **Some snowman noses aren't attached.** Every part of every snowman must be attached (section 6, attachment audit).

**Lobby penguins (third time I've asked):**
19. The lobby penguins still look horrible: **the flippers stick straight out and aren't even attached to the body.** Real penguins hold their flippers down and slightly out from the body, attached at the shoulder. **Get a good penguin model from the Creator Store** (follow the free-model rules: delete every script inside, check the triangle count, match the style). If you build one yourself instead, it must be compared side by side with a photo of a real penguin and look clearly better than now.

---

## 2. DIFFICULTY: THE GAME IS WAY TOO EASY (TOP PRIORITY)

**Problem:** Every level is unique, but the **difficulty stays almost flat** from Level 1 to Level 30. I could finish the whole game in one day. Your autopilot finished all 30 levels, which shows that the levels are *possible*, not that they're *hard*.

**Target, like "Bike of Hell" on Roblox:**
- Level 1 is very easy: it teaches you the basics.
- Each level gets a bit harder and **introduces something new**.
- By Level 25–30, the game is **genuinely hard**, with mechanics you've never seen before. **The last level should make most players die multiple times.** Finishing all 30 levels should be an achievement, not something everyone does in a day.

### Difficulty curve (use this as the plan)
| Levels | Feel | What changes |
|---|---|---|
| 1–3 | Tutorial | Wide paths, slow, forgiving, one new thing per level |
| 4–10 | Easy → medium | Hazards start combining, narrower lanes, first timing challenges |
| 11–20 | Medium → hard | Tight timing, fewer safe lanes, faster speed, boost-required routes, less reaction time |
| 21–27 | Hard | Combined set pieces back to back, precise jumps, moving platforms, very little room for error |
| 28–30 | Brutal | Everything at once, new late-game mechanics, death expected; mastery required |

### Late-game mechanics (introduce gradually; some only in the last ~10 levels)
- **Snow cannons that launch YOU** across a gap (aim and timing).
- **Lifts/elevators you have to time**: a platform or gondola rising and falling that you ski onto at the right moment.
- **Funnels and ice tubes**: enclosed chutes with banking and turns, then a drop.
- **Wall rides** on ice walls.
- **Moving platforms** over chasms (ice floes, moving lift platforms).
- **Collapsing paths** that crumble behind and ahead of you.
- **Rotating obstacles** (a spinning snowplow blade, a turning windmill or radio tower part).
- **Boost-gates** that are only possible with saved boost.
- **Speed traps**: sections where you have to slow down in time, then speed up again.
- **Narrow ridge runs** at high speed.
- **Dark or whiteout sections** where you read glowing markers.

### Checkpoints inside hard levels
If the last levels kill you multiple times, restarting a 3-minute level from the start is frustrating. Add **mid-level checkpoint flags** from about Level 11 on (1 checkpoint per level at first, 2–3 in the hardest levels). Dying returns you to the last checkpoint. Track deaths per level for stats and for the stars/rating.

### Measure difficulty, don't guess
For each level, compute and log: minimum safe-lane width, minimum reaction time to each hazard at expected speed, number of required precise inputs (timed jumps, boost gates, braking points), and density of hazards. Show a **difficulty graph across the 30 levels** in `PROGRESS.md`. It must rise steadily and steeply at the end. The autopilot (non-invincible) should start failing on late levels. That's expected and good. I'll do the final judgment by playing.

---

## 3. SPEED

**Problem:** Speed increases **too slowly**, and the top speed feels too low. The HUD caps around 99 mph.

What I want: speed keeps rising through the campaign, so late levels feel *really* fast. I suggested a max of 300–500 mph. **Give me your honest technical recommendation** before changing it, considering:
- At very high real speeds, chunk streaming, collision (tunneling through thin hazards) and hazard readability can break in Roblox. Test this.
- How fast it **feels** depends heavily on presentation: FOV, camera closeness and shake, speed lines, wind sound, snow spray, motion blur, and objects passing close by.
- The speedometer number is presentation: 1 stud = 0.28 m is our own choice, so the displayed mph can be tuned to feel right.

Then implement:
- A **steeper speed curve** across the campaign (each level faster than the last; late levels much faster than now), as fast as the engine can handle reliably.
- **Player control:** players can brake to slow down, and tuck and boost to go faster. Not everyone has to max out.
- **Boost pads / speed sections** in later levels for extreme top-speed moments.
- Better **speed feel** (all the presentation items above).
- Endless mode keeps ramping (with a reliable cap).

Report the real max speed in studs/s, what the HUD shows, and the test results at that speed.

---

## 4. LEVEL STRUCTURE: FINISH LINES AND A BASE CAMP

**Problem:** Levels currently flow straight into each other without a real break.

**New structure:**
- **Every level ends at a FINISH LINE** at the bottom of its run: a finish arch/banner, a crowd or flags, a celebration (confetti, sound, finish-time display). This is a clear "I beat this level" moment.
- After the finish, show the **level-complete screen**: time, stars, deaths, coins and rewards, with **Next Level** (big), **Replay** and **Base Camp** buttons.
- **Next Level** takes you straight to the next level's start (with a short, skippable transition such as riding the chairlift up). The player shouldn't have to walk anywhere to keep going.
- **BASE CAMP** (the bottom of the mountain): a small area at the base with a chairlift station and a **portal or gate for each level** the player has unlocked (locked ones visible but closed), plus the Endless gate. Players can go there to pick any level. Make it look like a real ski-resort base: lift station, ticket booth, benches, a small snack hut.
- **Keep it fast:** going from crash/finish to playing again must still take only a few seconds.

**ENDLESS is now a completely separate game mode:**
- Entered from its own gate/portal in the lodge area or Base Camp (unlocked after Level 30, as before).
- Its own HUD label, its own distance leaderboard, its own rewards.
- Procedural, ramping difficulty and speed forever, using every mechanic and set piece from the campaign.
- Campaign and Endless never blend into each other.

If you see a better way to do this structure, tell me your reasoning before building it.

---

## 5. LOBBY: FULL REVAMP INTO A BIG SKI LODGE

**Problem:** The lobby looks horrible. It's **too blocky**; it looks *below* low-poly. Buildings look like boxes with no detail or thought. **Some areas have no flooring.** The snowboards have no detail. There's a random **season pass stand** with **supports that aren't attached**. Your audits passed, but the lobby doesn't look good.

**Art direction change:** "Low-poly" was the wrong word. I want **stylized with realism, roughly medium-poly.** **You may raise the polygon count and add detail and realism** wherever it makes things look better (still within performance limits, and test the FPS). Look at popular, polished Roblox games' lobbies and real ski resorts for references.

**New concept: you spawn INSIDE a big Summit Lodge.** Like a real ski resort, you start indoors, get your gear and head out:
- **Inside the lodge** (big, warm, detailed): spawn area, the **ski shop**, **locker/outfits**, **Crate Co.** and the **crate shrine**, the quest board, leaderboards, a fireplace, wooden beams, rugs, tables and a café counter, windows looking out at the mountain, and gear racks with detailed skis and snowboards. All menus are reached from here.
- **The way out:** big doors leading onto the deck and the start area.
- **Outside**:
  - Frosty Rink (keep).
  - The big Daily Spin wheel (keep).
  - Summit Express chairlift (keep; make it look real).
  - **Outdoor eating area** with tables and parasols.
  - **Steaming hot tub/pool** with steam particles.
  - More activities: a snowball-throwing spot, a sled hill, a photo-spot sign, a fire pit with benches.
  - The start gate to the levels and the way to Base Camp/Endless.
- Keep the ideas I like: **Frosty Rink, Crate Shrine, Ski Shop, Crate Co., Summit Lodge.**
- **Every surface has flooring**: no gaps or missing floor anywhere. Snowboards and skis are real models (bindings, edges, graphics). **No unattached supports or legs anywhere.** Remove the random season-pass stand, or rebuild it properly where it makes sense.
- Use Creator Store models where they're clearly better than what you can build (free-model rules apply: no scripts, check triangles, consistent style).

---

## 6. NEW AUDITS (ADD THESE; RUN AFTER EVERY PHASE)

1. **Attachment audit:** for every model (lobby, assets and runtime-built set pieces), build a contact graph of its parts. **Every part must touch or overlap at least one other part of the same model**, and the model must be one connected piece, unless a part is explicitly marked as free (e.g. falling debris). This catches loose snowman noses, penguin flippers, Christmas lights, floating tags and unattached stand legs.
2. **Runtime grounding audit:** for every set-piece object built by the renderer (vehicles, logs, rocks, barriers, signs, props), check that its lowest points touch the terrain/surface (wheels on snow, logs on ground). Report floating or sunk objects per section.
3. **Pop-in audit:** for every moving or spawned set piece (chairs, penguins, vehicles, snowmobiles, sleds), check it is created *before* it can be in the player's view (distance and fog), and despawned only after it's out of view.
4. **Camera clipping audit:** during events (avalanche, bear, yeti) and in tunnels and caves, sample the camera position and check it's never inside an event or hazard volume.
5. **Ramp launch test:** for every ramp type, a real-simulation launch at low/medium/high speed. Log the launch height and airtime. Any ramp that doesn't launch upward fails.
6. **Reference check (manual, required):** for every model you change, put a screenshot next to a reference photo in `qa/rev3/` and write one line on how it compares. If it doesn't clearly look like the real thing, it's not done.

---

## 7. ANIMATIONS

The animations don't look horrible, but they're not great. Your report says "player animations didn't need publishing after all" because the game uses bundled poses. **I want real, better animations anyway.** The game is owned by the **Jack's Motion** group, so:
- Make proper animations (ski stance, carves, tuck, jump pop, airborne grabs, landing absorb, crash tumble, recovery, finish-line celebration).
- If they need to be uploaded under Jack's Motion, **give me exact step-by-step instructions** and I'll publish them, then you wire in the IDs. If the bundled-pose system gives equal quality, show me side-by-side proof.
- Compare against real skiing videos/photos. The pose should instantly read as skiing.

---

## 8. PHASE ORDER

1. **Phase A: audits + quick bug fixes.** Add the new audits (section 6) and run baselines. Fix bugs 1–18 from section 1. Do the lobby penguins (19) as part of phase E.
2. **Phase B: level structure.** Finish lines, level-complete screen, Next Level flow, Base Camp, Endless as a separate mode (section 4).
3. **Phase C: difficulty + speed.** Difficulty curve, late-game mechanics, checkpoints, speed recommendation then implementation, difficulty graph (sections 2–3). Rebalance levels in batches of 5, hardest last.
4. **Phase D: set-piece art.** Chairlift, rockfall cave-in, icicles, frozen pond/river, penguins, background skiers, Candy Cane Lane (section 1 visuals).
5. **Phase E: lobby revamp** (section 5), including the lobby penguins.
6. **Phase F: animations** (section 7).
7. **Phase G: final QA.** All audits, every level played start to finish (non-invincible autopilot logs + my playtest), the difficulty graph, performance in a focused session, and an updated `TEST_CHECKLIST.md`.

After each phase: update `PROGRESS.md`, `PROJECT_SUMMARY.md` and `TEST_CHECKLIST.md`, ask me for a .rbxl backup, and stop so I can playtest before the next phase if the phase changed how the game plays.

## 9. REPORT FORMAT

At the end of each phase: what changed, before/after screenshots (paths), audit counts, anything you couldn't do and why, and exactly what I should check in my playtest. **Nothing is "done" until I've checked it.**
