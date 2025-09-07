# Alkane Naming Quiz – Multi-Level, Progress Persistence, and Replay Incentives
Design Document (v1.0)

Author: Grant Chen
Last updated: 2025-09-07



## Summary

This document proposes a scalable design to add multi-level support (e.g., 7+ levels), persistent player progress via localStorage, and replay incentives (stars, XP, badges) to the Alkane Naming Quiz. The plan is phased to minimize regressions and to allow incremental delivery with measurable outcomes.


## Goals

- Allow users to choose among multiple levels (initially Level 1 and 2; scalable to 7+).
- Persist user progress locally (best score, stars, attempts, times, XP, badges).
- Encourage replay via mastery targets (stars/medals), “personal best” tracking, and lightweight awards.
- Maintain current single-page, asset-light approach (no backend required).
- Preserve accessibility and responsiveness; avoid breaking current quiz flow.


## Non-goals

- Server-backed accounts or cloud sync (out of scope for now).
- Cross-device profile migration (optional import/export only).
- Complex social leaderboards or competitive features.


## Current State

- Single-level quiz (Level 1) presented on page load.
- Questions fetched from JSON (“questions-level-1.json”).
- Molecules rendered via ChemDoodle on a responsive canvas.
- Score tracking only for the current session; no persistence.
- Simple completion screen with restart option.


## Proposed Changes (High-Level)

1. Level Selection UX
   - A dedicated Level Select screen with tiles for levels 1..N, showing lock/completed state, stars, best %, best time, and attempts.
   - Deep link support (?level=N) and “Continue” button (last played level).

2. Local Persistence
   - localStorage-based profile object with versioning.
   - Track per-level bests, attempts, star ratings; global XP, badges, and daily streaks.

3. Replay Incentives
   - Star system (0–3) driven by accuracy and time thresholds; optional “perfect” medal for 100%.
   - XP awarded for clears, personal best improvements, and streaks.
   - End-of-level summary highlighting improvements.

4. Gating/Progression
   - Lock Level N until prior level completed with at least 1 star (configurable).
   - Visible requirements on locked tiles.

5. Instrumentation (local)
   - Minimal metrics in localStorage to analyze drop-offs and retries (optional and private to device).


## Architecture

- Single-page app (SPA) with two main UI states:
  - LevelSelectScreen: grid of levels and “Continue.”
  - GameScreen: existing quiz layout (canvas, choices, feedback, Next).
- Router-lite behavior via URL params (?level=N) and History API (replaceState).
- Storage module (inline or separate file) encapsulates all localStorage IO, schema, and award logic.
- Config object for levels, thresholds, and gating rules with sane defaults.


## Data Model

Storage key: alkaneQuiz.v1.profile

Schema:
```json
{
  "version": 1,
  "lastLevel": 1,
  "levels": {
    "1": {
      "bestScore": 8,
      "maxScore": 10,
      "bestPct": 80,
      "bestTimeMs": 92000,
      "stars": 2,
      "attempts": 5,
      "lastPlayedAt": 1736200000000
    }
  },
  "badges": {
    "perfect": 1,
    "speedster": 2,
    "comeback": 1
  },
  "xp": 340,
  "daily": {
    "lastActive": "2025-09-07",
    "streak": 3
  }
}
```

Versioning/Migrations:
- Store version at root. If breaking changes occur, add a migration step (v2) while reading from localStorage and re-save upgraded schema.


## Level Content

- JSON files per level: questions-level-1.json, questions-level-2.json, …, questions-level-N.json
- Schema unchanged: array of { name: string, mol: string(MOL), distractors: string[] }
- Discovery of TOTAL_LEVELS either:
  - Static constant (e.g., 7), or
  - Metadata manifest (optional future: levels.json).


## Configuration

Global default thresholds:
```js
const levelConfig = {
  default: { targetTimeMs: 150000, twoStarPct: 80, threeStarPct: 95 },
  1: { targetTimeMs: 120000 },
  2: { targetTimeMs: 150000 }
  // Add overrides as needed for later levels
};
```

Gating rule:
- Level k is unlocked when level k-1 has stars >= 1 (configurable).


## UX and Accessibility

