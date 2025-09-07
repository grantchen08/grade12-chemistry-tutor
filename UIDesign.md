# Alkane Naming Quiz — UI Design Documentation and Improvement Plan

Version: 1.0  
Owner: UI/Frontend  
Last updated: 2025-09-07

## 1. Overview

This document describes the current user interface (UI) of the Alkane Naming Quiz web app and proposes a staged plan to improve usability, responsiveness, accessibility, and polish for mobile and desktop.

The app renders a molecule (via ChemDoodle) and asks the user to select the correct IUPAC name from multiple choices, tracking score through a session.

Primary technologies:
- HTML/CSS/JS (single file: index.html)
- ChemDoodle ViewerCanvas (ChemDoodleWeb.js, ChemDoodleWeb.css)
- JSON data source: questions-level-1.json

---

## 2. Users and Goals

- Learners of organic chemistry nomenclature
- Instructors previewing or demonstrating
- Casual quiz-takers on mobile

User goals:
- See a clear molecule rendering
- Pick an answer quickly with clear feedback
- Track score and progress
- Adjust label visibility for clarity

---

## 3. Information Architecture & Layout

Page structure:
- Header (title, score)
- Canvas (molecule viewer)
- Choices (answer options: 2-column grid, collapses to 1 column <= 620px)
- Feedback line
- Controls (Next button, label mode select)

DOM outline:
- .wrap (container)
  - .header
    - h1 (Alkane Naming Quiz)
    - .score (Score: <span id="score">)
  - .row
    - canvas#viewer (520x300)
  - .row.choices#choices (answer buttons injected)
  - .row.feedback#feedback
  - .controls
    - button#next.next.hidden
    - select#labelMode (none/terminal/all)

Navigation:
- Mouse/touch: tap choices to answer, Next to proceed
- Keyboard: 1–4 to select options, Enter to advance

---

## 4. Component Inventory

- Header
  - Title: page context
  - Score: numeric tally
- Viewer
  - canvas#viewer
  - ChemDoodle.ViewerCanvas instance: viewer
  - JS styling for bonds/atoms
- Choices
  - .choices grid container
  - .btn buttons created per option
  - .btn.correct / .btn.incorrect states
- Feedback
  - #feedback (text content, color indicates correctness)
- Controls
  - #next.next.hidden (revealed after answer)
  - #labelMode select (affects carbon labeling)
- Utility CSS
  - .wrap: card-like container
  - .row: vertical rhythm
  - .hidden: display:none

---

## 5. Styling Summary

- Theme: dark, neutral bluish-gray background
- Typography: system-ui/Arial with default sizes
- Buttons:
  - .btn base styles, hover border highlight
  - .correct/.incorrect: green/red borders and subtle fills
- Grid:
  - .choices uses CSS grid
  - 2 columns default, 1 column <= 620px
- Controls:
  - .controls uses flex with gap

Current responsive rules:
- Only the .choices grid switches from 2 cols to 1 col at 620px
- Canvas has fixed width/height attributes (520x300)

---

## 6. Behavior and State

Key JS functions:
- validateMolString(s)
- shuffle(arr)
- applyLabelMode(mode) → toggles ChemDoodle label settings, repaint
- drawMol(molString) → parse MOL, scale, load into viewer
- loadQuestion() → clear UI, draw molecule, render shuffled options
- checkAnswer(q, selected, btn) → set disabled, update score, styles, feedback; reveal Next
- end() → show final score; Next becomes “Play Again”
- fetchQuestions(url) → fetch/validate JSON
- start() → reset, load, shuffle, begin
- Keyboard: Enter to advance; number keys 1–4 select choices

---

## 7. Current UX and Accessibility Assessment

Strengths:
- Clear flow and fast interactions
- Keyboard shortcuts implemented
- Visual feedback for correct/incorrect answers
- Simple, legible dark theme

Issues:
- No meta viewport → page scales like desktop on phones
- Canvas fixed size → either tiny or overflow on small screens
- Touch targets and font sizes may be small on some devices
- Controls can wrap sub-optimally; limited spacing control on narrow screens
- Focus visibility not explicitly defined; hover-only cues not usable on touch
- Feedback is not announced to screen readers (no live region)
- No “remaining questions” indicator; progress clarity could improve

---

## 8. Design Principles and Targets

- Responsive-first: fluid layout, canvas scales with container
- Accessible: perceivable feedback, keyboard/touch friendly, visible focus
- Touch ergonomics: 44px min height targets, >=16px font to avoid zoom on iOS
- Clarity: consistent spacing, hierarchy, and contrast
- Performance: crisp rendering on high-DPI devices

