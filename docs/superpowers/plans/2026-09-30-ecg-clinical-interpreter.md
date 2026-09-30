# ECG Clinical Interpreter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a responsive clinician-facing ECG interpretation web app with a shared dashboard/wizard state, interactive axis analysis, deterministic calculations, rule-based pattern suggestions, pathophysiology prompts, and a copy-ready final report.

**Architecture:** Client-side TypeScript/React. One typed ECG assessment state is shared by dashboard and wizard. Pure domain modules handle calculations, axis inference, consistency checks, pattern recognition, pathophysiology, clinical prompts, and report generation; the UI consumes those outputs rather than duplicating medical logic.

**Tech Stack:** React, TypeScript, Vite, Vitest, React Testing Library, Playwright, responsive CSS Grid/Flexbox, SVG.

**Spec:** `docs/superpowers/specs/2026-09-30-ecg-clinical-interpreter-design.md`

## Global constraints
- ECG-only workflow; no symptom/history/lab intake.
- Dashboard and wizard share exactly one state.
- Phone and desktop are first-class; 320 CSS px must not clip.
- Structured clinician findings are the source of truth.
- Derived values are deterministic and testable.
- Pattern suggestions are reviewable and never silently become definitive diagnoses.
- Clinical prompts are concise and clinician-facing; no repetitive disclaimer layer.
- No accounts, identifiers, server persistence, ECG image analysis, OCR, or EHR integration in v1.
- Exact medical criteria/thresholds must be documented in `docs/references/clinical-criteria.md` with authoritative sources and tests.

## Review focus
- Partial states never emit NaN, fake normality, or contradictory report text.
- Switching dashboard/wizard preserves all findings.
- Axis input may yield a range/indeterminate state; never fabricate precision.
- Contradictions are compact non-blocking warnings.
- Isolated low-specificity findings such as S1Q3T3 do not independently trigger a PE conclusion.

### Task 1 — App shell, assessment state, responsive navigation
Create Vite/React/TS scaffold, typed `AssessmentState`, reducer/context, section metadata, desktop shell, mobile shell, dashboard, and wizard.
Tests: shared state across views, navigation, reset, 320px mobile layout, desktop three-column layout.
Commit: `feat: add shared ECG assessment shell and navigation`

### Task 2 — Deterministic ECG calculations
Create pure modules for rate↔RR, interval classification, Bazett/Fridericia/Framingham/Hodges QTc, Sokolow-Lyon, Cornell voltage, Cornell product.
Create `docs/references/clinical-criteria.md` with exact formulas/thresholds and sources.
Tests: known numeric examples plus empty/zero/negative/extreme input handling.
Commit: `feat: add ECG calculation engine`

### Task 3 — Axis engine and interactive wheel
Create axis math/rules plus accessible responsive SVG wheel using true lead orientations:
I 0°, II +60°, III +120°, aVF +90°, aVL -30°, aVR -150°.
Inputs: Positive / Equiphasic / Negative for each lead.
Output: compatible sector(s), angle/range, category, and indeterminate/contradictory state.
Tests: quadrant narrowing, equiphasic/perpendicular logic, conflicting and underdetermined inputs, mobile controls.
Commit: `feat: add interactive ECG axis wheel`

### Task 4 — Structured ECG assessment sections
Build Technical, Rate, Rhythm, P-wave, AV conduction, QRS, R-wave/Q-wave, Hypertrophy, ST, T-wave, QT, and Other Findings modules.
Use reusable segmented controls, lead selectors, numeric fields, and conditional branching.
ST/T use anatomical lead groups and mobile substeps.
Tests: branching, numeric validation, keyboard/touch behavior, 44px touch targets.
Commit: `feat: add structured ECG assessment sections`

### Task 5 — Consistency checks and pattern engine
Create non-blocking contradiction rules plus composable pattern modules for rhythm, conduction, ischemia, strain, and electrolyte patterns.
Pattern outputs include evidence, label, concise confidence wording, and review state.
Tests include:
- RBBB/LBBB and fascicular patterns
- AV block patterns
- inferior/posterior ischemic patterns
- pericarditis pattern
- acute RV strain
- hyperkalaemic morphology
- long-QT prompt eligibility
- isolated S1Q3T3 must not trigger PE
Commit: `feat: add ECG consistency and pattern engine`

### Task 6 — Pathophysiology and clinical prompts
Map accepted/active patterns and salient findings to concise:
1. mechanism,
2. important associations,
3. ECG-triggered clinical prompts.
Required tests include:
- acute RV strain → PE/other RV-afterload causes
- bifascicular disease → “in syncope, consider intermittent high-grade AV block”
- LVH strain → pressure-load/structural causes
- hyperkalaemic pattern
- low voltage + electrical alternans → large pericardial effusion/tamponade physiology
No generic correlate-clinically/disclaimer boilerplate.
Commit: `feat: add ECG pathophysiology and clinical prompts`

### Task 7 — Live interpretation, review, final report
Create deterministic report builder and live “Interpretation so far”.
Final screen has editable/copyable:
- ECG interpretation
- Pathophysiology
- Clinical considerations prompted by this ECG
Review screen shows incomplete sections and contradictions with direct jumps.
Tests: normal concise report, abnormal findings replacing generic normals, no duplicate statements, copy behavior, incomplete review.
Commit: `feat: add live and final ECG interpretation reports`

### Task 8 — Accessibility, integration, release readiness
Add end-to-end workflow tests for:
- dashboard-first workflow
- wizard-first workflow
- switching modes mid-assessment
- axis interaction
- report generation
- reset
- keyboard-only navigation
- mobile/desktop responsive behavior
Finish visual polish, README, and production build verification.
Required commands: `npm test -- --run`, `npm run build`, Playwright suite.
Commit: `chore: verify ECG interpreter v1`

## Implementation rule for medical criteria
Before coding the clinical rules in Tasks 2, 5, and 6, verify the contemporary criteria against authoritative cardiology sources and record them in `docs/references/clinical-criteria.md`. UI components must consume domain outputs rather than contain medical thresholds themselves.
