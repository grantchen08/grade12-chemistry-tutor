# Alkane Naming Quiz — Engagement & UX Enhancements (Phased Design Doc)

Author: Grant Chen  
Date: 2025-09-08  

## Executive Summary

To improve learner engagement and instructional efficacy, we will add progressive, low-friction incentives and guidance during each level. This includes:

- Clear progress indication and goals
- Positive reinforcement feedback and micro-celebrations
- Hints/assists after mistakes
- Visible star/time targets and (optional) timer
- Optional lightweight audio/visual polish
- Telemetry to assess effectiveness and guide iteration

Changes will be delivered in well-bounded phases, each shippable and reversible.

---

## Goals

- Increase completion rate and session length by providing moment-to-moment feedback.
- Reduce frustration through contextual hints and recovery tools (e.g., 50-50).
- Make target outcomes (stars/time) visible to guide effort.
- Preserve performance and accessibility across devices.

## Non-Goals

- Full-fledged learning-path management or adaptive difficulty (out-of-scope).
- Backend services (all data remains local).
- Complex molecule analytics (hints remain conceptual unless data is provided).

---

## Current State (Baseline)

- Two-screen app: Level Select and Game.
- ChemDoodle renders molecules from MOL strings.
- LocalStorage profile stores stars/XP/daily streaks.
- Known cleanups:
  - Removed “Continue” button on Level Select.
  - Removed URL deep linking (?level=N).
  - Fixed CSS visibility conflict with `.wrap` + `.hidden`.

---

## Phase Overview

- Phase 1: Foundation and UX cleanups (done/in-flight).
- Phase 2: Progress Indicator (counter + bar).
- Phase 3: Positive reinforcement and streak celebrations.
- Phase 4: Hints/Assists (Show Hint, 50-50).
- Phase 5: Live Targets + Timer feedback.
- Phase 6: Optional SFX and motion polish (respect user preferences).
- Phase 7: Analytics (local-only event logging).
- Phase 8: A/B toggles and configuration hardening.
- Phase 9: Future work (per-question explanations, adaptive hints).

---

## Phase 1 — Foundation & UX Cleanups

Scope:
- Ensure `.hidden` always hides on desktop grid layouts.
- Remove “Continue” button from Level Select.
- Remove deep linking; never alter URL during play.

Acceptance Criteria:
- No reserved blank area on Level Select on desktop.
- Selecting a level via tile starts the level.
- Refreshing returns to Level Select (no ?level=… param).

Key Changes:
```css
/* app.css: ensure hidden always wins over media queries */
.hidden {
  display: none !important;
}
```

```js
// Removed deep-linking support:
// - Delete getUrlLevel, setUrlLevel, and related calls.
// - Initial route always shows Level Select.
document.addEventListener('DOMContentLoaded', () => {
  showLevelSelect();
});
```

---

## Phase 2 — Visible Progress Indicator

Scope:
- Add “Question X of Y” and a progress bar that fills as level advances.

UX:
- Counter in header near score.
- Subtle fill animation on progress.

Acceptance Criteria:
- Counter increments on Next.
- Progress bar reaches 100% on last question.

Changes:
```html
<!-- In .header -->
<div class="progress-wrap">
  <div id="qCounter" class="q-counter">Question 1 of 10</div>
  <div class="progress-bar">
    <div id="progressFill" class="progress-fill" style="width: 0%"></div>
  </div>
</div>
```

```css
.progress-wrap { display: flex; flex-direction: column; gap: 6px; min-width: 200px; }
.q-counter { font-size: 14px; color: var(--color-muted); }
.progress-bar { height: 8px; background: var(--color-border); border-radius: 999px; overflow: hidden; }
.progress-fill { height: 100%; background: var(--color-accent); transition: width 200ms ease; }
```

```js
function updateProgress() {
  const total = Math.max(questions.length, 1);
  const current = Math.min(idx + 1, total);
  document.getElementById('qCounter').textContent = `Question ${current} of ${total}`;
  const pct = Math.round(((current - 1) / total) * 100);
  document.getElementById('progressFill').style.width = `${pct}%`;
}

// Call in start() after questions load and in loadQuestion() after rendering options.
```

---

## Phase 3 — Positive Reinforcement and Celebrations

Scope:
- Rotate short praise messages for correct answers.
- Track streaks; trigger lightweight confetti at milestones (3, 5, then every 5).

UX:
- Feedback text “pops” with animation.
- Confetti is brief and non-blocking.

Acceptance Criteria:
- Correct answer shows praise from a small rotating set.
- Confetti appears at configured streaks.

Changes:
```css
/* Subtle feedback pop animation */
.feedback.pop { animation: fb-pop 300ms ease; }
@keyframes fb-pop {
  0% { transform: scale(0.96); opacity: 0.6; }
  100% { transform: scale(1); opacity: 1; }
}
```

