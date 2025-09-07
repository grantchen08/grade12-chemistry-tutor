# Alkane Naming Quiz — UI Design Documentation (Post Phase 0–3)

Version: 1.1  
Owner: UI/Frontend  
Last updated: 2025-09-07  
Status: Phases 0–3 completed. Phases 4–5 skipped per request.

## 1) Overview

This document reflects the current user interface after implementing the first three phases of the improvement plan:
- Phase 0: Baseline mobile readiness
- Phase 1: Responsive canvas and DPR-aware rendering
- Phase 2: Accessibility and interaction polish
- Phase 3: Responsive layout refinements and typography

The app renders a molecule with ChemDoodle and asks the user to select the correct IUPAC name from multiple choices, with immediate feedback and score tracking.

Tech stack:
- HTML/CSS/JS (single page app)
- ChemDoodle ViewerCanvas (ChemDoodleWeb.js, ChemDoodleWeb.css)
- Question data: questions-level-1.json

## 2) Users and Goals

- Students learning organic nomenclature
- Instructors demonstrating examples
- Casual learners on phones/tablets

User goals:
- See molecule clearly on any device
- Make a quick, confident selection
- Receive clear, accessible feedback
- Track session score

## 3) Information Architecture & Layout

Structure:
- Header: Title + Score
- Canvas: Molecule viewer
- Choices: Multiple-choice answers
- Feedback: Correct/incorrect status
- Controls: Next button + Label mode select

DOM outline:
- .wrap
  - .header
    - h1
    - .score > #score
  - canvas#viewer
  - .choices#choices
  - .feedback#feedback (aria-live)
  - .controls
    - button#next
    - select#labelMode

Keyboard shortcuts:
- 1–4: select choices
- Enter: advance (Next/Play Again)

## 4) Visual Design

Theme:
- Dark UI with high-contrast accents
- System UI font stack for clarity

Typography:
- Title uses clamped size: clamp(18px, 3.5vw, 28px)
- Body/controls: 16px to avoid iOS zoom and improve legibility

Color accents:
- Hover/focus: blue highlight
- Correct/incorrect: green/red border and light fills
- Feedback text color shifts to reflect status

Spacing:
- Comfortable padding/margins on desktop
- Reduced margins on small screens to maximize space
- iOS safe-area insets respected for notched devices

## 5) Components

- Header
  - h1 title (responsive size)
  - Score display (Score: <span id="score">)
- Viewer
  - canvas#viewer with CSS width:100% and JS-managed drawing buffer
  - ChemDoodle viewer instance with tuned bond/atom styles
- Choices
  - .choices grid: auto-fit minmax(240px, 1fr)
  - .btn buttons with correct/incorrect states
  - ARIA: role="group" on container
- Feedback
  - #feedback with aria-live="polite" and role="status"
- Controls
  - #next: primary action; focused after answering
  - #labelMode: mode select with accessible label

## 6) Responsiveness

Implemented:
- Viewport meta tag: width=device-width, initial-scale=1, viewport-fit=cover
- Canvas is fluid in layout (width: 100%) and maintains aspect ratio via JS
- DevicePixelRatio-aware rendering for crisp visuals on high-DPI screens
- Choices grid adapts:
  - repeat(auto-fit, minmax(240px, 1fr)) for smart column count
  - Collapses to one column on narrow screens automatically
- Controls wrap with flex to prevent crowding
- Safe-area insets applied for iOS cutouts

## 7) Accessibility

Implemented:
- Feedback region is aria-live="polite" and role="status" for screen reader announcements
- Visible focus outlines for buttons/select/Next (keyboard users)
- Focus management: moves to Next/Play Again after answering or finishing
- Touch ergonomics: 16px base font size and increased button padding
- High-contrast state colors for correct/incorrect

Keyboard:
- 1–4 selects current options (if enabled)
- Enter triggers Next when visible

## 8) Behavior and Logic

- loadQuestion(): clears previous state, draws molecule, resizes canvas, injects options
- drawMol(): validates and parses MOL; scales and loads into ChemDoodle viewer
- sizeCanvas(): sizes canvas to container width, maintains aspect ratio, sets DPR-aware drawing buffers, calls viewer.resize() and repaint()
- checkAnswer(): locks choices, updates score/state, sets feedback, reveals and focuses Next
- end(): shows summary and switches Next to Play Again, moves focus
- label mode select: toggles ChemDoodle label display settings
- resize handling: debounced to reduce layout thrash

## 9) Implementation Summary by Phase

Phase 0 (Baseline mobile readiness):
- Added meta viewport
- Increased touch targets (button/select padding; 16px font)
- Tweaked container spacing and mobile margins

Phase 1 (Responsive canvas and crisp rendering):
- CSS: canvas#viewer { width: 100%; height: auto; display: block; }
- JS: sizeCanvas() computes width from container, maintains aspect ratio, applies DPR scaling, resizes ChemDoodle viewer, and repaints
- Debounced resize listener; called on startup and after each question renders

Phase 2 (Accessibility and interaction polish):
- aria-live="polite" + role="status" on feedback
- Visible focus styles for focus/focus-visible
- Focus moved to Next/Play Again after answering/finishing
- Minor ARIA on choices group

Phase 3 (Layout refinements and typography):
- Title font-size clamp for better scaling
- Choices grid uses auto-fit minmax(240px, 1fr) to adapt columns fluidly
- Safe-area insets for iOS
- Mobile spacing refinements for .wrap and .controls

## 10) Current Risks and Considerations

- ChemDoodle rendering:
  - Order of operations matters: set CSS size, adjust drawing buffer for DPR, viewer.resize(), then repaint.
- Very small devices:
  - If width < 280px, the minimum width guard could still produce cramped layouts; current min is 280px in JS.
- Font scaling/accessibility:
  - System-level text size increases may impact layout; current grid and wrapping should handle it, but QA on extreme settings is recommended if re-enabling full QA later.

## 11) Backlog and Deferred Items

Phases 4–5 have been skipped per request. If resumed later, consider:
- Phase 4: Progress indicator (Question X/Y), label mode persistence, reduced-motion hooks, optional light theme
- Phase 5: Cross-device QA, Lighthouse accessibility/performance passes, and contrast audits

## 12) Maintenance Notes

- If adjusting the canvas aspect ratio, update the initial width/height or compute from a desired ratio and verify viewer.resize() sequence.
- When changing min-width of choice buttons, update the minmax() value to align with desired columns on specific breakpoints.
- If adding more keyboard shortcuts, ensure focus and aria-live behavior remains predictable for screen readers.

## 13) Acceptance Criteria Met

- No horizontal scrolling on 320px devices
- Canvas fills container and remains sharp on Retina/HiDPI
- Choices adapt to screen size, with comfortable tap targets
- Feedback announced to assistive tech; visible focus rings for keyboard users
- Title scales gracefully across device sizes

## 14) Summary

Phases 0–3 delivered mobile readiness, responsive and crisp molecule rendering, visible and accessible interactions, and flexible grid/typography. The app is now comfortable to use on phones and desktops without additional zooming or panning and provides a more inclusive UX.

Suggested follow-up questions:
- Do you want a numeric progress indicator (“Question X of Y”) added next?
- Should we persist the label mode preference across sessions (localStorage)?
- Would you like a light theme option via prefers-color-scheme for bright environments?