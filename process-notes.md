# Process Notes

## /scope

**How the idea evolved:** Rudy arrived with the idea fully formed — board, nails, marble, popsicle stick, turn-based, scenes, formations, random events. The conversation sharpened it rather than invented it. Key evolution: cut AI opponent, cut mobile, cut extra random events, distilled to 2 formations and 2 board skins. The celebration moment (goal scoring) emerged as a late-conversation priority that wasn't explicitly in the first brain dump but turned out to be "everything."

**Pushback received:** Research examples were used productively. Rudy rejected HaxBall as aesthetic model (no visible board), rejected Soccer Physics as too chaotic (Futbolito is chess-like and deliberate), and validated Sensible Soccer as closest in spirit. The core distinguishing insight he articulated: "I don't want it to feel like a video game — I want it to feel like a physical object that's been digitized."

**What resonated:** The Sensible Soccer framing — evokes rather than simulates. The "board is the star" framing. The celebration moment question landed as the sharpest question of the session ("great question you see").

**Deepening rounds:** Rudy chose zero formal deepening rounds. Opted for one targeted question instead. The single question (what happens when someone wins?) surfaced a high-priority feature that wasn't fully articulated in the brain dump. That was a material improvement to the scope doc.

**Active shaping:** Rudy drove the direction completely. Every cut decision was his own. He pushed back specifically on each research reference. He defined "what's not in scope" before being asked. He self-organized the priority stack unprompted. Passive acceptance: zero.

## /prd

**What the learner added vs scope doc:** The intro cinematic was new — scope doc said "opens to a start screen" but Rudy expanded it into a full mood-setting sequence with a stadium zoom-in that narratively contrasts big-league football with street game. The dirty marble mechanic (bird dropping landing on the marble = drag trail for one full turn) was invented mid-conversation and not in scope at all. The idle turn reminder (5-second glow pulse instead of always-on indicator) was a significant refinement — he replaced a static UI element with a contextual ambient cue.

**"What if" moments that landed:** The inactive player clicking the marble — Rudy hadn't thought about it and immediately knew the answer (game enforces it, not the players). The dirty marble question (what if the dropping lands on the marble itself) surprised him and he invented the one-turn drag trail on the spot. Both were genuine "oh, I hadn't thought of that" moments.

**What he pushed back on or felt strongly about:** Zero pushback — he answered every question with conviction and no hesitation. The zoom-in intro framing ("from the big leagues down to the street game — that contrast is the whole story of futbolito") was delivered with the same energy as the "board is the star" moment in /scope. He knows exactly what this game is.

**Scope guard:** No scope creep occurred. Rudy kept every answer tight. The bird drop mechanic expanded slightly (dirty marble effect) but stayed well within 3-4 hours — it's a physics modifier, not a new feature.

**Deepening rounds:** One round of 5 questions. Surfaced: the zoom-in intro transition narrative, nail visual vs. physics separation (pure aesthetic), side-by-side formation preview layout mirroring board position, final score on win screen, and turn enforcement (game-enforced not social). No new edge cases that required scope re-evaluation, but the details meaningfully sharpened acceptance criteria throughout.

**Active shaping:** Rudy drove every answer. The dirty marble was his invention. The idle glow instead of always-on indicator was his refinement. The formation preview side-by-side mirroring the board layout was his logic. No passive acceptance — he thought through each question before answering.

## /spec

**Technical decisions made:**
- Stack locked immediately — single HTML file with embedded CSS/JS was already Rudy's established pattern, no debate needed.
- Physics library: Matter.js 0.20.0 chosen over hand-rolled physics. Rudy explicitly wanted to focus on game feel, not collision math. Planck.js was surfaced as an alternative but rejected — more complex setup, fewer simple examples.
- Custom renderer over Matter.js built-in — Rudy immediately understood the separation (Matter.js does math, Canvas 2D controls look) and validated it without hesitation.
- Deployment: Netlify drag-and-drop. Rudy needs a shareable URL for Devpost submission.
- Normalized {x,y} formation coordinates: Rudy confirmed the scaling approach made sense instantly ("define once, draw anywhere").
- Manual Matter.Engine.update() over Matter.Runner — keeps physics + render in one loop we control.

**What Rudy was confident about vs. uncertain:**
- Confident: everything. Every architectural proposal was validated quickly with no hesitation. He recognized the patterns from his own build experience.
- Uncertain: nothing surfaced as uncertain. He entered /spec with a clear mental model of what he was building.

**Deepening rounds:** Zero rounds chosen. Rudy said "ready to generate the spec" immediately after the epic walkthrough. All architectural components were covered in the mandatory phase — the epic-by-epic walkthrough was thorough enough that deepening rounds would have been redundant.

**Active shaping:** Rudy pushed back on hand-rolling physics immediately and with clear reasoning ("I want to focus on the game feeling right, not debugging collision math"). The bird drop aesthetic decision (drawing is the star, not the bird) was his framing — shaped the Bird Drop module's priority. No passive acceptance.

## /onboard