```js
let streak = 0;
const PRAISE = ["Nice!", "Great job!", "Correct!", "Well done!", "You got it!", "Awesome!"];

function praiseText() {
  return PRAISE[Math.floor(Math.random() * PRAISE.length)];
}

function celebrateIfStreak() {
  if (streak === 3 || streak === 5 || (streak > 0 && streak % 5 === 0)) {
    smallConfetti();
  }
}

function smallConfetti() {
  const n = 20;
  for (let i = 0; i < n; i++) {
    const piece = document.createElement('div');
    Object.assign(piece.style, {
      position: 'fixed', left: (50 + (Math.random() * 20 - 10)) + 'vw', top: '-10px',
      width: '6px', height: '10px', background: ['#287DB0','#40A02B','#C7D11D','#f39c12','#e74c3c'][i % 5],
      opacity: '0.9', zIndex: 9999
    });
    document.body.appendChild(piece);
    const dx = (Math.random() * 2 - 1) * 50;  // drift
    const t = 800 + Math.random() * 700;
    piece.animate(
      [{ transform: `translate(0, 0) rotate(0deg)` }, { transform: `translate(${dx}px, 100vh) rotate(${Math.random() * 720}deg)` }],
      { duration: t, easing: 'ease-out' }
    ).onfinish = () => piece.remove();
  }
}

// Hook into checkAnswer():
// on correct:
streak++;
feedbackEl.textContent = praiseText();
feedbackEl.style.color = 'var(--color-success)';
feedbackEl.classList.remove('pop'); void feedbackEl.offsetWidth; feedbackEl.classList.add('pop');
celebrateIfStreak();

// on incorrect:
streak = 0;
```

Perf/Accessibility:
- No large libraries; CSS animation only.
- Honor `prefers-reduced-motion` by no-op confetti if set (Phase 6).

---

## Phase 4 — Hints and Assists

Scope:
- Show Hint and 50-50 assist. Reveal after an incorrect answer (or always, configurable).
- Hints are conceptual unless data supports more specificity.

Data Model (optional extensions):
- Add optional per-question fields to JSON:
```json
{
  "name": "2-methylbutane",
  "mol": "MOL_STRING",
  "distractors": ["pentane", "propane", "3-methylbutane"],
  "hints": [
    "Find the longest continuous carbon chain.",
    "Number from the end nearest a substituent."
  ],
  "explanation": "The longest chain has 4 carbons (butane). A methyl group at carbon 2."
}
```

UX:
- After wrong answer: reveal “Show hint” and “50-50”.
- 50-50 disables one random wrong option (conservative).

Acceptance Criteria:
- Hint displays in feedback area with accent color.
- 50-50 disables one visible wrong option and dims it.

Changes:
```html
<!-- In right column, under .feedback -->
<div class="assist" id="assistRow">
  <button id="hintBtn" class="btn" type="button">Show hint</button>
  <button id="fiftyBtn" class="btn" type="button">50-50</button>
</div>
```

```css
.assist { display: flex; gap: 8px; flex-wrap: wrap; }
.assist .btn { padding: 10px 12px; font-size: 14px; }
```

```js
const hintBtn = document.getElementById('hintBtn');
const fiftyBtn = document.getElementById('fiftyBtn');

let hintUsedThisQuestion = false;
let fiftyUsedThisQuestion = false;

hintBtn.addEventListener('click', () => {
  if (!questions[idx]) return;
  showHint(questions[idx]);
});

fiftyBtn.addEventListener('click', () => {
  if (fiftyUsedThisQuestion) return;
  fiftyUsedThisQuestion = true;
  eliminateOneWrong(questions[idx]);
});

function showHint(q) {
  const pool = Array.isArray(q.hints) && q.hints.length ? q.hints : [
    "Step 1: Find the longest continuous carbon chain.",
    "Step 2: Number the chain from the end nearest a substituent.",
    "Step 3: Name substituents and their positions.",
    "Step 4: Combine locants, prefixes (di-, tri-), substituent, parent chain."
  ];
  const msg = pool[Math.floor(Math.random() * pool.length)];
  feedbackEl.textContent = `Hint: ${msg}`;
  feedbackEl.style.color = 'var(--color-accent)';
  feedbackEl.classList.remove('pop'); void feedbackEl.offsetWidth; feedbackEl.classList.add('pop');
}

function eliminateOneWrong(q) {
  const buttons = [...choicesEl.querySelectorAll('button')];
  const wrong = buttons.filter(b => b.textContent !== q.name && !b.disabled);
  if (!wrong.length) return;
  const remove = wrong[Math.floor(Math.random() * wrong.length)];
  remove.disabled = true;
  remove.style.opacity = 0.4;
  feedbackEl.textContent = 'Removed one incorrect option.';
  feedbackEl.style.color = 'var(--color-muted)';
}

// Reset per-question in loadQuestion():
hintUsedThisQuestion = false; fiftyUsedThisQuestion = false;
hintBtn.classList.add('hidden'); fiftyBtn.classList.add('hidden');

// Reveal after wrong in checkAnswer():
hintBtn.classList.remove('hidden'); fiftyBtn.classList.remove('hidden');
```

