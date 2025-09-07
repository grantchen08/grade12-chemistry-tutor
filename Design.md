# Alkane Naming Quiz — Design Document

Version: 1.0  
Owner: Grant Chen  
Status: Draft  
Last updated: 2025-09-07

## 1. Purpose

Build a progressive, standards-aligned quiz for Grade 12 chemistry focused on naming alkanes from molecular structures. The quiz should scaffold from recognition of straight-chain alkanes to complex branching, tie-breakers, and cycloalkanes, while reinforcing IUPAC rules through deliberate distractors and optional explanations.

## 2. Goals and Success Metrics

- Pedagogical goals
  - Students can correctly name C1–C10 alkanes and common cycloalkanes.
  - Students apply IUPAC rules: longest chain, lowest set of locants, alphabetical order, and ring-vs-chain parent selection.
  - Students avoid common pitfalls (wrong parent chain, wrong numbering direction, misapplied prefixes).

- Product goals
  - 90–120 well-constructed questions, grouped by difficulty levels.
  - 85%+ learner accuracy after completing levels 1–5.
  - <12 seconds median response time for straight-chain recognition (Level 1).
  - Optional “Explain” feedback increases post-explanation accuracy by ≥15% on similar items.

- Technical goals
  - Maintain compatibility with existing web app (index.html, questions.json).
  - Keep assets minimal: MOL V2000 strings, four-option multiple choice, keyboard shortcuts maintained.
  - Simple content pipeline: export MOL from common editors; JSON-based content with validation.

## 3. Scope

- In scope
  - Acyclic alkanes C1–C10.
  - Branched alkanes with methyl and ethyl substituents; trimethyl patterns.
  - Cycloalkanes (C3–C8) and simple branched cycloalkanes (1–2 substituents).
  - Four-option multiple-choice items.
  - Optional per-question explanations.

- Out of scope (v1)
  - Stereochemistry (R/S, cis/trans).
  - Functional groups beyond alkanes.
  - Automated structure generation; items are curated.

## 4. Target Users and Use Cases

- Personas
  - Grade 12 chemistry students practicing nomenclature.
  - Teachers assigning quick drills or checkpoints.
  - Self-paced learners reviewing for exams.

- Use cases
  - Level-by-level practice with mastery gates.
  - Timed drills for test readiness.
  - Mixed review sessions for spiral learning.

## 5. Pedagogical Design

### 5.1 Level Progression

- Level 1 — Straight-chain alkanes
  - Objective: Identify parent chain C1–C10; apply “-ane”.
  - Count: 12–15 items.
  - Examples: methane, ethane, … decane.
  - Distractors: neighboring chain lengths, cyclo- variants, -ene/-yne.

- Level 2 — Single substituent (one branch)
  - Objective: Longest chain, numbering from end with lowest locant.
  - Count: 15–18 items.
  - Examples: 2-methylpropane, 2-methylbutane, 3-methylpentane, 3-ethylhexane.
  - Distractors: wrong numbering, wrong parent chain, common vs IUPAC name.

- Level 3 — Two substituents
  - Objective: di-/tri- prefixes, alphabetical order (ignore di-/tri-), lowest set of locants.
  - Count: 18–20 items.
  - Examples: 2,3-dimethylbutane; 3,3-dimethylpentane; 3-ethyl-2-methylpentane.
  - Distractors: wrong alphabetical order, wrong locant set, wrong parent chain.

- Level 4 — Longest-chain traps and tie-breakers
  - Objective: True longest chain; first point of difference in ties.
  - Count: 15 items.
  - Examples: 3-methylpentane (vs 2-ethylbutane); 4-ethylhexane (vs 3,3-dimethylpentane).
  - Distractors: shorter-parent names; incorrect tie-breaking.

- Level 5 — Multiple substituents and numbering strategy
  - Objective: Minimize entire set of locants; handle 3+ substituents.
  - Count: 15–18 items.
  - Examples: 2,2,4-trimethylpentane; 3-ethyl-2,4-dimethylhexane.
  - Distractors: lowest first locant fallacy; alphabetization errors.

- Level 6 — Cycloalkanes (recommended)
  - Objective: Name rings; ring as parent if larger; correct ring numbering.
  - Count: 12–15 items.
  - Examples: cyclopentane, cyclohexane, methylcyclohexane, 1,3-dimethylcyclopentane.
  - Distractors: acyclic vs cyclic confusion, wrong ring numbering.