Level Select screen (scalable):
- Tiles (buttons) with:
  - Level title (e.g., “Level 3”)
  - Stars visualization (★/☆), aria-label announces “2 out of 3 stars”
  - Metadata: Best %, Best time, Attempts
  - Lock state (disabled, visible “Unlock: earn 1★ in Level k-1”)
- Continue button jumps to lastLevel.

Game screen:
- End-of-level summary shows:
  - Current run score, accuracy, stars earned, XP gained.
  - Any personal best improvements highlighted.
  - Primary action: Next Level (if unlocked); Secondary: Replay, Level Select.

Keyboard/AT:
- Tiles are focusable buttons with descriptive aria-labels and visible focus outlines.
- Maintain tab order and ensure role=”status” for results updates remains intact.


## Security/Privacy

- All data stored in browser localStorage; no transmission.
- Provide a simple “Reset Progress” and optional “Export/Import Progress” (JSON download/upload).
- Warn users that clearing site data will remove progress.


## Performance

- Storage footprint small (a few KB).
- Lazy-render tiles; reuse existing canvas and quiz logic.
- No change to molecule rendering path.


## Rollout Plan (Phased)

To reduce regressions, we propose four phases. Each phase produces shippable value and includes safeguards and tests.

### Phase 0 – Foundation (No UX change)

Scope:
- Introduce storage module (getProfile, saveProfile, recordLevelResult, computeStars, updateDaily).
- Add timer start/stop for a run.
- End-of-level: call recordLevelResult() and include XP/stars in the summary (no level selector yet).

Acceptance:
- Local profile updates on completion.
- XP increments; stars computed; no breaking change to existing gameplay.

Testing:
- Unit test computeStars() thresholds (manual console tests if no framework).
- Validate schema persists/reloads across refresh.
- Backwards compatibility: page loads even if localStorage is empty/corrupt (fallback to defaults).


### Phase 1 – Level Selection (2 levels)

Scope:
- Add Level Select screen with tiles for levels 1–2.
- Deep link (?level=1/2) and Continue button.
- Game start uses selected level; restart flows unchanged.
- Gating: lock Level 2 until Level 1 has >= 1 star (configurable).

Acceptance:
- User can pick Level 1 or 2.
- Locked Level 2 shows requirement clearly.
- Last played level persists; Continue works.

Testing:
- Navigation between Level Select and Game screens.
- Refresh with ?level=2 loads Level 2.
- Keyboard-only selection and focus management.


### Phase 2 – Scale to N Levels (e.g., 7) and Replay UX

Scope:
- Expand grid to TOTAL_LEVELS (7).
- Show best %, best time, attempts, stars; dynamic locked state per tile.
- End-of-level actions: Next Level (primary), Replay, Level Select.

Acceptance:
- Tiles render for all levels; locks behave per gating rule.
- End-of-level shows improvements and offers three actions.
- Replay retains level context and updates attempts.

Testing:
- Large level counts still responsive.
- Verify unlocking chain (1→2→…→7).
- End-of-level transitions correct for all levels.


### Phase 3 – Awards and Quality-of-Life

Scope:
- Badges: perfect (100%), speedster (beat target time), comeback (improve after failure).
- Daily streak tracking; small XP bonus on streak days.
- Optional: Export/Import progress; Reset progress button.
- Optional: Featured Level of the day (bonus XP).

Acceptance:
- Badges increment as criteria met; streak increases across days.
- XP gains reflect improvements (new stars, PB, streak).
- Import/Export works and validates schema version.

Testing:
- Clock-based tests (simulate date change).
- Import invalid JSON handling with safe fallback.
- Accessibility checks for tile aria-labels and button focus.


## Detailed Design

### Storage Module (Phase 0)