---

## 9. Proposed UI Improvements (Highlights)

- Add viewport meta for mobile scaling
- Make canvas responsive with JS resizing and DPR-aware drawing buffer
- Adjust layout for small screens (wrap controls, paddings, font sizes)
- Improve focus states and aria-live for feedback
- Optional: progress indicator, restart affordance, theme polish

---

## 10. Staged Implementation Plan

Phase 0 — Baseline mobile readiness (Low risk)
- Add viewport meta
- Slight CSS adjustments for mobile spacing

Acceptance criteria:
- On a phone, text and elements are readable without pinch-zoom
- No horizontal scrolling

Code:

```html
<!-- In <head> -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

```css
/* Minor spacing and touch target tweaks */
.wrap { max-width: 820px; margin: 40px auto; padding: 18px; }
.controls { margin-top: 12px; display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
.btn { font-size: 16px; line-height: 1.3; padding: 14px; }
.next { font-size: 16px; padding: 12px 16px; }
select { font-size: 16px; padding: 8px 10px; }
@media (max-width: 620px) {
  .wrap { margin: 16px; padding: 14px; }
}
```

Phase 1 — Responsive canvas and layout (Medium risk)
- Make canvas width fluid and maintain aspect ratio via JS
- Handle devicePixelRatio for crispness
- Debounce resize
- Ensure canvas resizes after loading each question

Acceptance criteria:
- Canvas fits container width; aspect ratio preserved
- Rendering looks sharp on Retina/HiDPI
- Resizing device/window updates canvas appropriately

Code:

```css
/* Ensure CSS width flows with container, JS sets drawing buffer */
canvas#viewer { width: 100%; height: auto; display: block; }
```

```js
// Add after viewer initialization
function sizeCanvas() {
  const canvas = document.getElementById('viewer');
  if (!canvas) return;

  // Compute container width
  const container = canvas.parentElement;
  const w = Math.max(280, Math.floor(container.clientWidth));
  const aspect = 300 / 520; // original canvas aspect
  const h = Math.round(w * aspect);

  // CSS size
  canvas.style.width = w + 'px';
  canvas.style.height = h + 'px';

  // DPR-aware drawing buffer
  const dpr = window.devicePixelRatio || 1;
  canvas.width = Math.round(w * dpr);
  canvas.height = Math.round(h * dpr);

  // Resize ChemDoodle viewer and scale context
  if (window.viewer) {
    if (typeof viewer.resize === 'function') viewer.resize(w, h);
    const ctx = canvas.getContext('2d');
    if (ctx) ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    if (viewer.repaint) viewer.repaint();
  }
}

function debounce(fn, ms = 150) {
  let t; return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
}

window.addEventListener('resize', debounce(sizeCanvas, 150));
// Call once after viewer init
sizeCanvas();
```

Integrate into loadQuestion:

```js
function loadQuestion() {
  feedbackEl.textContent = '';
  choicesEl.innerHTML = '';
  nextBtn.classList.add('hidden');

  if (idx >= questions.length) { end(); return; }
  const q = questions[idx];
  drawMol(q.mol);

  // Ensure canvas fits current layout
  sizeCanvas();

  const options = shuffle([q.name, ...q.distractors]);
  options.forEach(opt => {
    const btn = document.createElement('button');
    btn.className = 'btn';
    btn.textContent = opt;
    btn.addEventListener('click', () => checkAnswer(q, opt, btn));
    choicesEl.appendChild(btn);
  });
}
```

Phase 2 — Accessibility and interaction polish (Low–Medium risk)
- Add aria-live to feedback
- Ensure visible focus styles for buttons/select/Next
- Improve contrast for states
- Preserve keyboard support

Acceptance criteria:
- Screen readers announce feedback updates
- Tabbing reveals visible focus indicators
- Contrast ratios meet WCAG AA (>=4.5:1 for normal text)

Code:

```html
<!-- Feedback region becomes live -->
<div class="row feedback" id="feedback" aria-live="polite" role="status"></div>
```

```css
/* Visible focus rings for keyboard users */
.btn:focus, .next:focus, select:focus {
  outline: 2px solid rgba(77,163,255,0.9);
  outline-offset: 2px;
}

/* Ensure adequate contrast on feedback text */
.feedback { min-height: 24px; margin-top: 8px; color: #e7ecf3; }
.btn.correct { border-color: #2ecc71; background: rgba(46,204,113,0.18); }
.btn.incorrect { border-color: #e74c3c; background: rgba(231,76,60,0.18); }
```

Phase 3 — Responsive layout refinements and typography (Low risk)
- Clamp heading sizes
- Adjust grid behavior for medium screens
- Ensure controls wrap nicely; spacing around canvas improved
- Respect safe-area insets on iOS

Acceptance criteria:
- Title scales gracefully on small/large screens
- Choices grid uses 2 columns on larger phones, 1 on small
- No overlaps or cramped controls

Code:

```css
.header h1 { margin: 0; font-size: clamp(18px, 3.5vw, 28px); }
.choices { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
@media (max-width: 620px) { .choices { grid-template-columns: 1fr; } }

/* Safe area for modern phones */
body { padding-left: max(0px, env(safe-area-inset-left)); padding-right: max(0px, env(safe-area-inset-right)); }
```

Phase 4 — Optional UX enhancements (Medium risk, optional)
- Add progress indicator (e.g., “Question 3 of 10”)
- Add Restart button at end state
- Persist label mode preference (localStorage)
- Support reduced motion

Acceptance criteria:
- Users can see current progress
- Label mode persists across refresh
- Animations minimized if prefers-reduced-motion is set

Code:

```html
<!-- In header next to score -->
<div class="score">
  Score: <span id="score">0</span>
  • <span id="progress">1 / 10</span>
</div>
```

```js
// Update progress in loadQuestion()
const progressEl = document.getElementById('progress');
if (progressEl) progressEl.textContent = `${idx + 1} / ${questions.length}`;

// Persist label mode
const savedMode = localStorage.getItem('labelMode');
if (savedMode) labelModeEl.value = savedMode;
applyLabelMode(labelModeEl.value);
labelModeEl.addEventListener('change', e => {
  localStorage.setItem('labelMode', e.target.value);
  applyLabelMode(e.target.value);
});

// Reduced motion CSS
@media (prefers-reduced-motion: reduce) {
  * { transition: none !important; animation: none !important; }
}
```

Phase 5 — QA, performance, and cross-device testing (Ongoing)
- Test on iOS Safari, Android Chrome, Firefox, desktop browsers
- Verify no layout shifts, no horizontal scrolling
- Validate color/contrast and focus order
- Lighthouse pass (Performance, Accessibility, Best Practices)

Acceptance criteria:
- All supported browsers exhibit functional and visually consistent UI
- Accessibility score ≥ 95 in Lighthouse
- No console errors during normal operation

---

## 11. Risks and Mitigations

- ChemDoodle resizing behavior:
  - Risk: viewer.resize may not fully align with manual DPR scaling
  - Mitigation: set canvas CSS size first, then drawing buffer, call viewer.resize, then repaint; verify output
- Mobile device fragmentation:
  - Risk: layout issues on niche devices
  - Mitigation: test common breakpoints; add minimal media queries as needed
- Accessibility regressions:
  - Risk: visual polish overrides default focus outlines
  - Mitigation: explicit focus styles; test with keyboard only

---

## 12. Future Enhancements (Backlog)

- Light/dark auto-theming via prefers-color-scheme
- Larger answer choices count handling (adaptive grid)
- Timed mode or difficulty levels
- Audio/voice-over support for accessibility
- Analytics (privacy-respecting) to track completion and difficulty

Example theme hook:

```css
@media (prefers-color-scheme: light) {
  body { background: #f7f9fc; color: #1b2430; }
  .wrap { background: #fff; border-color: rgba(0,0,0,0.08); }
  .btn { background: rgba(0,0,0,0.03); color: #1b2430; border-color: rgba(0,0,0,0.12); }
}
```

---

## 13. Summary of Changes by File

index.html
- Add meta viewport
- Update CSS: touch targets, focus styles, responsive refinements, safe-area, canvas width:100%
- Add aria-live to feedback
- Add JS: sizeCanvas, debounce; integrate sizeCanvas in init and loadQuestion
- Optional: progress indicator, localStorage persistence, reduced motion

---

## 14. Testing Checklist

Functional:
- Answer selection sets correct/incorrect states
- Next advances; Enter key works
- Score increments correctly; end state message correct

Responsive:
- No horizontal scrolling at 320px width
- Canvas fills width; remains sharp on Retina
- Choices become single column at narrow widths

Accessibility:
- Tab through controls; visible focus at each step
- Screen reader announces feedback after answer
- Color contrast meets AA

Performance:
- No excessive reflows on resize (debounced)
- No blocking network calls on main thread (questions fetched once per start)

---

## 15. Rollout Plan

- Implement Phase 0–2 in a single release (mobile readiness + a11y)
- Soft launch for internal testing across devices
- Gather feedback and defect logs
- Follow up with Phase 3–4 refinements
- Continuous QA (Phase 5)