- Level 7 — Mixed cumulative challenge
  - Objective: Integrate all rules on C7–C10 and ring/chain choices.
  - Count: 20 items.
  - Examples: 2,2,4-trimethylpentane; 4-ethyl-2,3-dimethylhexane; methylcycloheptane.
  - Distractors: parent chain swaps; locant/alpha errors; cyclo vs acyclic.

### 5.2 Spiral Review and Mastery

- Checkpoints: 5-question mini-tests at end of each level (≥80–90% to pass).
- Mixed reviews: After Levels 3 and 5, 8–10 items sampling prior content.
- Final assessment: 25–30 questions with weighted coverage (L1/2: 25%, L3/4: 35%, L5: 25%, L6: 15%).

## 6. Content Model

### 6.1 Question JSON Schema (backward-compatible extension)

Existing fields remain; new fields are optional. The app should ignore unknown fields safely.

```json
{
  "type": "object",
  "required": ["name", "mol", "distractors"],
  "properties": {
    "id": { "type": "string", "description": "Unique ID (e.g., l2-q-014)" },
    "name": { "type": "string", "description": "Correct IUPAC name" },
    "mol": { "type": "string", "description": "V2000 MOL block (carbon skeleton OK)" },
    "distractors": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 2,
      "maxItems": 5
    },
    "level": {
      "type": "integer",
      "minimum": 1,
      "maximum": 7,
      "description": "Curricular level"
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" },
      "description": "e.g., ['longest-chain', 'alphabetization', 'cyclo']"
    },
    "explain": {
      "type": "string",
      "description": "Brief rationale shown after answering"
    }
  },
  "additionalProperties": true
}
```

Notes:
- Keep “name”, “mol”, “distractors” to satisfy current app.
- “level” enables sequencing/filtering; “explain” improves learning.
- “id” aids analytics and de-duplication.

### 6.2 File Structure

- index.html — existing app UI and logic.
- questions.json — content bank (can include level, explain).
- Optional future:
  - questions-level-1.json, … (if splitting by level).
  - assets/schema/question.schema.json (for validation in CI).

### 6.3 Example Items (ready-to-paste)

The following examples conform to the current validator (counts line matches, minimal carbon-only V2000).

```json
[
  {
    "id": "l1-q-003",
    "level": 1,
    "name": "hexane",
    "mol": "hexane\n  ChatGPT 09072025\n\n  6  5  0  0  0  0            999 V2000\n    0.0000    0.0000    0.0000 C   0  0\n    1.5000    0.0000    0.0000 C   0  0\n    3.0000    0.0000    0.0000 C   0  0\n    4.5000    0.0000    0.0000 C   0  0\n    6.0000    0.0000    0.0000 C   0  0\n    7.5000    0.0000    0.0000 C   0  0\n  1  2  1  0\n  2  3  1  0\n  3  4  1  0\n  4  5  1  0\n  5  6  1  0\nM  END",
    "distractors": ["pentane", "heptane", "cyclohexane"],
    "tags": ["straight-chain"]
  },
  {
    "id": "l2-q-008",
    "level": 2,
    "name": "3-methylpentane",
    "mol": "3-methylpentane\n  ChatGPT 09072025\n\n  6  5  0  0  0  0            999 V2000\n    0.0000    0.0000    0.0000 C   0  0\n    1.5000    0.0000    0.0000 C   0  0\n    3.0000    0.0000    0.0000 C   0  0\n    4.5000    0.0000    0.0000 C   0  0\n    6.0000    0.0000    0.0000 C   0  0\n    3.0000    1.2000    0.0000 C   0  0\n  1  2  1  0\n  2  3  1  0\n  3  4  1  0\n  4  5  1  0\n  3  6  1  0\nM  END",
    "distractors": ["2-methylpentane", "2-ethylbutane", "hexane"],
    "tags": ["single-substituent", "longest-chain"],
    "explain": "Longest chain has 5 carbons; numbering from either end gives the substituent at C3 (not 2-)."
  },
  {
    "id": "l3-q-012",
    "level": 3,
    "name": "2,3-dimethylbutane",
    "mol": "2,3-dimethylbutane\n  ChatGPT 09072025\n\n  6  5  0  0  0  0            999 V2000\n    0.0000    0.0000    0.0000 C   0  0\n    1.5000    0.0000    0.0000 C   0  0\n    3.0000    0.0000    0.0000 C   0  0\n    4.5000    0.0000    0.0000 C   0  0\n    1.5000    1.2000    0.0000 C   0  0\n    3.0000    1.2000    0.0000 C   0  0\n  1  2  1  0\n  2  3  1  0\n  3  4  1  0\n  2  5  1  0\n  3  6  1  0\nM  END",
    "distractors": ["2,2-dimethylbutane", "3,3-dimethylbutane", "pentane"],
    "tags": ["two-substituents", "lowest-set-of-locants"],
    "explain": "Two methyls on a butane parent; numbering 2,3 minimizes the set of locants."
  },
  {
    "id": "l4-q-006",
    "level": 4,
    "name": "4-ethylhexane",
    "mol": "4-ethylhexane\n  ChatGPT 09072025\n\n  8  7  0  0  0  0            999 V2000\n    0.0000    0.0000    0.0000 C   0  0\n    1.5000    0.0000    0.0000 C   0  0\n    3.0000    0.0000    0.0000 C   0  0\n    4.5000    0.0000    0.0000 C   0  0\n    6.0000    0.0000    0.0000 C   0  0\n    7.5000    0.0000    0.0000 C   0  0\n    4.5000    1.2000    0.0000 C   0  0\n    5.7000    1.2000    0.0000 C   0  0\n  1  2  1  0\n  2  3  1  0\n  3  4  1  0\n  4  5  1  0\n  5  6  1  0\n  4  7  1  0\n  7  8  1  0\nM  END",
    "distractors": ["3-ethylhexane", "3,3-dimethylpentane", "heptane"],
    "tags": ["longest-chain", "tie-breaker"],
    "explain": "The true parent is hexane; the side chain is ethyl at C4 (not two methyls on a pentane)."
  },
  {
    "id": "l6-q-004",
    "level": 6,
    "name": "methylcyclohexane",
    "mol": "methylcyclohexane\n  ChatGPT 09072025\n\n  7  7  0  0  0  0            999 V2000\n    0.0000    1.2000    0.0000 C   0  0\n    1.0392    0.6000    0.0000 C   0  0\n    1.0392   -0.6000    0.0000 C   0  0\n    0.0000   -1.2000    0.0000 C   0  0\n   -1.0392   -0.6000    0.0000 C   0  0\n   -1.0392    0.6000    0.0000 C   0  0\n    0.0000    2.4000    0.0000 C   0  0\n  1  2  1  0\n  2  3  1  0\n  3  4  1  0\n  4  5  1  0\n  5  6  1  0\n  6  1  1  0\n  1  7  1  0\nM  END",
    "distractors": ["ethylcyclohexane", "cycloheptane", "hexane"],
    "tags": ["cyclo", "ring-parent"],
    "explain": "Ring is parent; one methyl substituent on cyclohexane."
  }
]
```