```js
const STORAGE_KEY = 'alkaneQuiz.v1.profile';

function getProfile() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) return JSON.parse(raw);
  } catch (e) { /* safe fallback */ }
  return { version: 1, lastLevel: 1, levels: {}, badges: {}, xp: 0, daily: { lastActive: null, streak: 0 } };
}

function saveProfile(p) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(p));
}

function nowISODate() { return new Date().toISOString().slice(0,10); }

function updateDaily(profile) {
  const today = nowISODate();
  if (profile.daily.lastActive !== today) {
    const y = new Date(today); y.setDate(y.getDate() - 1);
    const yISO = y.toISOString().slice(0,10);
    profile.daily.streak = (profile.daily.lastActive === yISO) ? (profile.daily.streak + 1) : 1;
    profile.daily.lastActive = today;
  }
}

function computeStars({ pct, timeMs, targetTimeMs, twoStarPct=80, threeStarPct=95 }) {
  let stars = 0;
  if (pct > 0) stars = 1;
  if (pct >= twoStarPct || (targetTimeMs && timeMs && timeMs <= targetTimeMs)) stars = Math.max(stars, 2);
  if ((pct >= threeStarPct) && (targetTimeMs && timeMs && timeMs <= targetTimeMs)) stars = Math.max(stars, 3);
  return stars;
}

function recordLevelResult(levelId, { score, maxScore, timeMs }, config) {
  const profile = getProfile();
  updateDaily(profile);
  profile.lastLevel = Number(levelId);

  const pct = Math.round((score / Math.max(1, maxScore)) * 100);
  const cfg = config[levelId] || config.default || {};
  const stars = computeStars({ pct, timeMs, targetTimeMs: cfg.targetTimeMs, twoStarPct: cfg.twoStarPct, threeStarPct: cfg.threeStarPct });

  const prev = profile.levels[levelId] || { bestScore: 0, maxScore, bestPct: 0, bestTimeMs: null, stars: 0, attempts: 0, lastPlayedAt: 0 };
  const improvements = {
    newBestScore: score > (prev.bestScore || 0),
    newBestPct: pct > (prev.bestPct || 0),
    newBestTime: prev.bestTimeMs ? (timeMs && timeMs < prev.bestTimeMs) : !!timeMs,
    newStars: stars > (prev.stars || 0)
  };

  profile.levels[levelId] = {
    bestScore: Math.max(prev.bestScore || 0, score),
    maxScore,
    bestPct: Math.max(prev.bestPct || 0, pct),
    bestTimeMs: (prev.bestTimeMs && timeMs) ? Math.min(prev.bestTimeMs, timeMs) : (prev.bestTimeMs || timeMs || null),
    stars: Math.max(prev.stars || 0, stars),
    attempts: (prev.attempts || 0) + 1,
    lastPlayedAt: Date.now()
  };

  // XP and badges
  let xpGain = 10; // base per clear
  if (improvements.newStars) xpGain += 15;
  if (improvements.newBestPct) xpGain += 10;
  if (improvements.newBestTime) xpGain += 10;
  if (pct === 100) { xpGain += 20; profile.badges.perfect = (profile.badges.perfect || 0) + 1; }
  if (profile.daily.streak >= 3) xpGain += 5;

  profile.xp += xpGain;
  saveProfile(profile);

  return { profile, pct, stars, xpGain, improvements };
}

function getLevelSummary(levelId) {
  const p = getProfile();
  return p.levels[levelId] || null;
}
```


### Level Select UI (Phase 1 and 2)

HTML (added container):
```html
<div id="levelSelectScreen" style="display:none">
  <h2>Select Level</h2>
  <div id="levelGrid" class="grid"></div>
  <button id="continueBtn" class="next">Continue</button>
</div>
```

CSS (reusing tokens):
```css
#levelSelectScreen .grid {
  display: grid; gap: 12px;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
}
.level-tile {
  border: 1px solid var(--color-border); border-radius: 10px; padding: 12px;
  background: var(--color-surface); cursor: pointer; text-align: left;
}
.level-tile.locked { opacity: 0.6; cursor: not-allowed; }
.level-title { font-weight: 700; margin-bottom: 6px; }
.level-meta { font-size: 12px; color: var(--color-muted); }
.stars { color: gold; letter-spacing: 2px; }
```

