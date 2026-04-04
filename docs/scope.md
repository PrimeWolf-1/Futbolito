# Futbolito

## Idea
A digital adaptation of a traditional Salvadoran street game — played on a handmade board with nails as players, a marble as the ball, and a popsicle stick to flick — built to feel like a physical object you could reach through the screen and touch.

## Who It's For
Two players sitting together (hot seat), one screen. Anyone who grew up playing informal street games and wants to feel that again — or anyone who wants to experience it for the first time. The game is for people who appreciate craft and authenticity over polish.

## Inspiration & References
- **Sensible Soccer (1992)** — top-down, whole pitch visible, evokes the game rather than simulates it. Closest in spirit. The view, the toy-like player representation, the drawing-on-paper quality. https://arcadespot.com/game/sensible-soccer/
- **HaxBall** — mechanically relevant (browser-based top-down soccer, physics-driven) but explicitly NOT the aesthetic model. Futbolito needs visible board edges, physical texture, the object itself front and center. https://www.haxball.com
- **The original game itself** — handmade board, nails hammered into wood, nylon string borders, a marble, a popsicle stick. That physical object is the design reference. The digital version should look like someone digitized the real thing, not designed a video game inspired by it.

**Design energy:** Raw, authentic, handmade-but-with-care. Worn wood. Chipped paint. Beat-up nails — some bent, some rusted, some split. The marble looks like real glass. The popsicle stick looks like real wood. Neighborhood, not corporate. El Salvador backyard, not app store. The board is the star — not a backdrop, the subject.

## Goals
- Build a complete, playable 2-player Futbolito game that captures the authentic feel of the physical street game
- Prove out a spec-driven development process (SDD) that Rudy can reuse on future projects (ORION, PrimeWolf, etc.)
- Produce a submission that demonstrates full planning-to-build workflow, not just a finished app

## What "Done" Looks Like
After 3-4 hours of building, the finished game:
- Opens to a start screen where two players enter their names and choose their nail formation (2 options)
- Loads a top-down board view — one of 2 board skin options (e.g. classic worn green felt, rustic wood), set inside one of 2 ambient scenes (e.g. backyard afternoon, street at night)
- Players take turns flicking the marble using a slingshot drag-and-release mechanic — click and drag back, release to shoot, power determined by drag distance
- The marble collides with nails (players), bounces off walls, and can enter the goal
- Live score displayed on screen; first to 3 goals wins
- Goals trigger a full celebration moment: "GOAL!!" with scorer's name, confetti, fan nails jumping wildly, winning player's nails reacting
- A random bird drop event can occur mid-game — the bird flies across and drops on the board (sometimes on a player's hand)
- On game end, a win screen celebrates the winner
- Built as a single HTML file (embedded CSS + JS), desktop-first

## What's Explicitly Cut
- **AI opponent** — v1 is 2-player hot seat only. AI adds complexity without serving the core experience.
- **Mobile support** — desktop first. Mobile gestures can come in a future version.
- **Third formation option** — 2 formation choices max. Decision happens pre-match; doesn't need to be exhaustive.
- **Additional board skins and scenes** — 2 of each is enough to show the system works. More can be added later.
- **Additional random events** — the bird drop stays (it's the personality of the game). Wind nudge and falling fan nail are cut for v1.
- **Multiplayer / networking** — hot seat only, same device, same screen.

## Loose Implementation Notes
- Single `.html` file, embedded CSS and JavaScript — Rudy's established build pattern
- Physics: needs marble-to-nail collision, marble-to-wall bounce, power-from-drag-distance. A lightweight 2D physics approach (Matter.js or hand-rolled) should work for this scale.
- Slingshot mechanic: click on marble → drag back → release. During drag, a visible string or dotted line renders from the marble showing direction and power — the player sees exactly where the marble will go before releasing.
- Nail formations: stored as coordinate maps, rendered as styled circles on the board canvas
- Board rendering: likely HTML5 Canvas for the game surface, DOM overlays for UI (score, names, turn indicator)
- Celebration moment: CSS animation + canvas burst for confetti, nail jump animation, bold text overlay. This is a priority — it has to feel alive.
- Turn logic: after marble comes to rest (velocity below threshold), switch active player. Goal → reset marble to center → scored-on team gets first touch.
- No backend, no accounts, no persistence needed — pure client-side
