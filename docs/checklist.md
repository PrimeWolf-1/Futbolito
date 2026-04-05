# Build Checklist

## Build Preferences

- **Build mode:** Autonomous
- **Comprehension checks:** N/A (autonomous mode)
- **Git:** Commit after each item with message: "Complete step N: [title]". GitHub repo to be created in item 1.
- **Verification:** Yes — checkpoints after items 4, 7, and 9. Agent pauses, summarizes what was built, learner verifies before continuing.
- **Check-in cadence:** N/A (autonomous mode)

---

## Checklist

- [x] **1. HTML scaffold + screen controller + GitHub repo**
  Spec ref: `spec.md > Architecture Overview` + `spec.md > Screens > Screen Controller`
  What to build: Create `futbolito.html` with embedded CSS and JS. Add Matter.js CDN script tag. Set up four screen divs (`#screen-intro`, `#screen-setup`, `#screen-game`, `#screen-win`) with class `.screen` — all hidden by default except `#screen-intro`. Implement `Screens.show(id)` utility. Stub out empty `CONFIG`, `STATE`, and `canvas` setup. Initialize the `<canvas id="game-canvas">` element and get its 2D context. Create a GitHub repo named `futbolito`, initialize git in the project folder, add the remote, push `futbolito.html` and the `docs/` folder as the first commit.
  Acceptance: Opening `futbolito.html` in Chrome shows `#screen-intro` (can be a blank colored div). Running `Screens.show('screen-setup')` in the browser console hides intro and shows setup. GitHub repo exists with first commit containing `futbolito.html` and `docs/`.
  Verify: Open `futbolito.html` in Chrome. Open DevTools console. Run `Screens.show('screen-setup')` — confirm `#screen-intro` disappears and `#screen-setup` appears. Check GitHub repo URL — first commit is visible.

- [x] **2. Physics engine + board canvas render**
  Spec ref: `spec.md > Physics > Engine Setup` + `spec.md > Physics > Bodies` + `spec.md > Renderer > Draw Loop` + `spec.md > Renderer > drawBoard()`
  What to build: Initialize Matter.js engine with `gravity.y = 0`. Create the `gameLoop()` function using `requestAnimationFrame` — calls `Matter.Engine.update(engine, 1000/60)` each frame. Implement `drawBoard()`: fill canvas with board skin (use `'worn-felt'` — deep green `#2d5a1b` fill, nylon string border 8px inside edges, off-white `#e8dcc8`, lineWidth 3, goal mouths as rectangular cutouts at each end ~60px wide). Place four static wall bodies outside canvas edges. Place two goal sensor bodies (`isSensor: true`, labels `'goal-p1'` and `'goal-p2'`) just inside each goal opening. Create marble body (circle, radius 12, restitution 0.75, frictionAir 0.015) at canvas center. Implement `drawMarble()` with radial gradient glass effect. Place hardcoded test nails from 4-3-3 formation for one player and draw them as grey circles. Start the game loop.
  Acceptance: Canvas shows a green board with visible off-white border, goal cutouts at each end, nails in a formation pattern on one half, marble at center with glass gradient. Physics engine is running.
  Verify: Open `futbolito.html` in Chrome. Board renders with border, goals, nails, and marble. Open console, run `Matter.Body.setVelocity(marble, {x: 5, y: 3})` — marble moves and bounces off walls. Nails are static (marble bounces off them).

- [x] **3. Formation data + setup screen**
  Spec ref: `spec.md > Formations` + `spec.md > Screens > #screen-setup`
  What to build: Implement the `Formations` object with `'4-3-3'` and `'3-5-2'` coordinate arrays as specified. Build `#screen-setup` with two-column flex layout (P1 left, P2 right). Each column: name `<input type="text">` + two formation cards side by side, each card containing a label and a mini `<canvas>` (~120×80px) showing the nail dot preview. Draw formation previews using the same `Formations` arrays scaled to mini canvas dimensions (plain Canvas 2D, no Matter.js). Clicking a formation card adds `.selected` CSS class (border highlight). "Start Match" button validates both names non-empty AND both formations selected. On valid submit: call `Game.init()` — destroy any existing physics bodies, create nail bodies for both players (P2 mirrored: `mirroredX = 1 - x`), assign board skin and scene randomly, set `STATE.activePlayer = 0`, set `STATE.turnStartTime = Date.now()`, start game loop, call `Screens.show('screen-game')`.
  Acceptance: Setup screen shows side-by-side two-column layout. Formation cards show dot previews of each nail layout. Selecting a card highlights it. Start Match with incomplete inputs does nothing. With both names and formations selected, clicking Start Match launches the game with both players' nail formations visible on their correct halves of the board.
  Verify: Reload app — setup screen appears (skip intro via click). Fill in two names, select formations for both players, click Start Match. Board shows with P1 nails on left half, P2 nails (mirrored) on right half. Marble at center.

