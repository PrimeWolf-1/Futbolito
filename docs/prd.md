# Futbolito — Product Requirements

## Problem Statement
People who grew up playing informal street games like Futbolito — a handmade board game using nails, a marble, and a popsicle stick — have no digital version that captures the authentic feel of the physical object. Existing browser-based soccer games either simulate real football (too complex) or look like software (too clean). Futbolito fills that gap: a two-player hot-seat game that feels like someone digitized the real street object, not designed a video game inspired by it.

---

## User Stories

### Epic: Intro & Mood-Setting

- As a first-time player, I want an intro cinematic that sets the mood before I touch any menus, so that I feel the game's world before I'm asked to interact with it.
  - [ ] On load, a top-down stadium broadcast view appears with a slowly panning camera
  - [ ] The camera zooms into a backyard/street setting where the board is sitting — a continuous dive from big-league stadium to street game
  - [ ] The intro plays automatically; clicking or pressing anything skips it immediately and jumps to the setup screen
  - [ ] If left alone, the intro completes and the setup screen appears automatically

---

### Epic: Match Setup

- As a player, I want to enter my name and pick my formation before the match starts, so that the game feels personalized and I have a strategic choice to make.
  - [ ] Setup screen shows both players' inputs simultaneously — Player 1 on the left, Player 2 on the right
  - [ ] Each player has a name text input field
  - [ ] Each player has a formation selector showing a visual preview of the nail layout — Player 1's formation preview is on the left, Player 2's is on the right, mirroring their board position
  - [ ] Two formation options are available per player (e.g. 4-3-3 and 3-5-2)
  - [ ] Player 1 always plays on the left side of the board; Player 2 always plays on the right — no side selection needed
  - [ ] A "Start Match" button launches the game once both names are entered
  - [ ] Board skin is assigned randomly at match start — not a player choice; shared for both players
  - [ ] If players return to this screen via "Play Again," their names persist but formations reset to default (must be re-selected)

---

### Epic: Board & Match View

- As a player, I want to see the full board at all times during the match, so that I can read the game state and plan my shot.
  - [ ] Top-down view of the entire board is always visible — nothing is off-screen
  - [ ] Nails are rendered with visual personality: some bent, some rusted, some split — purely aesthetic, all bounce identically
  - [ ] Board is set inside an ambient scene (backyard afternoon or street at night) — randomly assigned at match start
  - [ ] Nylon string borders are visible on all four sides; the marble cannot leave the board
  - [ ] Scoreboard is visible at all times during the match, showing both player names and their current score (e.g. "Rudy 2 — Carlos 1")
  - [ ] Goals are clearly visible at each end of the board

---

### Epic: Turn-Based Shooting Mechanic

- As the active player, I want to click and drag the marble to aim and shoot, so that I feel in control of my shot direction and power.
  - [ ] Only the active player can interact with the marble — if the inactive player clicks, nothing happens
  - [ ] Clicking directly on the marble initiates the drag; clicking anywhere else does nothing
  - [ ] Dragging back from the marble renders a visible slingshot string or dotted aim line showing direction and projected power
  - [ ] Drag distance determines shot power; there is a maximum drag distance enforced by the game
  - [ ] Releasing the mouse fires the marble in the aimed direction
  - [ ] The marble bounces off nails and nylon string borders until it comes to rest
  - [ ] Once the marble stops moving (velocity below threshold), the turn switches to the other player
  - [ ] If the active player has not taken a shot within 5 seconds of their turn starting, a subtle green glow or pulse appears around the marble as a gentle reminder — no text, no sound required, just the visual cue

---

### Epic: Goal Scoring & Celebration

- As a player, I want a big celebration moment when I score, so that scoring feels earned and exciting.
  - [ ] When the marble enters the goal, a celebration sequence triggers immediately
  - [ ] Celebration displays: large "GOAL!!" text, scorer's name, confetti burst, fan nails jumping animation, scoring player's nails reacting
  - [ ] Celebration plays for 3 seconds automatically, then resets
  - [ ] Clicking during the celebration skips it immediately and resets
  - [ ] After celebration/reset: marble returns to board center; the scoring team takes the first turn
  - [ ] Scoreboard updates immediately when the goal is scored (before or as the celebration begins)

---

### Epic: Win Condition & End Screen

