# DSA Coach Mini — Master Build Prompt
> Feed this entire file to Anti Gravity. Replace `YOUR_SARVAM_API_KEY` with the real key before pasting.

---

## Your Task

Build a **single self-contained HTML file** called `dsa_coach_mini.html`. No backend, no npm, no build step. Everything — HTML, CSS, JS — lives in one file. It must work by opening the file directly in a browser (or serving it locally).

---

## API Configuration

```javascript
const SARVAM_API_KEY = "YOUR_SARVAM_API_KEY";
const SARVAM_CHAT_URL = "https://api.sarvam.ai/v1/chat/completions";
const SARVAM_TTS_URL  = "https://api.sarvam.ai/text-to-speech";
const SARVAM_STT_URL  = "https://api.sarvam.ai/speech-to-text";
const SARVAM_MODEL    = "sarvam-m";
const SARVAM_TTS_MODEL = "bulbul:v3";
```

All API calls include the header: `api-subscription-key: ${SARVAM_API_KEY}`

---

## Data (Hardcode These — No External Fetch)

```javascript
const PROBLEMS = [
  {
    id: "two-sum",
    title: "Two Sum",
    topic: "Array",
    difficulty: "Easy",
    description: "Given an array of integers and a target, return the indices of the two numbers that add up to the target. Assume exactly one solution exists.",
    examples: [
      { input: "nums = [2,7,11,15], target = 9", output: "[0,1]" },
      { input: "nums = [3,2,4], target = 6",      output: "[1,2]" }
    ]
  },
  {
    id: "reverse-array",
    title: "Reverse an Array",
    topic: "Array",
    difficulty: "Easy",
    description: "Given an array, return it reversed in-place.",
    examples: [
      { input: "[1,2,3,4,5]", output: "[5,4,3,2,1]" },
      { input: "[1]",          output: "[1]" }
    ]
  },
  {
    id: "reverse-linked-list",
    title: "Reverse a Linked List",
    topic: "Linked List",
    difficulty: "Medium",
    description: "Given the head of a singly linked list, reverse it and return the new head.",
    examples: [
      { input: "1→2→3→4→5", output: "5→4→3→2→1" },
      { input: "1→2",         output: "2→1" }
    ]
  }
];

const LANGUAGES = [
  { code: "en-IN", label: "English", short: "EN" },
  { code: "hi-IN", label: "हिंदी",   short: "HI" },
  { code: "te-IN", label: "తెలుగు",  short: "TE" }
];

const TIMER_SECONDS = 30;
```

---

## App State

```javascript
let state = {
  screen: "select",         // "select" | "thinking" | "solving"
  problem: null,            // selected PROBLEMS entry
  language: LANGUAGES[0],   // active language object
  pseudocode: "",           // textarea content
  timerRemaining: TIMER_SECONDS,
  timerRunning: false,
  timerDone: false,
  hints: [],                // array of { level, text, audio } — loaded on demand
  hintsVisible: [false, false, false],
  codeInput: "",
  reviewResult: null,       // { assessment, edgeCases, complexity, tips: [tip1, tip2] | [] }
  reviewAudio: null,        // base64 audio for full review
  tip1Audio: null,
  tip2Audio: null,
  isRecording: false,
  mediaRecorder: null,
  audioChunks: []
};
```

---

## Screen 1 — Problem Select

- Header: "DSA Coach" in white, language toggle (EN | HI | TE pills) top-right
- 3 problem cards in a column (or grid), each showing:
  - Problem title (large)
  - Topic badge (gray pill)
  - Difficulty badge (green = Easy, yellow = Medium)
  - Short description excerpt
- Click card → set `state.problem`, switch to screen "thinking"
- Black background, cards with subtle dark border

---

## Screen 2 — Thinking Mode

**Layout (top to bottom):**

### A. Problem Bar (always visible, top)
- Title · Topic badge · Difficulty badge
- "← Back" link (returns to select, resets state)

### B. Problem Detail
- Full description text
- Example inputs/outputs in a monospaced block

### C. Pseudocode / Notes Area

**Two modes toggled by two buttons: [ ✏️ Type ] [ 🎙️ Speak ]**

**Type mode (default):**
- `<textarea>` placeholder: "Write your approach, pseudocode, or notes here..."
- No enforcement — freeform

**Speak mode:**
- Large 🎙️ button — hold or click to start/stop recording
- While recording: button pulses red, label = "Recording... click to stop"
- On stop:
  1. Send audio blob to Sarvam STT with `language_code: state.language.code`
  2. Get transcript
  3. Send transcript to sarvam-m with this system prompt:

```
You are a pseudocode assistant. The student has verbally explained their approach.
Convert their explanation into a clean numbered pseudocode outline.
Rules:
- High-level steps only, no runnable code
- Maximum 8 steps
- Language of response: {state.language.code}
- Output ONLY the numbered pseudocode, nothing else
```
  4. Write the structured pseudocode into the textarea
  5. Student may continue editing

