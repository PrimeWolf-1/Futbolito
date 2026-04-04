# Futbolito — Technical Spec

## Stack

| Layer | Choice | Rationale |
|---|---|---|
| Language | HTML + CSS + JavaScript | Rudy's established build pattern — single-file, no toolchain |
| Physics | Matter.js 0.20.0 | Handles marble/nail/wall collision math; custom renderer controls visuals |
| Rendering | Canvas 2D API (native) | Full visual control — Matter.js physics runs invisibly |
| UI / Screens | DOM + CSS | Four screen divs, shown/hidden with JS |
| Deployment | Netlify drag-and-drop | No build step — drop the `.html` file, get a shareable URL |

**Key documentation:**
- Matter.js API docs: https://brm.io/matter-js/docs/
- Matter.js CDN: `https://cdn.jsdelivr.net/npm/[email protected]/build/matter.min.js`
- Canvas 2D API: https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D
- Netlify drop deploy: https://app.netlify.com/drop

---

## Runtime & Deployment

- **Runtime:** Browser (desktop). No Node, no server, no build step.
- **Deployment:** Netlify drag-and-drop. Drag `futbolito.html` onto netlify.com/drop → live URL in ~30 seconds.
- **No API keys required.** All logic is client-side. Matter.js loads from CDN.
- **Browser target:** Chrome/Firefox/Safari desktop. No mobile support in v1.

---

## Architecture Overview

```
futbolito.html
│
├── <head>
│   ├── Matter.js (CDN)
│   └── <style> — all CSS embedded
│
└── <body>
    ├── DOM Screens (one visible at a time)
    │   ├── #screen-intro        CSS zoom animation → auto-advances to setup
    │   ├── #screen-setup        Name inputs + formation picker → Start Match
    │   ├── #screen-game         Canvas + HUD overlay (score, names, turn cue)
    │   └── #screen-win          Winner name + final score + Play Again
    │
    ├── <canvas id="game-canvas">
    │   └── Custom draw loop reads Matter.js body positions each frame
    │
    └── <script>
        ├── CONFIG               Constants (canvas size, speeds, thresholds)
        ├── STATE                Single source of truth for all game data
        ├── Screens              show() / hide() screen transitions
        ├── Intro                CSS animation trigger + skip handler
        ├── Setup                Form validation + formation preview canvases
        ├── Formations           {x,y} coordinate arrays for each formation
        ├── Physics              Matter.js engine, bodies, collision events
        ├── Renderer             requestAnimationFrame draw loop
        ├── Game                 Turn logic, idle reminder, goal detection
        ├── Celebration          Goal animation, confetti, nail jump
        └── BirdDrop             Random trigger, flight animation, drop effect
```

**Data flow — one shot cycle:**
```
Player drags marble
  → mousedown/mousemove/mouseup events
  → Game module calculates velocity vector
  → Matter.Body.setVelocity(marble, vector)
  → Matter.js Engine.update() runs each frame
  → Physics resolves collisions (marble ↔ nails, marble ↔ walls, marble ↔ goal sensors)
  → Renderer reads body positions → draws frame
  → Game module checks marble velocity each frame
  → velocity < VELOCITY_THRESHOLD → switch active player
  → collision with goal sensor → Celebration.trigger()
```

---

## Screens

### Screen Controller

Utility that manages which screen is visible. Only one screen is visible at a time.

```js
// Implementation pattern
const Screens = {
  show(id) {
    document.querySelectorAll('.screen').forEach(s => s.style.display = 'none');
    document.getElementById(id).style.display = 'block';
  }
};
```

All four screen divs share the class `.screen`. Default: all hidden except `#screen-intro`.

---

### #screen-intro

Implements `prd.md > Intro & Mood-Setting`.

- Full-viewport div with a background representing a stadium overhead view (CSS gradient or image).
- Inner div with the board image — CSS `transform: scale() translateY()` keyframe animates the zoom from wide stadium view into the board close-up over ~4.5 seconds.
- Two layers cross-fade as scale increases: stadium layer fades out, board layer fades in.
- On animation end (`animationend` event) → `Screens.show('screen-setup')`.
- Click or keydown at any point → clear animation → `Screens.show('screen-setup')` immediately.

```css
@keyframes introDive {
  0%   { transform: scale(1) translateY(0); opacity: 0.6; }
  60%  { transform: scale(4) translateY(-10%); opacity: 1; }
  100% { transform: scale(12) translateY(-20%); opacity: 0; }
}
```