- [x] **4. Slingshot input + turn system**
  Spec ref: `spec.md > Game Module > Slingshot Input` + `spec.md > Game Module > Turn Flow` + `spec.md > Renderer > drawAimLine()` + `spec.md > Renderer > drawIdleGlow()`
  What to build: Wire `canvas.mousedown`: if click is within marble radius + 4px AND `STATE.activePlayer` matches the event — begin drag (`STATE.isDragging = true`, `STATE.dragStart = marble.position`). `canvas.mousemove` during drag: update `STATE.dragCurrent`, compute aim vector (marble.position minus dragCurrent, reversed), clamp magnitude to `MAX_DRAG_DISTANCE` (120px), set `STATE.aimTarget`. `canvas.mouseup` during drag: compute power = magnitude / MAX_DRAG_DISTANCE (0–1), set velocity via `Matter.Body.setVelocity`, clear drag state. Implement `drawAimLine()`: dotted line from marble center to aimTarget, visible only while `STATE.isDragging`. Each frame: if marble speed < `VELOCITY_THRESHOLD` (0.5) and marble was moving → `Game.endTurn()` (toggle `STATE.activePlayer`, reset `STATE.turnStartTime`, clear dirty/idle state). Idle check each frame: if `Date.now() - STATE.turnStartTime > 5000` and `!STATE.isDragging` → `STATE.idleGlowActive = true`. Implement `drawIdleGlow()`: sin-wave green pulse around marble. Add `#scoreboard` DOM element (absolutely positioned over canvas) showing both player names.
  Acceptance: Active player can click and drag the marble — dotted aim line appears during drag showing direction. Releasing fires the marble. Inactive player clicks do nothing. After marble stops moving, turn switches. After 5 seconds of inactivity on a turn, green glow pulses around marble. Scoreboard shows both player names.
  Verify: Start a match. P1 drags and shoots — aim line visible, marble fires in aimed direction, bounces off nails and walls. Marble slows and stops — turn switches to P2. Sit idle for 5+ seconds — green glow appears around marble. Confirm P2 cannot interact with marble during P1's turn.

---

### ✅ CHECKPOINT 1 — After item 4
*Agent pauses here. Verify: core gameplay loop is working — aim, shoot, bounce, turn switch, idle reminder.*

---

- [x] **5. Goal detection + scoreboard update**
  Spec ref: `spec.md > Physics > Collision Events` + `spec.md > Game Module > Goal Handling` + `spec.md > Screens > #screen-game`
  What to build: Confirm goal sensor bodies are placed correctly from item 2 (adjust positioning if needed to match the visual goal openings). Wire `Matter.Events.on(engine, 'collisionStart')` — check pair labels for marble + `'goal-p1'` or `'goal-p2'` → call `Game.onGoal(scoringPlayer)`. Implement `Game.onGoal(scoringPlayer)`: increment `STATE.scores[scoringPlayer - 1]`, update `#scoreboard` DOM immediately to show current score (e.g. "Rudy 2 — Carlos 1"), then call `Celebration.trigger(scoringPlayer)`.
  Acceptance: When the marble enters a goal opening, the scoreboard score increments immediately for the correct player. Goal detection fires reliably — marble visually entering the goal always triggers it, and the marble can't trigger the wrong goal.
  Verify: Start a match. Use console to run `Matter.Body.setVelocity(marble, {x: -15, y: 0})` to fire marble at P1's goal — scoreboard increments P2's score. Repeat for opposite goal — P1's score increments.