JS (rendering tiles, gating, navigation):
```js
const TOTAL_LEVELS = 7;
const levelGrid = document.getElementById('levelGrid');
const continueBtn = document.getElementById('continueBtn');

function msToClock(ms) {
  const s = Math.floor(ms/1000);
  const m = Math.floor(s/60);
  const r = s % 60;
  return `${String(m).padStart(2,'0')}:${String(r).padStart(2,'0')}`;
}

function renderLevelGrid() {
  const profile = getProfile();
  levelGrid.innerHTML = '';
  for (let i = 1; i <= TOTAL_LEVELS; i++) {
    const s = getLevelSummary(String(i)) || {};
    const prev = getLevelSummary(String(i-1)) || {};
    const locked = i > 1 && !(prev.stars >= 1);
    const stars = '★'.repeat(s.stars || 0) + '☆'.repeat(3 - (s.stars || 0));
    const best = s.bestPct ? `${s.bestPct}%` : '-';
    const time = s.bestTimeMs ? msToClock(s.bestTimeMs) : '-';
    const attempts = s.attempts || 0;

    const btn = document.createElement('button');
    btn.className = 'level-tile' + (locked ? ' locked' : '');
    btn.disabled = locked;
    btn.setAttribute('aria-label',
      locked
        ? `Level ${i} locked. Unlock by earning 1 star in Level ${i-1}.`
        : `Level ${i}. Best ${best}. ${s.stars || 0} of 3 stars. Attempts ${attempts}.`
    );
    btn.innerHTML = `
      <div class="level-title">Level ${i}</div>
      <div class="stars" aria-hidden="true">${stars}</div>
      <div class="level-meta">Best: ${best} • Time: ${time} • Attempts: ${attempts}</div>
      ${locked ? `<div class="level-meta">Unlock: earn 1★ in Level ${i-1}</div>` : ''}
    `;
    btn.addEventListener('click', () => {
      start(i);
      showGame();
    });
    levelGrid.appendChild(btn);
  }
  continueBtn.onclick = () => {
    const p = getProfile();
    start(p.lastLevel || 1);
    showGame();
  };
}

function showLevelSelect() {
  document.getElementById('levelSelectScreen').style.display = 'block';
  document.querySelector('.wrap').style.display = 'none'; // hide game UI
  renderLevelGrid();
}

function showGame() {
  document.getElementById('levelSelectScreen').style.display = 'none';
  document.querySelector('.wrap').style.display = 'block';
}
```

Router-lite:
- On load, if no level param, show LevelSelectScreen.
- If ?level=N is present, start that level if unlocked; else show LevelSelectScreen.

```js
function getUrlLevel() {
  const params = new URLSearchParams(location.search);
  const lvl = parseInt(params.get('level'), 10);
  return Number.isFinite(lvl) ? lvl : null;
}

function setUrlLevel(level) {
  const params = new URLSearchParams(location.search);
  params.set('level', String(level));
  history.replaceState({}, '', `${location.pathname}?${params.toString()}`);
}

document.addEventListener('DOMContentLoaded', () => {
  const urlLevel = getUrlLevel();
  if (urlLevel) { start(urlLevel); showGame(); }
  else { showLevelSelect(); }
});
```


### Game Flow Adjustments

- Start timer when a run begins; stop at completion to compute timeMs.
- On end(), call recordLevelResult(), update summary, and present actions.

```js
function start(level = currentLevel) {
  currentLevel = level;
  setUrlLevel(level);
  score = 0;
  scoreEl.textContent = score;
  idx = 0;
  window._runStartedAt = Date.now();

  fetchQuestions(`./questions-level-${currentLevel}.json`)
    .then(data => { questions = shuffle(data); sizeCanvas(); loadQuestion(); })
    .catch(e => {
      console.error(e);
      feedbackEl.style.color = 'var(--color-error)';
      feedbackEl.textContent = 'Failed to load questions. See console for details.';
      questions = [];
    });
}

function end() {
  const total = questions.length;
  const pct = total ? Math.round((score / total) * 100) : 0;
  const timeMs = window._runStartedAt ? (Date.now() - window._runStartedAt) : null;

  const res = recordLevelResult(String(currentLevel), { score, maxScore: total, timeMs }, levelConfig);

  // Build summary with improvements
  const imp = res.improvements;
  const improvementsText = [
    imp.newStars ? `New star rating: ${res.stars}★` : '',
    imp.newBestPct ? `New best accuracy: ${res.profile.levels[currentLevel].bestPct}%` : '',
    imp.newBestTime ? `New best time: ${msToClock(res.profile.levels[currentLevel].bestTimeMs)}` : ''
  ].filter(Boolean).join(' • ');

  choicesEl.innerHTML = '';
  feedbackEl.style.color = 'var(--color-text)';
  feedbackEl.innerHTML = `
    Quiz complete. Score: <strong>${score} / ${total}</strong> (${pct}%).
    <br/>Stars: ${'★'.repeat(res.stars)}${'☆'.repeat(3-res.stars)} • XP +${res.xpGain}
    ${improvementsText ? `<br/><span style="color: var(--color-success)">${improvementsText}</span>` : ''}
  `;

  // Actions
  nextBtn.textContent = 'Next Level';
  nextBtn.classList.remove('hidden');
  nextBtn.onclick = () => {
    const next = Math.min(TOTAL_LEVELS, currentLevel + 1);
    start(next);
  };

  // Add Replay and Level Select
  injectSecondaryActions();
}

function injectSecondaryActions() {
  const container = document.getElementById('rightCol');

  let replay = document.getElementById('replayBtn');
  if (!replay) {
    replay = document.createElement('button');
    replay.id = 'replayBtn';
    replay.className = 'next';
    replay.textContent = 'Replay Level';
    container.appendChild(replay);
  }
  replay.onclick = () => start(currentLevel);

  let select = document.getElementById('selectBtn');
  if (!select) {
    select = document.createElement('button');
    select.id = 'selectBtn';
    select.className = 'next';
    select.textContent = 'Level Select';
    container.appendChild(select);
  }
  select.onclick = showLevelSelect;
}
```


