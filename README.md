# Practical_Conjurer
This HTML,CSS and JavaScript code can changes graphical pictures appearance with your hands gesture with help of your camera. 

For HTML
Line 1        <!DOCTYPE html>          — tells browser this is HTML5
Line 2        <html lang="en">         — root element, language = English
Line 3–5      <head> block             — page metadata and title
Line 99       </head>
Line 100      <body>                   — visible page content starts
Line 101      <canvas id="c">          — the single canvas where ALL particles are drawn
Line 103–146  <div id="ui">            — the entire overlay UI (title, buttons, panels)
Line 104      #title div               — "✦ PARTICLE CONJURER ✦" text
Line 105      #tmpl-name               — current shape name e.g. "❤ HEART"
Line 106      #gest-label              — bottom status bar e.g. "EXPANDING"
Line 108–117  #templates               — the 8 emoji shape buttons 
Line 119–145  #right panel             — gesture guide + expansion meter + camera section
Line 122–125  .gr gesture rows         — the 4 gesture cards (it also move by keyboard key like O,C,N and P)
Line 130–134  #exp-meter               — the cyan→pink expansion progress bar
Line 137–141  #cam-preview             — camera video box + landmark overlay canvas
Line 138      <video id="vid">         — live webcam feed element
Line 139      <canvas id="lm-canvas">  — skeleton hand drawing overlaid on video
Line 142      <button id="cam-btn">    — "Enable Gesture Camera" button
Line 148      #mp-loading              — "Loading model…" spinner bar
Line 151      <script src="...">       — loads MediaPipe Hands ML library from CDN
Line 569–572  </script></body></html>  — closing tags

For CSS
Line 6        <style> opens
Line 7        @import Google Fonts      — loads Orbitron font from Google
Line 8        * { }                     — resets ALL elements: zero margin/padding
Line 9        body { }                  — black background, hide scrollbars
Line 10       #c { }                    — canvas fills entire screen (position:fixed)
Line 11       #ui { }                   — UI overlay sits on top, pointer-events:none so clicks pass through to canvas
Line 13–18    #title { }                — gradient rainbow text using -webkit-background-clip trick
Line 19–22    #tmpl-name { }            — shape name centered at bottom
Line 23–29    #gest-label { }           — pill-shaped status bar, glows cyan when active
Line 32–43    #templates + .tb { }      — left column buttons, scale on hover/active
Line 46–49    #right { }                — right panel flex column
Line 51–60    .gr + .gg + .ge etc       — gesture guide cards, cyan border when .lit class added
Line 62–67    #exp-meter { }            — expansion bar background and gradient fill
Line 69–89    #cam-section + #vid { }   — camera preview box, video mirrored with scaleX(-1)
Line 74        #lm-canvas { }           — landmark skeleton canvas overlaid on video, also mirrored
Line 75–76    #cam-dot { }              — red/green status dot in corner of camera
Line 78–89    #cam-btn { }              — camera button with cyan glow on hover
Line 91–97    #mp-loading { }           — fixed bottom loading bar
Line 98        </style> closes