### D. Timer Section

- Large label: "Thinking Timer"
- Subtext: "Coach and Code unlock when the timer ends"
- Big countdown display: `00:30` → counts down
- [ Start Timer ] button — blue, prominent
- On start: timer runs, button hidden
- At 0: `state.timerDone = true`, switch to screen "solving"
- Timer cannot be paused or reset

---

## Screen 3 — Solving Mode

**Layout:**

### Top: Problem Bar (same as thinking mode, always visible)

### Middle: Pseudocode Review (collapsed by default)
- "Your Notes" accordion — shows the pseudocode they wrote, read-only

### Bottom: Two side-by-side panels

---

### LEFT PANEL — Coach 🤖

**Unlocked (timerDone = true):**

Header: "Coach" + language toggle (same pill toggle, synced with top bar)

**3 hint cards stacked vertically:**

Each card:
```
[ Level 1 — Concept        ▾ ]  [ 🔊 Listen ]
[ Level 2 — Direction      ▾ ]  [ 🔊 Listen ]
[ Level 3 — Pseudocode     ▾ ]  [ 🔊 Listen ]
```

- All 3 available immediately after unlock
- Click card header → expand and **fetch hint** (if not already fetched)
- Once fetched, text shows inside the card
- 🔊 Listen → call TTS on that hint text, play audio
- Loading spinner while fetching

**Hint prompts (send to sarvam-m):**

System prompt for all hints:
```
You are a DSA coach. Never give the full solution or working code.
Respond ONLY in the language matching locale: {state.language.code}
Problem: {problem.title}
Description: {problem.description}
```

Level 1 user message:
```
Give a Level 1 hint. Ask ONE guiding question that helps the student think about the problem. Do not name any technique or data structure.
```

Level 2 user message:
```
Give a Level 2 hint. Point the student toward the right technique or data structure without explaining how to implement it. 2-3 sentences max.
```

Level 3 user message:
```
Give a Level 3 hint. Write a numbered pseudocode outline (5-7 steps max). High-level only — no runnable code, no syntax.
```

---

### RIGHT PANEL — Code 💻

**Unlocked (timerDone = true):**

Header: "Code"

- `<textarea>` for code input — monospaced font, dark background, line-height generous
- Placeholder: "Write or paste your solution here..."
- [ Check My Approach ] button — blue, full width at bottom of panel
- While loading: button shows spinner + "Reviewing..."

**On click — send to sarvam-m:**

System prompt:
```
You are a DSA code reviewer. Do NOT provide a corrected or full solution.
Respond ONLY in the language matching locale: {state.language.code}
Problem: {problem.title}
Description: {problem.description}
```

User message:
```
Review this student's code:

{state.codeInput}

Respond in this exact JSON structure:
{
  "assessment": "Is the logic correct? Brief explanation.",
  "edgeCases": "What edge cases might be missed?",
  "complexity": "Likely time complexity and why.",
  "isOptimal": true or false,
  "tip1": "First improvement tip (only if isOptimal is false, else empty string)",
  "tip2": "Second improvement tip (only if isOptimal is false, else empty string)"
}
Output ONLY valid JSON, no markdown fences.
```

**Display the review result below the button:**

```
────────────────────────────────
✅ Logic Assessment
{assessment text}

⚠️ Edge Cases
{edgeCases text}

⏱ Complexity
{complexity text}

[ 🔊 Listen to Full Review ]
────────────────────────────────
```

If `isOptimal === false`, also show:
```
────────────────────────────────
💡 Tip 1 — Make It Better
{tip1 text}
[ 🔊 Listen to Tip 1 ]

💡 Tip 2 — Make It Better
{tip2 text}
[ 🔊 Listen to Tip 2 ]
────────────────────────────────
```

**TTS for review:**
- "Listen to Full Review" → concatenate assessment + edgeCases + complexity → TTS → play
- "Listen to Tip 1" → TTS tip1 text → play
- "Listen to Tip 2" → TTS tip2 text → play
- All use `state.language.code` as `target_language_code`

---

## TTS Helper Function

```javascript
async function speak(text) {
  const response = await fetch(SARVAM_TTS_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "api-subscription-key": SARVAM_API_KEY
    },
    body: JSON.stringify({
      inputs: [text],
      target_language_code: state.language.code,
      speaker: "meera",  // use "meera" for EN, "arvind" for HI, "default" for TE — or just "meera" for all
      model: SARVAM_TTS_MODEL,
      enable_preprocessing: true
    })
  });
  const data = await response.json();
  // data.audios[0] is base64 WAV
  const audio = new Audio("data:audio/wav;base64," + data.audios[0]);
  audio.play();
}
```

---

## STT Helper Function