- [x] **6. Goal celebration**
  Spec ref: `spec.md > Celebration Module` + `spec.md > Renderer > drawCelebration()`
  What to build: Implement `Celebration.trigger(scoringPlayer)`: set `STATE.celebrationActive = true` and `STATE.celebrationStartTime = Date.now()`. Spawn ~60 confetti particles at marble position (store in `STATE.confettiParticles` — each with vx, vy, gravity, color, size, rotation as specified). Set `STATE.nailJumpActive = true`. Show DOM goal text overlay: large "GOAL!!" + scorer's name, fade in via CSS. Reset marble to board center with zero velocity via `Matter.Body.setPosition` and `Matter.Body.setVelocity`. Set 3-second auto-end `setTimeout` → `Celebration.end()`. Add click listener on canvas → `Celebration.end()`. In `Celebration.end()`: clear timer, remove click listener, clear confetti array, set `nailJumpActive = false`, hide goal text. If `STATE.pendingWin` set: call `Game.triggerWin(pendingWin)`. Else: call `Game.endTurn()`. Draw confetti particles each frame in `drawCelebration()` — update positions with gravity. In `drawNails()` during celebration: apply `offsetY = Math.sin(t * 12) * 6 * Math.max(0, 1 - t/0.6)` to nail y positions.
  Acceptance: Scoring a goal triggers: confetti burst from marble position, nails jump (animated, dampens out), "GOAL!!" + scorer name overlay fades in. Auto-ends after 3 seconds — OR clicking skips it. After end: marble is at board center, correct player starts their turn, goal text is gone.
  Verify: Score a goal — confirm confetti, nail jump, text, and auto-end all work. Score another goal, click during celebration — confirms skip works. Marble is at center and turn starts correctly after each celebration.

- [x] **7. Win condition + win screen**
  Spec ref: `spec.md > Game Module > Goal Handling` + `spec.md > Screens > #screen-win`
  What to build: In `Game.onGoal()`, after incrementing score: if `STATE.scores[scoringPlayer - 1] >= 3` → set `STATE.pendingWin = scoringPlayer` (celebration handles the transition). In `Celebration.end()` when `STATE.pendingWin` is set: populate `#screen-win` DOM — winner name (large), final score string (e.g. "Rudy 3 — Carlos 1"), "Play Again" button — then call `Screens.show('screen-win')`. Implement `Game.reset()`: stop game loop, destroy all physics bodies, reset `STATE.scores` to `[0, 0]`, clear all celebration/bird state. Wire "Play Again" button: call `Game.reset()`, repopulate name inputs from `STATE.playerNames`, reset formation selections to unselected, call `Screens.show('screen-setup')`.
  Acceptance: First player to 3 goals triggers: goal celebration plays, then win screen appears with correct winner name and final score. "Play Again" returns to setup screen with names pre-filled and formation cards reset to unselected.
  Verify: Score 3 goals for one player (use console velocity trick to speed this up). Win screen appears after celebration with correct winner and score. Click Play Again — setup screen shows with names intact, formations unselected.

---

### ✅ CHECKPOINT 2 — After item 7
*Agent pauses here. Verify: complete match lifecycle works — setup → gameplay → goal scoring → celebration → win screen → play again.*

---

- [x] **8. Intro cinematic**
  Spec ref: `spec.md > Screens > #screen-intro`
  What to build: Style `#screen-intro` as full-viewport. Add stadium layer (CSS radial gradient — dark green/crowd feel) and board layer (the board drawn or represented). Implement `@keyframes introDive` as specified: scale 1→12, translateY 0→-20%, opacity progression over ~4.5 seconds — stadium layer fades out, board layer fades in as scale increases. Wire `animationend` event → `Screens.show('screen-setup')`. Wire `click` and `keydown` on document during intro → clear animation classes → `Screens.show('screen-setup')` immediately. `#screen-intro` is the default visible screen on page load.
  Acceptance: Page load shows intro cinematic — stadium view zooms continuously into board close-up over ~4.5 seconds, then setup screen appears automatically. Clicking or pressing any key at any point during the intro skips it immediately.
  Verify: Hard-reload the page (`Cmd+Shift+R`) — intro cinematic plays automatically and transitions to setup. Hard-reload again, click immediately — skips straight to setup with no flicker.