## 7. Item Development Guidelines

- Structure creation
  - Use ChemDoodle Sketcher, Avogadro, MarvinSketch for drawing; export V2000 MOL.
  - Use implicit hydrogens; carbon-only is sufficient for rendering and naming.
  - Ensure counts line matches atom/bond lines; keep 2D planar coordinates for clarity.

- Distractor design
  - “Near-miss” names only: wrong numbering (2- vs 3-), wrong parent (ethyl+shorter chain), wrong alphabetical order.
  - Include cyclo vs acyclic and -ene/-yne distractors in Level 1 for discrimination practice.
  - Avoid unrelated functional groups (stay within alkanes).

- Explanations (if used)
  - 1–2 sentences focusing on the applied rule (longest chain, lowest set, alphabetization).
  - Do not include full step-by-step (keeps flow fast).

## 8. UI/UX Behavior (current app with optional enhancements)

- Core interactions (as-is)
  - Render molecule on ChemDoodle canvas.
  - Show 4 choices; on click: lock answers, mark correct/incorrect, update score.
  - “Next” button/Enter key to advance; 1–4 keys for choices.

- Enhancements (non-breaking)
  - Show “Explain” text (if present) under feedback after selection.
  - Level filter: dropdown to play a specific level or mixed mode.
  - Progress bar per level (n/N).
  - LocalStorage for high score and unlocked levels.

- Accessibility
  - Ensure buttons have focus states; support keyboard 1–4, Enter.
  - Provide ARIA-live region for correctness feedback.
  - High-contrast color scheme toggle if feasible.

## 9. Progression and Scoring

- Scoring
  - +1 per correct; no penalty for wrong; per-level score tracked.
  - Final screen shows total and percent; optionally per-level stats.

- Progression
  - Levels unlocked after checkpoint pass (≥80–90%).
  - Mixed reviews available after Levels 3 and 5.

- Timing (optional)
  - Show timer per question; record median response for analytics.

## 10. Validation and QA

- JSON validation
  - Run a schema check (optional local script) to ensure fields and counts line integrity.
  - De-duplication: validate unique “name”+“mol” pairs; unique “id”.