---

### #screen-setup

Implements `prd.md > Match Setup`.

- Two-column flex layout: Player 1 (left) | Player 2 (right).
- Each column:
  - `<input type="text" placeholder="Enter name">` — required, validated on Start Match.
  - Two formation cards side by side. Each card: a label + a mini `<canvas>` (~120×80px) showing nail dot preview.
  - Selecting a formation card adds `.selected` class (CSS border highlight).
- "Start Match" button: validates both names non-empty AND both formations selected → calls `Game.init()`.
- On Play Again: name inputs retain their values (populated from `STATE.playerNames`). Formation selections reset to unselected — player must re-pick.
- Board skin and ambient scene assigned randomly at `Game.init()` — not player-facing.

**Formation preview canvas:** Uses the same `Formations` coordinate arrays as the game, scaled to fit the mini canvas dimensions. Drawn with plain Canvas 2D — no Matter.js.

---

### #screen-game

Implements `prd.md > Board & Match View`, `prd.md > Turn-Based Shooting Mechanic`, `prd.md > Goal Scoring & Celebration`, `prd.md > Bird Drop Event`.

- Full-viewport screen. Background CSS class set at game init: `.scene-backyard` or `.scene-street-night` (randomly assigned).
- `<canvas id="game-canvas">` centered inside — fixed pixel dimensions (e.g. 900×600px), board drawn to fill canvas.
- HUD overlay (DOM elements, absolutely positioned over canvas):
  - `#scoreboard` — "PlayerOne 2 — PlayerTwo 1", always visible
  - No static turn indicator — idle glow on canvas handles turn reminder (see Game module)

---

### #screen-win

Implements `prd.md > Win Condition & End Screen`.

- DOM elements: winner name (large), final score string (e.g. "Rudy 3 — Carlos 1"), "Play Again" button.
- "Play Again" → `Game.reset()` → repopulate name inputs from `STATE.playerNames` → `Screens.show('screen-setup')`.

---

## Formations

Implements formation data used by `prd.md > Match Setup` and `prd.md > Board & Match View`.

Nail positions are stored as normalized `{x, y}` coordinates in the range `[0, 1]` — relative to each player's half of the board. At render time, positions are scaled to canvas pixel coordinates.

```js
const Formations = {
  "4-3-3": [
    // Goalkeeper
    { x: 0.1, y: 0.5 },
    // Defenders (4)
    { x: 0.3, y: 0.2 }, { x: 0.3, y: 0.4 },
    { x: 0.3, y: 0.6 }, { x: 0.3, y: 0.8 },
    // Midfielders (3)
    { x: 0.55, y: 0.25 }, { x: 0.55, y: 0.5 }, { x: 0.55, y: 0.75 },
    // Forwards (3)
    { x: 0.75, y: 0.2 }, { x: 0.75, y: 0.5 }, { x: 0.75, y: 0.8 }
  ],
  "3-5-2": [
    // Goalkeeper
    { x: 0.1, y: 0.5 },
    // Defenders (3)
    { x: 0.3, y: 0.25 }, { x: 0.3, y: 0.5 }, { x: 0.3, y: 0.75 },
    // Midfielders (5)
    { x: 0.52, y: 0.1 }, { x: 0.52, y: 0.3 }, { x: 0.52, y: 0.5 },
    { x: 0.52, y: 0.7 }, { x: 0.52, y: 0.9 },
    // Forwards (2)
    { x: 0.75, y: 0.35 }, { x: 0.75, y: 0.65 }
  ]
};
```

Player 2's formation is mirrored horizontally: `mirroredX = 1 - x`. Applied at physics body creation.

---

## Physics

Implements `prd.md > Turn-Based Shooting Mechanic` and `prd.md > Board & Match View`.

### Engine Setup

```js
const engine = Matter.Engine.create();
engine.gravity.y = 0;  // Top-down view — no gravity
const world = engine.world;
```

Matter.js docs — Engine: https://brm.io/matter-js/docs/classes/Engine.html

### Bodies

| Body | Type | Properties |
|---|---|---|
| Marble | `Matter.Bodies.circle` | radius: 12, restitution: 0.75, frictionAir: 0.015, label: 'marble' |
| Nail (any) | `Matter.Bodies.circle` | radius: 8, isStatic: true, restitution: 0.7, label: 'nail' |
| Wall (×4) | `Matter.Bodies.rectangle` | isStatic: true, thickness: 20, restitution: 0.6 |
| Goal sensor (×2) | `Matter.Bodies.rectangle` | isStatic: true, isSensor: true, label: 'goal-p1' / 'goal-p2' |