For JavaScript
Line 153      <script> opens
Line 154      (function(){ — wraps everything in IIFE to avoid polluting global scope

━━━ CANVAS SETUP (160–174) ━━━
Line 160      Get canvas element + 2D drawing context
Line 162      resize() — sets canvas width/height to window size
Line 163      Listens for window resize to keep canvas full screen
Line 165      N = 3000 — total particle count
Line 166–169  Float32Arrays — typed arrays for x/y/z position, target, phase, color (faster than regular arrays)
Line 171–174  Initialize particles at random positions and random phase offsets

━━━ TEMPLATES (179–222) ━━━
Line 179      NAMES array — display names for each shape
Line 180–210  TMPLS array — 8 arrow functions, each takes (i, n) and returns [x, y, z]
              Line 181    Heart — parametric heart curve formula
              Line 182    Flower — polar rose r = 1.8 * |cos(5θ/2)|
              Line 183    Saturn — ring (circle) + random sphere points
              Line 184    Fireworks — 10 radial shell directions
              Line 185    Galaxy — 3-arm spiral with randomness
              Line 186    DNA — two helices offset by π
              Line 187    Sphere — Fibonacci lattice distribution
              Line 188    Tornado — expanding upward helix
Line 211–219  PALETTES — color arrays for each template (RGB 0–1 values)
Line 221      curTmpl — tracks which template is active
Line 222      expansion / targetExpansion — current and target scale
Line 223      colorShift — 0–1 value that slowly cycles colors

Line 225–229  buildTarget() — calls the template function, fills tx/ty/tz arrays, updates UI name
Line 230      switchTemplate() — wraps index, calls buildTarget, updates button highlights
Line 231      Event listeners on all 8 .tb buttons → call switchTemplate

Line 233–238  updateColors() — lerps between palette colors based on colorShift value

━━━ 3D + MOUSE CONTROLS (241–262) ━━━
Line 241–244  rotX/Y, drag, zoom — state variables for orbit control
Line 246      mousedown — start drag, record mouse position
Line 247      mousemove — if dragging, update target rotation angles
Line 248      mouseup/mouseleave — stop drag
Line 249      wheel — adjust zoom (camera Z position)
Line 250–252  Touch equivalents for mobile

Line 254–262  project(x,y,z) — the 3D math:
              Step 1: rotate around Y axis (left/right spin)
              Step 2: rotate around X axis (up/down tilt)
              Step 3: perspective divide — far objects get smaller scale
              Returns: [screenX, screenY, depth, scale]

━━━ RENDER LOOP (265–301) ━━━
Line 265      (function loop(){ requestAnimationFrame(loop) — runs 60 times/second
Line 267      Smooth rotX/Y toward target (easing)
Line 268      Auto-spin slowly when not dragging
Line 270      Increment colorShift — slow rainbow cycle
Line 273      Smooth expansion toward targetExpansion at 10% per frame
Line 275–280  Per-particle update:
              - Calculate wave offset using sin + phase
              - Compute target = template position × expansion + wave
              - Lerp current position toward target at 5% per frame
              - Update size with pulse animation
Line 282      Clear canvas with semi-transparent black — creates motion trails
Line 284      Sort all 3000 particles by Z depth (back to front)
Line 286–295  Draw each particle as a radial gradient glow circle
Line 298–300  Update expansion meter bar and value text in UI

━━━ GESTURE ENGINE (303–568) ━━━
Line 303–316  Get all the DOM elements needed for camera UI

Line 318–325  isFingerExtended(lms, tipIdx, mcpIdx)
              — returns true if finger tip Y < base knuckle Y by 2% margin
              — Y=0 is top of frame so smaller Y = higher up = extended

Line 327–329  dist2D(a, b) — Euclidean distance between two landmarks

Line 331–360  classifyGesture(lms) — the core gesture logic:
Line 333–336  Check index/middle/ring/pinky extended vs MCP base knuckles (5,9,13,17)
Line 337      thumbDist — distance from thumb tip (4) to wrist (0)
Line 338      extCount — how many of 4 fingers are extended
Line 341      Pinch check — thumb tip (4) to index tip (8) distance < 0.08
Line 344      Open hand — 3+ fingers extended
Line 347      Point — only index extended, others all down
Line 350      Fist — zero fingers extended
Line 353      Otherwise idle

Line 362–372  smoothGesture() — keeps last 6 gesture readings, returns majority vote winner

Line 374–405  applyGesture(g) — the effects:
Line 376–378  Highlights matching gesture guide card with .lit class
Line 380–388  Updates status label text and color
Line 390–391  'open' → targetExpansion += 0.12 (fast ramp up)
Line 392–393  'fist' → targetExpansion -= 0.12 (fast ramp down)
Line 394–398  'point' → switchTemplate if 1s cooldown passed
Line 399–400  'pinch' → colorShift += 0.015 (spin colors fast)
Line 401–404  idle → gently drift expansion back to 1.0

Line 407–436  drawLandmarks(lms) — draws the green skeleton on lm-canvas:
Line 413–419  21 bone connection pairs
Line 421–426  Draws lines between connected landmarks
Line 429–435  Draws dots at each landmark (thumb+index tips are pink)

Line 438–546  camBtn click handler — the full camera startup sequence:
Line 444–447  Check getUserMedia API exists
Line 450–462  Request camera stream — catch specific error types
Line 464–467  Attach stream to video element, wait for metadata, play
Line 469–473  Show camera preview, resize landmark canvas
Line 479–485  Check if MediaPipe Hands loaded from CDN
Line 488–495  new Hands() — initialize model with options
Line 497–515  hands.onResults() callback — called every frame with detected landmarks:
              - If hand found: draw landmarks, classify, smooth, apply
              - If no hand: clear canvas, drift expansion back to 1
Line 518–524  sendFrame() — the manual frame loop:
              - Checks video is ready (readyState >= 2)
              - Calls hands.send({image: vid}) to process current frame
              - Loops via requestAnimationFrame
Line 526–535  hands.initialize() — loads WASM model, then starts sendFrame()
Line 537–539  3-second check — warns if no results received yet

Line 542–563  setupKeyboardFallback() — O/C/N/P key bindings
Line 566      setupKeyboardFallback() — called immediately so keys always work

Line 568      })() — closes IIFE
Line 569      </script> closes
