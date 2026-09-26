# DSA Coach Mini — Demo Build Overview
> Sarvam Demo · Single HTML · No backend · 30-minute build target

---

## What This Is

A self-contained, single-file DSA coaching tool built for a timed demo. It enforces the **"think before you code"** loop — students must sit through a timer before the Coach or Code panels unlock. The centerpiece is the same hint spoken naturally in **English, Hindi, and Telugu** via Sarvam TTS.

---

## Scope

### Problems (hardcoded, no search)
| Problem | Topic | Difficulty |
|---|---|---|
| Two Sum | Array | Easy |
| Reverse an Array | Array | Easy |
| Reverse a Linked List | Linked List | Medium |

### Languages
| Language | Locale | Used For |
|---|---|---|
| English | en-IN | Coach text + TTS |
| Hindi | hi-IN | Coach text + TTS |
| Telugu | te-IN | Coach text + TTS |

A single toggle in the top bar controls language for the **entire session** — Coach writes in it, TTS speaks in it.

---

## Core Flow

```
Select Problem
      ↓
  Thinking Mode
  ┌──────────────────────────────────┐
  │  Pseudocode / Notes Area         │
  │  • Type freely  OR               │
  │  • 🎙️ Speak (EN / HI / TE)      │
  │    → Sarvam STT transcribes      │
  │    → LLM structures into         │
  │      basic pseudocode steps      │
  └──────────────────────────────────┘
      ↓
  Start 30s Timer  ← fixed, demo constraint
      ↓
┌─────────────┴─────────────┐
↓                           ↓
COACH (🔒 until timer)     CODE (🔒 until timer)
  ↓                           ↓
Progressive Hints (3 lvls)  Write / paste code
  ↓                           ↓
🔊 Listen (EN/HI/TE)        "Check My Approach"
                               ↓
                          AI Code Review
                          (qualitative, no exec)
                               ↓
                          🔊 Listen to Review
                          + 2 Improvement Tips
                            (if not optimal)
```

**Lock rule:** Timer expiry is the **only** unlock condition. Before it, both panels show a visible 🔒 indicator legible from across the room.

---

## Feature Details

### 1 — Pseudocode Area (Thinking Mode)

Two input modes, student's choice:

**Type mode** (default)
- Plain textarea, no Monaco editor (demo simplification)
- Freeform — no structure enforced

**Voice mode** (new addition)
- 🎙️ mic button beside the textarea
- Records speech in whichever language is selected (EN / HI / TE)
- Sends audio to **Sarvam STT** for transcription
- Transcription is passed to **sarvam-m** with the prompt:
  > "Convert this spoken explanation into a numbered pseudocode outline. Keep it high-level, no runnable code."
- Result is written back into the pseudocode textarea — student can edit before timer starts
- Voice and type can be mixed: speak first, then refine by typing

---

### 2 — Hint System (Coach Panel, post-timer)

3 levels, all available at once after timer ends:

| Level | Name | What It Contains |
|---|---|---|
| 1 | Concept | A guiding question only — no technique named |
| 2 | Direction | Points at the right technique / data structure |
| 3 | Pseudocode | Numbered steps, still no runnable code |

Each hint has a **🔊 Listen** button → Sarvam TTS speaks it in the active language.

**Rule:** Coach never produces a full working solution at any level, in any language.

---

### 3 — Code Review ("Check My Approach", post-timer)

Student pastes / writes code in the Code panel, then hits **Check My Approach**.

The Coach returns (in the active language):
1. **Logic assessment** — is the approach correct?
2. **Edge cases** — what might be missed?
3. **Time complexity** — likely Big-O
4. **🔊 Listen** button — entire review spoken via Sarvam TTS

**If the solution is not optimal**, two additional items follow:
5. **Tip 1** — first concrete improvement direction
6. **Tip 2** — second concrete improvement direction

Both tips also get their own **🔊 Listen** buttons.

The Coach does **not** run code, return Accepted/Wrong Answer, or produce a corrected solution. This is a review, not an executor.

---

## Design Language

- **Palette:** Black background · White text · Blue (`#2563EB`) for all interactive / unlocked states · Muted gray for locked states
- **Locked panels:** Visually desaturated, 🔒 icon, no pointer events — unambiguous from across the room
- **Problem bar:** Always visible at top — title, topic badge, difficulty badge
- **Language toggle:** Top-right, always accessible
- **Timer:** Large countdown, centered, prominent during thinking phase
- **Layout:** Problem info (top) → Thinking/Pseudocode (middle) → Coach + Code side-by-side (bottom, unlocked post-timer)

---

## Tech Stack

```
Single self-contained HTML file
Vanilla JS — no build step, no framework
Sarvam Chat Completions (sarvam-m) — hints, pseudocode structuring, code review
Sarvam Text-to-Speech (bulbul:v3) — hint/review audio in en-IN / hi-IN / te-IN
Sarvam Speech-to-Text — voice pseudocode input in EN / HI / TE
API key injected at top of file as a JS const — no backend needed
State in memory only — resets on page refresh
```

---

## APIs Used

| Purpose | Sarvam Endpoint |
|---|---|
| Hints + Code Review + Pseudocode structuring | Chat Completions — `sarvam-m` |
| TTS (hint audio, review audio, tips audio) | `bulbul:v3` |
| STT (voice pseudocode input) | Sarvam STT endpoint |

---

## What Was Cut (vs Full Spec)

```
✗ Topics beyond Array / Linked List
✗ Problem search, filters, library
✗ 7-min / custom timer — fixed 30s instead
✗ Languages beyond EN / HI / TE
✗ Hint levels 4–5 (Pseudocode Assistance, Code Assistance)
✗ Sandboxed code execution (Judge0/Piston), Run/Submit states
✗ Monaco editor — plain textarea
✗ Auth, database, persistence
✗ Submission history, dashboard, progress tracking
✗ Multi-programming-language selection
```

---

## Demo Script Notes

- State the 30s timer is a demo constraint — real product minimum is 7 minutes
- The language toggle moment (same hint, three languages, audio) is the headline demo beat
- Voice pseudocode → structured steps shows AI-assist without removing student thinking
- Code Review + Tips shows the coaching depth without needing a judge