- Walls are placed outside the visible board edges — marble cannot leave the board.
- Goal sensors sit just inside the goal opening at each end. Width ~60px (goal mouth size), height ~20px.
- Nail positions derived from `Formations` arrays, scaled from normalized coords to canvas pixels at game init.

### Collision Events

```js
Matter.Events.on(engine, 'collisionStart', (event) => {
  event.pairs.forEach(pair => {
    const labels = [pair.bodyA.label, pair.bodyB.label];
    if (labels.includes('marble') && labels.includes('goal-p1')) {
      Game.onGoal(1); // Player 1 scored
    }
    if (labels.includes('marble') && labels.includes('goal-p2')) {
      Game.onGoal(2); // Player 2 scored
    }
  });
});
```

Matter.js docs — Events: https://brm.io/matter-js/docs/classes/Events.html

### Speed Cap

Max marble velocity enforced after `setVelocity` call — prevents tunneling through nails:

```js
const MAX_SPEED = 18; // px per frame — tunable during build
const vel = marble.velocity;
const speed = Math.sqrt(vel.x ** 2 + vel.y ** 2);
if (speed > MAX_SPEED) {
  const scale = MAX_SPEED / speed;
  Matter.Body.setVelocity(marble, { x: vel.x * scale, y: vel.y * scale });
}
```

### Physics Update Loop

Matter.js is driven manually (not via `Matter.Runner`) so we control the timestep:

```js
// Inside requestAnimationFrame loop:
Matter.Engine.update(engine, 1000 / 60); // Fixed 60fps timestep
```

---

## Renderer

### Draw Loop

```js
function gameLoop() {
  Matter.Engine.update(engine, 1000 / 60);
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  drawBoard();       // board skin texture/color
  drawNails();       // styled circles at nail body positions
  drawMarble();      // marble with optional dirty trail
  drawAimLine();     // visible during drag only
  drawIdleGlow();    // green pulse if idle > 5s
  drawCelebration(); // confetti + nail jump if active
  drawBird();        // bird + dropping if active

  requestAnimationFrame(gameLoop);
}
```

### drawBoard()

- Fills canvas with board skin. Two options (randomly assigned at game init):
  - `'worn-felt'` — deep green fill (`#2d5a1b`) with subtle noise/grain overlay (CSS or canvas noise pattern)
  - `'rustic-wood'` — warm brown fill (`#8B5A2B`) with horizontal grain lines drawn at slight angles
- Nylon string border: 4 stroked lines ~8px inside the wall physics bodies, off-white (`#e8dcc8`), lineWidth 3.
- Goal mouths: rectangular cutouts in the border string at each end, ~60px wide.

### drawNails()

Each nail body gets visual personality assigned once at game init (`nail.visualStyle`):

```js
// Assigned randomly at init, stored on the body object
nail.visualStyle = {
  tint: randomFrom(['#8a8a8a', '#7a5c3a', '#6b6b5a']), // steel, rust, aged
  radius: nail.circleRadius + randomBetween(-1, 2),     // slight size variance
  rotation: randomBetween(-0.3, 0.3)                    // head tilt
}
```

All nails have identical physics (`circleRadius: 8`). Visual variance is cosmetic only.

### drawMarble()

- Filled circle, radius 12. Gradient fill to simulate glass: radial gradient from near-white highlight to deep color (e.g. `#1a3a6b` blue).
- If `STATE.marbleDirty`: draw the last 6 marble positions as faded circles (opacity decreasing from 0.4 → 0.05).

```js
// Trail: stored in STATE.marbleTrail = [] (push each frame, slice to last 6)
STATE.marbleTrail.forEach((pos, i) => {
  ctx.globalAlpha = (i / STATE.marbleTrail.length) * 0.4;
  ctx.beginPath();
  ctx.arc(pos.x, pos.y, 10, 0, Math.PI * 2);
  ctx.fillStyle = '#8B6914'; // dirty yellow-brown
  ctx.fill();
});
ctx.globalAlpha = 1;
```

### drawAimLine()

Visible only while `STATE.isDragging`:

```js
// Dotted line from marble center toward aim target
ctx.setLineDash([6, 4]);
ctx.strokeStyle = 'rgba(255,255,255,0.6)';
ctx.lineWidth = 2;
ctx.beginPath();
ctx.moveTo(marble.position.x, marble.position.y);
ctx.lineTo(aimTarget.x, aimTarget.y);
ctx.stroke();
ctx.setLineDash([]); // reset
```

### drawIdleGlow()

```js
// Pulse via sin wave over time
if (STATE.idleGlowActive) {
  const pulse = 0.4 + 0.4 * Math.sin(Date.now() / 200);
  ctx.beginPath();
  ctx.arc(marble.position.x, marble.position.y, 20, 0, Math.PI * 2);
  ctx.strokeStyle = `rgba(80, 220, 80, ${pulse})`;
  ctx.lineWidth = 3;
  ctx.stroke();
}
```

---

## Game Module

Implements `prd.md > Turn-Based Shooting Mechanic`, `prd.md > Goal Scoring & Celebration`, `prd.md > Win Condition & End Screen`.

### State (relevant fields)

```js
const STATE = {
  // Players
  playerNames: ['', ''],
  playerFormations: [null, null],
  scores: [0, 0],
  activePlayer: 0,           // 0 or 1

  // Turn
  turnStartTime: null,
  idleGlowActive: false,
  isDragging: false,
  dragStart: null,           // {x, y} — where drag began (marble position)
  dragCurrent: null,         // {x, y} — current mouse position

  // Marble
  marbleDirty: false,
  marbleTrail: [],

  // Celebration
  celebrationActive: false,
  celebrationTimer: null,

  // Bird
  birdActive: false,
  bird: { x: 0, y: 0, hasDropped: false },
  dropping: null,            // {x, y} or null
  droppingTimer: null,

  // Board
  boardSkin: 'worn-felt',    // or 'rustic-wood'
  ambientScene: 'backyard',  // or 'street-night'
};
```

### Turn Flow

```
Game.init()
  → create physics bodies from selected formations
  → assign board skin + scene randomly
  → STATE.activePlayer = 0 (P1 shoots first)
  → STATE.turnStartTime = Date.now()
  → start gameLoop()

Each frame:
  → if marble speed < VELOCITY_THRESHOLD (0.5) AND marble was moving:
      → Game.endTurn()

Game.endTurn()
  → STATE.marbleDirty = false (clear dirty effect)
  → STATE.activePlayer = 1 - STATE.activePlayer (toggle 0↔1)
  → STATE.turnStartTime = Date.now()
  → STATE.idleGlowActive = false

Idle check (each frame):
  → if Date.now() - STATE.turnStartTime > 5000 AND !STATE.isDragging:
      → STATE.idleGlowActive = true
```

### Slingshot Input

```
canvas.mousedown:
  → distance(event, marble.position) < marble.radius + 4?
      → if STATE.activePlayer matches event side: begin drag
      → else: ignore

canvas.mousemove (during drag):
  → STATE.dragCurrent = {x, y}
  → compute aimVector = marble.position - dragCurrent (reversed)
  → clamp magnitude to MAX_DRAG_DISTANCE (120px)
  → STATE.aimTarget = marble.position + aimVector (dotted line endpoint)

canvas.mouseup (during drag):
  → power = magnitude(aimVector) / MAX_DRAG_DISTANCE  // 0.0–1.0
  → velocity = normalize(aimVector) * power * MAX_SPEED
  → Matter.Body.setVelocity(marble, velocity)
  → STATE.isDragging = false
  → STATE.idleGlowActive = false
```

### Goal Handling

```
Game.onGoal(scoringPlayer)
  → STATE.scores[scoringPlayer - 1]++
  → update #scoreboard DOM immediately
  → Celebration.trigger(scoringPlayer)
  → if STATE.scores[scoringPlayer - 1] >= 3: Game.triggerWin(scoringPlayer)

Game.triggerWin(winner)
  → after Celebration completes:
      → populate #screen-win with winner name + score string
      → Screens.show('screen-win')
```

---

## Celebration Module

Implements `prd.md > Goal Scoring & Celebration`.