- Rendering validation
  - Smoke test all items in the app once; confirm draw, label modes, and keyboard shortcuts.

- Pedagogical QA
  - Review by subject-matter expert (teacher) for correctness and distractor plausibility.
  - Pilot with 5–10 students; adjust distractors based on error patterns.

## 11. Content Roadmap

- Phase 1 (MVP, 2–3 days)
  - L1–L3: 50 items with explanations on 50%.
  - Add level and explain fields; basic schema checker.

- Phase 2 (1 week)
  - L4–L6: +50 items; add mixed reviews and checkpoints.
  - Implement level filter and progress bar.

- Phase 3 (1 week)
  - L7: +20 items; finalize 120-item bank.
  - Analytics: store per-level accuracy/time; refine items.

## 12. Risks and Mitigations

- Risk: MOL validation failures (counts mismatch).
  - Mitigation: add a pre-commit validation script; spot-check renders.

- Risk: Overly tricky distractors frustrate learners.
  - Mitigation: calibrate difficulty; provide “Explain” and level selection.

- Risk: Performance with large JSON.
  - Mitigation: lazy-load by level or split files; no-cache fetch settings remain.

## 13. Acceptance Criteria

- Content
  - ≥90 items covering Levels 1–7; each item has 3–4 plausible distractors.
  - Coverage: each rule (longest chain, lowest set, alphabetization, cyclo) has ≥15 targeted items.

- Functionality
  - App loads questions.json without errors; all items render.
  - Level filter (if implemented) loads appropriate subset.
  - Explanations display when present.

- Quality
  - Pilot accuracy improvement post-explanation ≥15% on repeated patterns.
  - No broken keyboard shortcuts or navigation.

## 14. Implementation Notes (for current app)

- Backward compatibility
  - The app can ignore unknown fields; ensure it still reads name/mol/distractors.
  - If adding level filter: filter array before shuffling and play loop.

- Data loading
  - Keep fetch with no-cache headers; suggest hosting via local server to avoid CORS.

- Label mode
  - Retain current label toggles (skeletal, terminal, all C) to aid student recognition.

## 15. Appendices

### 15.1 Level Name Lists (convert to MOLs)

- Level 2 (sample 10)
  - 2-methylpropane; 2-methylbutane; 3-methylpentane; 2-ethylbutane; 2-methylpentane; 3-ethylpentane; 2-ethylpentane; 4-methylhexane; 3-ethylhexane; 2-methylheptane.

- Level 3 (sample 10)
  - 2,2-dimethylpropane; 2,3-dimethylbutane; 3,3-dimethylpentane; 2,4-dimethylpentane; 3,4-dimethylhexane; 2-ethyl-3-methylbutane; 3-ethyl-2-methylpentane; 4-ethyl-2-methylhexane; 2-ethyl-4-methylhexane; 3-ethyl-4-methylhexane.

- Level 4 (sample 8)
  - 3-methylpentane; 4-ethylhexane; 2,5-dimethylhexane; 3,4-dimethylhexane; 2-ethyl-3-methylpentane; 3-ethyl-2-methylpentane; 2,4-dimethylpentane; 3,3-dimethylpentane.

- Level 5 (sample 8)
  - 2,2,4-trimethylpentane; 3-ethyl-2,4-dimethylhexane; 4-ethyl-3,3-dimethylhexane; 2,3,4-trimethylpentane; 2,2-dimethylhexane; 3,5-dimethylheptane; 2,2,5-trimethylhexane; 4-ethyl-2,2-dimethylheptane.

- Level 6 (sample 8)
  - cyclopentane; cyclohexane; methylcyclohexane; ethylcyclopentane; 1,2-dimethylcyclohexane; 1,3-dimethylcyclopentane; 1-ethyl-3-methylcyclohexane; 1,4-dimethylcyclohexane.

- Level 7 (sample 10)
  - 2,2-dimethylbutane; 3-ethyl-3-methylpentane; 2,2,4-trimethylpentane; 3-ethyl-2,2-dimethylhexane; 4-ethyl-2,3-dimethylhexane; 5-ethyl-3,3-dimethylheptane; 3-ethyl-4,4-dimethylhexane; methylcycloheptane; 1-ethyl-2,4-dimethylcyclohexane; 2-ethyl-3,3-dimethylpentane.

### 15.2 Content Checklist

- [ ] MOL block renders correctly and passes counts validation.
- [ ] Correct name follows IUPAC rules for the drawn structure.
- [ ] 3–4 distractors are plausible near-misses.
- [ ] Level and tags assigned appropriately.
- [ ] Optional explanation is concise and rule-focused.