---

## Phase 5 — Live Targets and Timer

Scope:
- Display star/time targets from levelConfig and highlight when player is on track.
- Optional elapsed timer indicator (no penalties).

UX:
- Two “chips” in the header: “80%+” (accuracy) and target time (e.g., 02:00).
- Chip turns “good” when current progress meets threshold.

Acceptance Criteria:
- Accuracy chip reflects current accuracy based on answered items.
- Time chip reflects elapsed vs. targetTimeMs.

Changes:
```html
<!-- In header -->
<div class="targets">
  <span id="accTarget" class="target-chip" title="Target accuracy for 2★">80%+</span>
  <span id="timeTarget" class="target-chip" title="Target time for 2★">02:00</span>
</div>
```

```css
.targets { display: flex; gap: 6px; align-items: center; color: var(--color-muted); font-size: 12px; }
.target-chip { padding: 4px 8px; border: 1px solid var(--color-border); border-radius: 999px; background: var(--color-surface); }
.target-chip.good { border-color: var(--color-success); color: var(--color-success); background: var(--color-success-soft); }
```

```js
let tickInterval = null;

function setupTargets() {
  const cfg = levelConfig[currentLevel] || levelConfig.default || {};
  document.getElementById('timeTarget').textContent = cfg.targetTimeMs ? msToClock(cfg.targetTimeMs) : '--';
  document.getElementById('accTarget').textContent = `${cfg.twoStarPct || 80}%+`;
}

function computeCurrentAccuracy() {
  const answered = idx; // completed items
  if (answered <= 0) return 0;
  return Math.round((score / answered) * 100);
}

function startTimerTick() {
  if (tickInterval) clearInterval(tickInterval);
  const started = window._runStartedAt;
  tickInterval = setInterval(() => {
    const cfg = levelConfig[currentLevel] || levelConfig.default || {};
    const elapsed = Date.now() - started;
    const timeGood = cfg.targetTimeMs ? elapsed <= cfg.targetTimeMs : false;
    const accGood = computeCurrentAccuracy() >= (cfg.twoStarPct || 80);
    document.getElementById('timeTarget').classList.toggle('good', !!timeGood);
    document.getElementById('accTarget').classList.toggle('good', !!accGood);
  }, 1000);
}

function stopTimerTick() { if (tickInterval) clearInterval(tickInterval); tickInterval = null; }

// Call setupTargets() + startTimerTick() in start() post-questions; stop in end().
```

---

## Phase 6 — Optional Audio/Visual Polish

Scope:
- Short correct/incorrect sounds with a mute toggle.
- Respect user preferences: `prefers-reduced-motion` and a user “mute” setting persisted in localStorage.

Acceptance Criteria:
- Sounds are short and subtle; user can mute.
- Confetti disabled if motion reduction is preferred.

Changes:
```html
<!-- Settings toggle (e.g., in header or footer) -->
<button id="muteBtn" class="btn" type="button" aria-pressed="false">Sound: On</button>
```

```js
let muted = JSON.parse(localStorage.getItem('quiz.muted') || 'false');

function setMuted(m) {
  muted = !!m;
  localStorage.setItem('quiz.muted', JSON.stringify(muted));
  const btn = document.getElementById('muteBtn');
  btn.textContent = `Sound: ${muted ? 'Off' : 'On'}`;
  btn.setAttribute('aria-pressed', String(!muted));
}

document.getElementById('muteBtn').addEventListener('click', () => setMuted(!muted));

const sndCorrect = new Audio('./sfx/correct.mp3');
const sndWrong = new Audio('./sfx/wrong.mp3');

function playSfx(audio) { if (!muted) { try { audio.currentTime = 0; audio.play(); } catch {} } }

// In checkAnswer(): playSfx(sndCorrect) or playSfx(sndWrong)

const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
function smallConfettiSafe() { if (!prefersReducedMotion) smallConfetti(); }
```

Assets:
- Provide two short audio files (~50–200ms), compressed.

---

## Phase 7 — Local Analytics/Telemetry

Scope:
- Capture lightweight events (per session) to evaluate changes:
  - timeToFirstAnswer, correctRate, avgStreak, hintUsageRate, 50-50 usage, drop-off index.