```js
Celebration = {
  trigger(scoringPlayer) {
    STATE.celebrationActive = true;
    this.spawnConfetti();        // fill STATE.confettiParticles array
    this.startNailJump();        // set STATE.nailJumpActive = true
    this.showGoalText(scoringPlayer); // DOM overlay fade-in
    // Reset marble position
    Matter.Body.setPosition(marble, BOARD_CENTER);
    Matter.Body.setVelocity(marble, { x: 0, y: 0 });

    STATE.celebrationTimer = setTimeout(() => this.end(), 3000);
    canvas.addEventListener('click', this._skipHandler);
  },

  end() {
    clearTimeout(STATE.celebrationTimer);
    canvas.removeEventListener('click', this._skipHandler);
    STATE.celebrationActive = false;
    STATE.confettiParticles = [];
    STATE.nailJumpActive = false;
    hideGoalText();
    // If win was flagged, show win screen now
    if (STATE.pendingWin) Game.triggerWin(STATE.pendingWin);
    else Game.endTurn(); // scoring team gets first turn
  }
};
```

### Confetti Particles

```js
// Spawned at trigger — ~60 particles
STATE.confettiParticles = Array.from({ length: 60 }, () => ({
  x: marble.position.x,
  y: marble.position.y,
  vx: randomBetween(-6, 6),
  vy: randomBetween(-10, -2),
  gravity: 0.3,
  color: randomFrom(['#e63946','#f4a261','#2a9d8f','#e9c46a','#ffffff']),
  size: randomBetween(4, 9),
  rotation: randomBetween(0, Math.PI * 2)
}));

// Each frame during celebration:
STATE.confettiParticles.forEach(p => {
  p.vy += p.gravity;
  p.x += p.vx;
  p.y += p.vy;
  // draw rectangle at p.x, p.y with p.color, p.size, p.rotation
});
```

### Nail Jump

```js
// In drawNails(), during celebration:
if (STATE.nailJumpActive) {
  const t = (Date.now() - STATE.celebrationStartTime) / 1000;
  offsetY = Math.sin(t * 12) * 6 * Math.max(0, 1 - t / 0.6); // dampens out
}
```

---

## Bird Drop Module

Implements `prd.md > Bird Drop Event`.

### Trigger

```js
// Checked every 5 seconds via setInterval
setInterval(() => {
  if (!STATE.birdActive && marbleIsMoving() && !STATE.celebrationActive) {
    if (Math.random() < 0.08) BirdDrop.trigger();
  }
}, 5000);
```

Expected frequency: ~once per 62 seconds on average. Tunable via the probability value.

### Flight & Drop

```js
BirdDrop = {
  trigger() {
    STATE.birdActive = true;
    STATE.bird = { x: -40, y: randomBetween(80, CANVAS_HEIGHT - 80), hasDropped: false };
    STATE.birdFrame = 0; // for wing flap animation (2 frames)
  },

  update() {
    if (!STATE.birdActive) return;
    STATE.bird.x += 3.5; // ~2.5s to cross 900px canvas

    // Wing flap — toggle every 12 frames
    if (frameCount % 12 === 0) STATE.birdFrame = 1 - STATE.birdFrame;

    // Drop at ~40% across canvas
    if (STATE.bird.x > CANVAS_WIDTH * 0.4 && !STATE.bird.hasDropped) {
      STATE.bird.hasDropped = true;
      STATE.dropping = { x: STATE.bird.x, y: STATE.bird.y + 20 };
      STATE.droppingTimer = setTimeout(() => { STATE.dropping = null; }, 3000);
    }

    // Bird exits screen
    if (STATE.bird.x > CANVAS_WIDTH + 40) STATE.birdActive = false;
  }
};
```

### Draw Bird

Two-frame silhouette animation using Canvas 2D paths — a simple black bird shape. Frame 0: wings level. Frame 1: wings slightly raised. ~15 lines of canvas path code.

### Marble Interaction

Checked each frame in the game loop:

```js
if (STATE.dropping) {
  const dist = distance(marble.position, STATE.dropping);

  // Marble rolls through dropping (center within ~25px)
  if (dist < 25 && !STATE.droppingContactHandled) {
    Matter.Body.set(marble, 'frictionAir', 0.06); // 4x normal drag
    STATE.droppingContactHandled = true;
  }

  // Dropping lands directly on marble (at drop moment)
  if (dist < marble.circleRadius + 8 && justDropped) {
    STATE.marbleDirty = true;
  }
}

// Restore frictionAir on turn end (in Game.endTurn())
Matter.Body.set(marble, 'frictionAir', 0.015);
STATE.droppingContactHandled = false;
```

---

## State — Full Shape