- As a player, I want the game to end clearly and feel satisfying when someone wins, so that the match has a proper conclusion.
  - [ ] First player to score 3 goals wins the match
  - [ ] On the winning goal, the goal celebration plays first (same 3-second auto or click-to-skip sequence)
  - [ ] After the celebration, a win screen appears
  - [ ] Win screen shows: winner's name, final score (e.g. "Rudy 3 — Carlos 1"), and a "Play Again" button
  - [ ] "Play Again" returns to the setup screen with names pre-filled and formations reset

---

### Epic: Bird Drop Event

- As a player, I want a random bird drop event to happen mid-game, so that the match has unexpected chaos that matches the street-game energy.
  - [ ] A bird randomly flies across the board as an overlay — no fixed timer, fully random
  - [ ] The bird drops a disgusting greenish-yellow dropping with maggots onto the board
  - [ ] The event plays as an overlay while the marble is still in motion — it does not pause or interrupt gameplay
  - [ ] If the marble rolls through the dropping, it is slowed (increased friction/drag) for the remainder of that turn
  - [ ] If the dropping lands directly on the marble, the marble is considered "dirty" — it moves slower and leaves a small visual trail on the next turn, then returns to normal
  - [ ] The dropping disappears from the board after 3 seconds regardless of marble interaction
  - [ ] The dirty marble effect clears after one full turn

---

## What We're Building

Everything required for a complete, playable match from open to close:

1. **Intro cinematic** — stadium zoom-in to backyard board, skippable by click
2. **Setup screen** — player names (persistent on replay), visual formation picker (side-by-side, per player), random board skin, Start Match button
3. **Match view** — full top-down board, score display (names + numbers), ambient scene, nail formations rendered for both players
4. **Slingshot mechanic** — click marble to aim, drag to set power/direction, aim line visible during drag, max drag distance enforced, release to shoot
5. **Physics** — marble bounces off nails and string borders, marble comes to rest to end the turn, nails are visual only (consistent bounce behavior)
6. **Turn system** — only active player can interact with marble, idle reminder glow at 5 seconds, auto-switch on marble rest
7. **Goal detection** — marble enters goal triggers celebration, score increments, marble resets to center, scoring team shoots first
8. **Goal celebration** — "GOAL!!" + scorer name + confetti + nail jump animation, 3 seconds auto or click-to-skip
9. **Win screen** — triggers at 3 goals, shows winner + final score + Play Again button
10. **Bird drop event** — random mid-game overlay, dropping slows marble on contact, dirty marble effect for one turn, disappears after 3 seconds

---

## What We'd Add With More Time

- **Sound design** — crowd reaction, marble bounce sounds, goal horn, bird squawk. Silence works for v1 but audio would elevate everything.
- **Animated nail reactions** — nails that get hit could wobble or shake on impact, not just on goal.
- **Third formation option** — a third nail layout as a strategic choice for future versions.
- **Additional board skins and scenes** — 2 of each is enough to prove the system; more variety is easy to add later.
- **Mobile support** — touch gestures for the slingshot mechanic once desktop is solid.
- **Wind nudge / falling fan nail events** — additional random events were cut for v1 but would add more chaos.

---

## Non-Goals

- **AI opponent** — v1 is strictly 2-player hot seat. An AI adds significant complexity and doesn't serve the core experience of two people playing together.
- **Mobile support** — desktop only. The slingshot mechanic is designed for mouse input. Touch gestures are a future problem.
- **Networking / multiplayer** — same screen, same device. No backend, no accounts, no syncing.
- **Persistence / save states** — no game history, no stats, no accounts. Session only. The game starts fresh every time.
- **Configurable win condition** — first to 3 goals is fixed. No settings screen, no match length options.

---

## Open Questions

- **Formation coordinate maps** — what are the exact nail positions for each formation on the board? This needs to be decided before /spec so the physics grid can be designed around it. *Answer before /spec.*
- **Board skin visuals** — what do "classic worn green felt" and "rustic wood" look like specifically? The aesthetic descriptions exist in the scope doc but the exact visual treatment (colors, texture approach) will need to be defined during spec. *Can wait until /spec or early /build.*
- **Max drag distance value** — what's the exact pixel or percentage cap? Close enough to "hard but can't fly out" for now; can be tuned during build. *Can wait until /build.*
- **Bird event frequency** — how often should the bird appear? Too frequent loses the surprise; too rare means players never see it. A reasonable starting point (e.g. once per 60-90 seconds with variance) can be tuned during build. *Can wait until /build.*