**Technical experience:** Intermediate beginner. Builds single-file HTML apps with embedded CSS/JS. Uses Claude as primary dev partner. Has integrated Groq, Twelve Data, and TradingView APIs. Doesn't write code from scratch — guides the AI and catches problems.

**Learning goals:** Wants the process, not just the project. Specifically: scope docs, PRDs, technical specs, and multi-session context management. Plans to apply SDD to ORION and PrimeWolf agency projects after this.

**Creative sensibility:** Old school, raw, authentic, handmade. For Futbolito — wants it to feel like the original street game from El Salvador. Grounded neighborhood feel, not corporate polish. Appreciates craft when it's earned.

**Prior SDD experience:** Has seen unplanned (ORION, v15, messy) vs. lightly planned (MIA, smoother). Never used formal artifacts. Understands why planning matters from lived experience. New to the vocabulary and document types.

**Energy and engagement:** High intent, clear goals, self-aware about gaps. Came in with a specific project idea and a specific pain point. Not here to explore — here to learn a system. Move efficiently.

## /checklist

**Sequencing decisions:** Scaffold first (everything depends on it), physics + board render second (riskiest piece — library integration — build early to leave time to pivot), setup screen third (formations and UI before gameplay), slingshot + turn system fourth (core gameplay loop). Goal detection, celebration, and win screen come next as a natural escalating chain. Intro cinematic placed late (mood-setting but not load-bearing — a buggy intro should never block gameplay testing). Bird drop last before submission (completely optional overlay, no dependencies). Devpost always last.

**Methodology preferences:** Autonomous mode — Rudy wants to move fast. Verification yes — checkpoints after items 4, 7, and 9. No comprehension checks (N/A for autonomous). Git: commit after each item; GitHub repo to be created in item 1 (not yet set up). Netlify deploy included as part of final submission item — Rudy confirmed he wants a live link for judges.

**Item count and estimated time:** 10 items. Estimated 3-4 hours at 15-30 minutes per item. Checkpoints at: item 4 (core gameplay loop working), item 7 (full match lifecycle working), item 9 (feature-complete). 

**What Rudy was confident about vs. needed guidance on:** Rudy answered every question with one word ("build", "autonomous", "yes"). Zero hesitation across the entire session. Did not engage with the sequencing question — deferred completely to the agent. Accepted the proposed 10-item order without pushback or questions.

**Deepening rounds:** Zero rounds chosen. Moved through every question at maximum speed. No refinement of item granularity, no dependency questioning, no sharpening of verification steps.

**Active shaping:** Minimal — Rudy drove direction in /scope, /prd, and /spec but in /checklist deferred entirely to the agent. The sequencing logic, item breakdown, and checkpoint placement were all agent-generated and accepted without modification. The one material contribution: confirmed he wants Netlify deploy + live link for Devpost (influenced final item scope).

**Submission planning:** Core story is clear — digital adaptation of a real Salvadoran street game, feels like a physical object not a video game. "Wow moment" is the board with full visual personality + goal celebration. GitHub repo does not yet exist (to be created in item 1). Netlify deploy confirmed. No demo video planned.

## /build

**Total items completed:** 10/10. All checklist items finished. Build is complete.

**Build mode:** Autonomous. 10 items dispatched to subagents sequentially, each receiving full spec context. Checkpoints at items 4, 7, and 9 — Rudy verified with "Y" at each.

**Checklist revisions during build:** None to the structure. One mid-build interruption after checkpoint 1: Rudy requested a visual overhaul before continuing past item 4. The interruption happened again after items 5-7 completed (the agent had already built them before seeing the message). Both visual overhaul passes were handled as additional subagent dispatches outside the checklist items, not as checklist revisions.

**Visual overhaul passes (2 total):**
- Pass 1 (after checkpoint 1, before item 5): Real nail rendering with radial gradient + cross-slot, team colors, field lines, nylon string border with 3-layer shadow/body/highlight, marble power boost (MAX_SPEED 18→30), felt grain texture via offscreen canvas.
- Pass 2 (mid-build, after items 5-7): Full deep overhaul — cylindrical nail shafts with drop shadows, 5-layer glass marble, walnut wood frame (22px border) with grain lines, goal net structures with post caps and net grid, outdoor backyard gradient background, string border and field lines adjusted to new BORDER=22.

**Checkpoint observations:**
- Checkpoint 1 (after item 4): Rudy approved with "Y" but immediately requested visual overhaul. Core gameplay was working but board looked too flat/digital.
- Checkpoint 2 (after item 7 — visual overhaul pass 2 done): Rudy approved with "Y." Full match lifecycle working.
- Checkpoint 3 (after item 9): Rudy approved with "Y." Feature-complete.

**Deployment:** GitHub repo created at github.com/PrimeWolf-1/Futbolito. Netlify drag-and-drop deploy confirmed live. Devpost submission completed.

**Overall:** Build went smoothly. No broken items, no checklist rework needed. The main friction point was visual fidelity — the "physical object you can reach through the screen" bar required two overhaul passes to meet. Rudy was clear about what he wanted and called it out at the right moment both times. The codebase is a single `futbolito.html` file with embedded CSS and JS, ~1500+ lines, fully functional.