```js
const STATE = {
  // Players
  playerNames: ['', ''],          // string[2]
  playerFormations: [null, null],  // 'four-three-three' | 'three-five-two'
  scores: [0, 0],                  // number[2]
  activePlayer: 0,                 // 0 | 1

  // Turn
  turnStartTime: null,             // Date.now() timestamp
  idleGlowActive: false,
  isDragging: false,
  dragCurrent: null,               // {x, y} | null
  aimTarget: null,                 // {x, y} | null — dotted line endpoint

  // Marble
  marbleDirty: false,
  marbleTrail: [],                 // [{x,y}] last 6 positions
  droppingContactHandled: false,

  // Celebration
  celebrationActive: false,
  celebrationStartTime: null,
  celebrationTimer: null,
  confettiParticles: [],
  nailJumpActive: false,
  pendingWin: null,                // null | 1 | 2

  // Bird
  birdActive: false,
  birdFrame: 0,
  bird: { x: 0, y: 0, hasDropped: false },
  dropping: null,                  // {x, y} | null
  droppingTimer: null,

  // Board
  boardSkin: null,                 // 'worn-felt' | 'rustic-wood'
  ambientScene: null,              // 'backyard' | 'street-night'
};
```

---

## File Structure

```
futbolito/
├── futbolito.html          # The entire app — HTML + CSS + JS in one file
│
├── docs/                   # Hackathon planning artifacts
│   ├── learner-profile.md
│   ├── scope.md
│   ├── prd.md
│   └── spec.md             # This file
│
└── process-notes.md        # Learning journal (updated each phase)
```

That's the full project. One deliverable file. Everything else is documentation.

---

## Key Technical Decisions

### 1. Matter.js with custom renderer (not built-in renderer)
**Decision:** Use Matter.js physics engine but draw everything with the Canvas 2D API manually.
**Why:** Matter.js's built-in renderer draws grey shapes on white — incompatible with the handmade aesthetic. Custom rendering gives full visual control while keeping physics complexity out of our code.
**Tradeoff accepted:** More render code to write, but complete freedom over how everything looks.

### 2. Normalized formation coordinates {x, y} in [0,1]
**Decision:** Store nail positions as normalized floats, scale to canvas pixels at runtime.
**Why:** The same coordinate data drives both the mini formation preview canvas (120×80px) and the full game board (900×600px). Define once, draw anywhere.
**Tradeoff accepted:** One extra scaling step at render time — negligible cost.

### 3. Manual Matter.js update (not Matter.Runner)
**Decision:** Call `Matter.Engine.update(engine, 1000/60)` inside our own `requestAnimationFrame` loop instead of using `Matter.Runner`.
**Why:** Keeps physics and rendering in a single loop we control. Easier to pause, reset, and manage game states (celebration freeze, etc.).
**Tradeoff accepted:** We manage the timestep ourselves — fine for a fixed 60fps desktop target.

---

## Dependencies & External Services

| Dependency | Version | CDN | Docs | Notes |
|---|---|---|---|---|
| Matter.js | 0.20.0 | `cdn.jsdelivr.net/npm/matter-js@0.20.0/build/matter.min.js` | https://brm.io/matter-js/docs/ | Physics only. No API key. |
| Netlify | — | — | https://app.netlify.com/drop | Free tier. Drag-and-drop deploy. No account required for drop. |

No external APIs. No API keys. No accounts required to play.

---

## Open Issues

### 1. Exact nail formation coordinates need playtesting
The `{x, y}` values in the `Formations` object above are reasonable starting points but will need tuning once the board is rendered. Nails that are too close together will cluster the marble; too far apart and formations feel meaningless. Plan to iterate on these values early in the build before other systems depend on them.

### 2. Goal sensor positioning depends on final board layout
Goal mouth width (~60px) and sensor placement need to match the visual goal openings exactly. Off-by-a-few-pixels here causes the marble to visually enter a goal but not trigger detection (or worse, trigger without visually scoring). Verify sensor placement matches the drawn board during the Board & Match View checklist item.

### 3. Bird drop frequency tuning
The 8% per 5-second check is a starting estimate. In a 3-minute game (~36 five-second windows), this yields ~2-3 bird events — probably right. But playtest first. Too frequent = loses the surprise. Too rare = players never see it. Adjust the probability constant during build.

### 4. Intro cinematic scope vs. build time
The stadium-zoom-to-backyard intro is visually ambitious for a single CSS animation. If it's consuming build time, the fallback is a simpler fade-in of the board. The intro is mood-setting — important, but not load-bearing for gameplay. Don't let it block the core build.