- [x] **9. Bird drop event**
  Spec ref: `spec.md > Bird Drop Module`
  What to build: Implement `BirdDrop.trigger()`: set `STATE.birdActive = true`, initialize `STATE.bird = { x: -40, y: randomBetween(80, CANVAS_HEIGHT - 80), hasDropped: false }`, set `STATE.birdFrame = 0`. Implement `BirdDrop.update()` (called each frame inside `gameLoop`): increment `bird.x += 3.5`, toggle `STATE.birdFrame` every 12 frames. At `bird.x > CANVAS_WIDTH * 0.4` and `!bird.hasDropped`: set `STATE.dropping = { x: bird.x, y: bird.y + 20 }`, set 3-second timeout to null `STATE.dropping`. When `bird.x > CANVAS_WIDTH + 40`: set `STATE.birdActive = false`. Implement `drawBird()`: two-frame silhouette using Canvas 2D paths — frame 0 wings level, frame 1 wings slightly raised. Draw dropping as a small greenish-yellow splat circle when `STATE.dropping` is set. Marble interaction each frame: if `STATE.dropping` and `distance(marble.position, STATE.dropping) < 25` and `!STATE.droppingContactHandled` → set `marble.frictionAir = 0.06`, set `STATE.droppingContactHandled = true`. If dropping lands directly on marble at drop moment → `STATE.marbleDirty = true` (draw trail in `drawMarble()`). Restore `frictionAir = 0.015` and reset `droppingContactHandled` in `Game.endTurn()`. Wire `setInterval` every 5000ms: if `!STATE.birdActive && marbleIsMoving() && !STATE.celebrationActive && Math.random() < 0.08` → `BirdDrop.trigger()`.
  Acceptance: Bird randomly flies across the board mid-game (temporarily raise probability to 0.9 for testing). Bird displays as a two-frame flapping silhouette. A dropping appears around 40% across the board and disappears after 3 seconds. Marble rolling through the dropping moves noticeably slower for the remainder of that turn. If dropping lands directly on marble, marble leaves a dirty trail on the next shot. Trail clears after that turn.
  Verify: Temporarily set trigger probability to `0.9`. Start a match and shoot the marble. Bird should appear within first 5-second check. Observe: flight animation, dropping at 40%, marble slowdown on contact. Confirm drop disappears after 3 seconds. Restore probability to `0.08` and commit.

---

### ✅ CHECKPOINT 3 — After item 9
*Agent pauses here. Verify: full game is feature-complete — intro, setup, gameplay, goals, celebration, win screen, and bird drop all working.*

---

- [ ] **10. Netlify deploy + Devpost submission**
  Spec ref: `prd.md > What We're Building`
  What to build: Confirm GitHub repo is current — commit any remaining changes, push `futbolito.html` and `docs/` folder. Deploy: drag `futbolito.html` onto `app.netlify.com/drop` → copy the live URL. Then fill out Devpost submission: project name "Futbolito", tagline (e.g. "The street game from the neighborhood, digitized"), project story using `scope.md` and `prd.md` as source — explain what it is, why it matters, what you learned building it with spec-driven development. Add built-with tags: HTML, CSS, JavaScript, Matter.js, Netlify. Screenshot the board, the celebration moment, and the win screen for the image gallery. Upload `docs/` folder artifacts (scope, PRD, spec, checklist) as supplemental files. Link the GitHub repo. Link the Netlify URL. Review all fields and submit.
  Acceptance: GitHub repo is public and up to date. Netlify URL is live — game loads and is fully playable in an incognito window. Devpost submission has green "Submitted" badge, all required fields complete, screenshots showing the board and celebration moment, and both the repo and live URL linked.
  Verify: Open the Netlify URL in an incognito window — intro plays, setup works, full match is playable. Open Devpost submission page — green "Submitted" badge is visible. Read the project description aloud — would a stranger who's never heard of Futbolito understand what it is and why it's cool?