- Store in localStorage ring buffer; optional download as JSON from a hidden debug button.

Acceptance Criteria:
- No PII; all data local.
- Toggleable debug download for QA.

Changes:
```js
const TELEMETRY_KEY = 'alkaneQuiz.telemetry.v1';
function pushEvent(evt) {
  const buf = JSON.parse(localStorage.getItem(TELEMETRY_KEY) || '[]');
  buf.push({ t: Date.now(), ...evt });
  while (buf.length > 2000) buf.shift();
  localStorage.setItem(TELEMETRY_KEY, JSON.stringify(buf));
}

// Examples:
pushEvent({ type: 'answer', level: currentLevel, idx, correct: selected === q.name, streak });
pushEvent({ type: 'assist_hint', level: currentLevel, idx });
pushEvent({ type: 'assist_5050', level: currentLevel, idx });
pushEvent({ type: 'end', level: currentLevel, score, total: questions.length, timeMs });
```

---

## Phase 8 — A/B Toggles & Config Hardening

Scope:
- Feature flags in localStorage or URL only for QA:
  - ff_confetti, ff_sfx, ff_hints, ff_targets.
- Safe defaults (off for heavy extras).

Acceptance Criteria:
- QA can flip flags without code changes.
- App gracefully ignores unknown flags.

Changes:
```js
function ff(name, def = true) {
  const raw = localStorage.getItem(`ff.${name}`);
  if (raw === null) return def;
  return raw === 'true';
}
```

Usage:
- Gate confetti with `if (ff('confetti', true)) smallConfettiSafe();`
- Gate sfx, hints, targets similarly.

---

## Phase 9 — Future Enhancements

- Per-question explanations displayed after incorrect attempts.
- Smarter hints (e.g., computed chain length) if we add a parser or enrich JSON.
- Combo multipliers with session XP bonuses.
- “Practice mode” with infinite questions from unlocked levels.

---

## Performance Considerations

- Keep DOM operations minimal; reuse nodes in choices container.
- Confetti count low and duration short; no third-party libs.
- Timer tick at 1 Hz only; avoid frequent reflows.
- Audio assets small and preloaded.

---

## Accessibility

- Feedback region uses `aria-live="polite"`; keep status concise.
- Add `aria-hidden` toggles when hiding whole screens.
- Respect `prefers-reduced-motion: reduce` to disable confetti animations.
- Ensure assist buttons are keyboard reachable; visible focus styles already present.
- Use `role="status"` for milestone messages if needed.

---

## QA Plan

- Unit smoke: Start level, answer correct/incorrect, verify counter and progress bar.
- Streak logic: Validate confetti at 3, 5, 10.
- Hints: Verify hidden by default and revealed after incorrect; 50-50 disables one wrong item.
- Targets: Verify chip turns “good” when thresholds met; timer stops at end.
- Accessibility: Keyboard-only flow; screen reader labels; motion preference respected.
- Regression: Level Select locked/unlocked logic unaffected; no URL changes; localStorage profile intact.

---

## Rollout Plan

- Phase 2 + 3 first (progress + praise) — lowest risk, immediate value.
- Phase 4 (assists) next — adds complexity; ensure no exploit (e.g., 50-50 twice).
- Phase 5 (targets) — verify levelConfig coverage.
- Phase 6 (SFX) optional — ship muted by default; add toggle.
- Phase 7 (telemetry) — QA-only.
- Feature flags allow gradual enablement.

---

## Open Questions

- Do we want hints/explanations authored per question in JSON now or keep generic?
- Should 50-50 remove one or two options? (Start with one to keep difficulty.)
- Should we show a tiny elapsed timer explicitly, or keep only target chip?
- Do we add a small end-of-level “best streak” summary line?

---

## Appendix: Minimal Integration Points

```js
// start(): after questions load
setupTargets();
startTimerTick();
updateProgress();
streak = 0;

// loadQuestion(): end
updateProgress();
hintBtn.classList.add('hidden');
fiftyBtn.classList.add('hidden');

// checkAnswer(): correct
streak++;
feedbackEl.textContent = praiseText();
feedbackEl.style.color = 'var(--color-success)';
feedbackEl.classList.remove('pop'); void feedbackEl.offsetWidth; feedbackEl.classList.add('pop');
if (ff('confetti', true)) smallConfettiSafe();
if (ff('sfx', true)) playSfx(sndCorrect);

// checkAnswer(): incorrect
streak = 0;
hintBtn.classList.remove('hidden');
fiftyBtn.classList.remove('hidden');
if (ff('sfx', true)) playSfx(sndWrong);

// end():
stopTimerTick();
```

```css
/* Respect reduced motion */
@media (prefers-reduced-motion: reduce) {
  .feedback.pop { animation: none; }
}
```