```javascript
async function transcribeAudio(audioBlob) {
  const formData = new FormData();
  formData.append("file", audioBlob, "recording.wav");
  formData.append("language_code", state.language.code);
  formData.append("model", "saarika:v2");  // Sarvam STT model

  const response = await fetch(SARVAM_STT_URL, {
    method: "POST",
    headers: { "api-subscription-key": SARVAM_API_KEY },
    body: formData
  });
  const data = await response.json();
  return data.transcript;
}
```

---

## Chat Helper Function

```javascript
async function chat(systemPrompt, userMessage) {
  const response = await fetch(SARVAM_CHAT_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "api-subscription-key": SARVAM_API_KEY
    },
    body: JSON.stringify({
      model: SARVAM_MODEL,
      messages: [
        { role: "system", content: systemPrompt },
        { role: "user",   content: userMessage }
      ],
      max_tokens: 600
    })
  });
  const data = await response.json();
  return data.choices[0].message.content;
}
```

---

## Design System

```css
:root {
  --bg:        #0a0a0a;
  --surface:   #141414;
  --surface2:  #1e1e1e;
  --border:    #2a2a2a;
  --text:      #f5f5f5;
  --muted:     #6b7280;
  --blue:      #2563eb;
  --blue-hover:#1d4ed8;
  --green:     #16a34a;
  --yellow:    #ca8a04;
  --red:       #dc2626;
  --locked-bg: #111111;
  --locked-text: #374151;
  --radius:    8px;
  --font:      system-ui, -apple-system, sans-serif;
  --mono:      'Courier New', monospace;
}

body { background: var(--bg); color: var(--text); font-family: var(--font); margin: 0; }

/* Locked panel overlay */
.panel-locked {
  pointer-events: none;
  opacity: 0.35;
  filter: grayscale(1);
  position: relative;
}
.panel-locked::after {
  content: "🔒";
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  font-size: 3rem;
  opacity: 1;
  filter: none;
}

/* Timer countdown */
.timer-display {
  font-size: 5rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: var(--blue);
  text-align: center;
  letter-spacing: 0.05em;
}

/* Language pills */
.lang-toggle { display: flex; gap: 4px; }
.lang-pill {
  padding: 4px 12px;
  border-radius: 20px;
  border: 1px solid var(--border);
  cursor: pointer;
  font-size: 0.8rem;
  background: var(--surface2);
  color: var(--muted);
}
.lang-pill.active {
  background: var(--blue);
  color: white;
  border-color: var(--blue);
}

/* Difficulty badges */
.badge-easy   { background: #14532d; color: #4ade80; }
.badge-medium { background: #713f12; color: #fbbf24; }
.badge { padding: 2px 10px; border-radius: 12px; font-size: 0.75rem; font-weight: 600; }

/* Recording pulse */
@keyframes pulse-red {
  0%, 100% { box-shadow: 0 0 0 0 rgba(220,38,38,0.7); }
  50%       { box-shadow: 0 0 0 10px rgba(220,38,38,0); }
}
.recording { animation: pulse-red 1s infinite; background: var(--red) !important; }

/* Blue button */
.btn-blue {
  background: var(--blue); color: white;
  border: none; border-radius: var(--radius);
  padding: 10px 20px; cursor: pointer; font-size: 1rem;
  transition: background 0.15s;
}
.btn-blue:hover { background: var(--blue-hover); }
.btn-blue:disabled { opacity: 0.5; cursor: not-allowed; }
```

---

## Critical Rules

1. **Timer is the only unlock.** Do not add any other unlock mechanism.
2. **Coach never produces runnable code** at any hint level.
3. **Language toggle is global** — changing it re-renders all hint text labels but does not auto-re-fetch already-loaded hints. A note ("Switch language to re-load hints in a new language") is acceptable.
4. **Voice pseudocode writes into the textarea** — student can edit afterward.
5. **Code review JSON must be parsed safely** — wrap in try/catch, show a friendly error if parsing fails.
6. **Tips only appear if `isOptimal === false`** — do not show them if the solution is already good.
7. **All loading states must be visible** — spinner or disabled button while any API call is in flight.
8. **No external CDN, no imports** — everything inline. Exception: you may use a Google Font link tag for a clean sans-serif if desired.
9. **File must open correctly from `file://` or `localhost`** — no module imports that break without a server.
10. **Mobile-friendly is nice but not required** — optimize for a laptop/projector view first.

---

## Deliverable

One file: `dsa_coach_mini.html`

Test checklist before handing off:
- [ ] Problem select renders 3 cards
- [ ] Language toggle switches EN/HI/TE and persists
- [ ] Thinking mode shows textarea + Speak button
- [ ] Voice record → STT → pseudocode generation → textarea fill works
- [ ] Timer starts, counts down, reaches 0
- [ ] At 0: both panels unlock visually and functionally
- [ ] All 3 hints load on expand with correct language
- [ ] 🔊 Listen plays audio for each hint
- [ ] Code review fires, displays assessment/edge cases/complexity
- [ ] 🔊 Listen plays full review audio
- [ ] Tips appear (with audio) only when solution is not optimal
- [ ] No console errors on happy path