## Testing Strategy

- Functional:
  - Load, play, and complete a level; verify scores and stars recorded.
  - Lock/unlock logic across chain of levels.
  - Deep linking (?level=N) and navigation back to Level Select.
- Persistence:
  - Refresh page; ensure profile persists and Continue works.
  - Corrupt localStorage entry gracefully resets to defaults.
- Accessibility:
  - Keyboard-only navigation through Level Select and game.
  - Screen reader announces tile states correctly.
- Performance:
  - Rendering 7+ tiles on mobile and desktop; no jank on transitions.
- Browser coverage:
  - Latest Chrome, Firefox, Edge, Safari (desktop and iOS/Android).
- Time-based logic:
  - Simulate different durations; verify star/time thresholds and awards.


## Risks and Mitigations

- Risk: localStorage quota errors or JSON corruption.
  - Mitigation: wrap reads/writes in try/catch; fall back to defaults; provide “Reset Progress.”
- Risk: Gating frustrates users if thresholds too strict.
  - Mitigation: start with low bar (1★ = completion) and adjustable config; show clear requirements.
- Risk: UI clutter on smaller screens.
  - Mitigation: responsive grid; concise metadata; use accordions/line wraps sparingly.
- Risk: Regressions to existing quiz flow.
  - Mitigation: phased rollout; keep game logic isolated from selection UI; manual smoke tests per phase.


## Metrics and Success Criteria

- Adoption: % of users who reach Level Select (Phase 1+).
- Engagement: average attempts per level; % of replays after completion.
- Mastery: distribution of stars by level; % of users improving PBs.
- Retention (local): daily streaks >= 2 days.

All metrics are local-only (no network) unless an analytics plan is introduced later.


## Rollback Plan

- Since changes are client-side only, revert by deploying previous index.html that omits Level Select and storage references.
- Storage remains in localStorage but unused; safe to leave or provide reset action.


## Open Questions

- Do we want per-level custom thresholds now, or use global defaults until levels diverge in difficulty?
- Should “Next Level” respect lock state (i.e., if locked, route to Level Select with requirement tooltip)?
- Do we need Import/Export in Phase 3 or as a separate Phase 4?


## Appendix

### Example Level Tile aria-labels

- Unlocked: “Level 3. Best 80 percent. 2 of 3 stars. Attempts 5.”
- Locked: “Level 4 locked. Unlock by earning 1 star in Level 3.”


### Example Import/Export (optional)

```js
function exportProgress() {
  const data = localStorage.getItem(STORAGE_KEY) || '{}';
  const blob = new Blob([data], { type: 'application/json' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'alkane-quiz-progress.json';
  a.click();
}

function importProgress(file) {
  const reader = new FileReader();
  reader.onload = () => {
    try {
      const obj = JSON.parse(reader.result);
      if (obj && typeof obj.version === 'number') {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(obj));
        alert('Progress imported.');
        location.reload();
      } else {
        alert('Invalid progress file.');
      }
    } catch {
      alert('Invalid JSON.');
    }
  };
  reader.readAsText(file);
}
```
