# ECG Clinical Interpreter — Design Specification

Date: 2026-09-30
Status: Approved

## Product goal

Build a clinician-facing web application for systematic manual interpretation of 12-lead ECGs. It must support both non-linear dashboard review and a structured step-by-step workflow, calculate objective derived values, recognize coherent ECG patterns, explain likely electrophysiological/pathophysiological mechanisms, and surface concise clinical considerations prompted by the ECG.

The application is ECG-only. It does not ingest ECG images and does not require symptom, laboratory, medication, imaging, or demographic entry.

## Interaction model

The default screen is a dashboard containing all ECG domains. Each domain can be opened directly and has a status: Not assessed, Incomplete, Normal, Abnormal, or Complete.

A prominent Assess systematically action launches the same assessment in wizard mode. Dashboard and wizard are two views over the same state; switching modes never loses findings.

Assessment order:
1. Technical check
2. Rate
3. Rhythm
4. Electrical axis
5. P waves / atrial abnormalities
6. PR interval / AV conduction
7. QRS duration / intraventricular conduction
8. R-wave progression / Q waves
9. Voltage / hypertrophy
10. ST segments
11. T waves
12. QT / QTc
13. Other findings
14. Final interpretation

## Responsive layout

Desktop / large tablet:
- persistent left section navigation
- central active assessment module
- right live Interpretation so far panel
- dashboard uses a compact grid of domain cards

Phone:
- mobile is a first-class interface, not a shrunken desktop UI
- vertically stacked dashboard cards
- one focused wizard section at a time
- compact progress indicator
- Back / Next controls
- direct section jumping
- large touch targets
- no page-level horizontal clipping at 320 CSS px
- numeric inputs use numeric-friendly keyboards where possible

## Data and inference layers

Every statement belongs to one of three layers:

1. Clinician-entered findings: raw observations such as QRS 138 ms, P waves present, ST elevation in II/III/aVF.
2. Derived classifications: deterministic consequences such as QTcF, axis class, prolonged PR, wide QRS, Cornell voltage criterion.
3. Suggested higher-level interpretation: coherent pattern suggestions such as RBBB, LAFB, acute RV strain, Wellens-type morphology, or hyperkalaemic repolarization pattern.

Higher-level interpretations are reviewable and must not silently turn uncertain input into a definitive diagnosis.

## Input conventions

Prefer structured controls:
- segmented choices
- yes / no / uncertain
- normal / abnormal / not assessed
- present / absent / uncertain
- positive / equiphasic / negative
- anatomical lead groups
- numeric measurements

Free text is reserved for unusual morphology or clinician comments.

## Technical check

Capture:
- paper speed, default 25 mm/s
- calibration, default 10 mm/mV
- tracing quality adequate?
- artifact suspected?
- lead reversal suspected?

## Rate

Allow entry of heart rate or RR interval and derive the other when possible.

Capture regular, regularly irregular, or irregularly irregular rhythm.

## Rhythm

Structured findings include:
- P waves present
- one P before each QRS
- one QRS after each P
- PR constant
- sinus-compatible P morphology
- atrial activity regular
- dropped beats
- AV dissociation

Conditional branching should avoid irrelevant questions.

Potential suggestions include sinus rhythm, sinus bradycardia/tachycardia, AF, flutter, junctional rhythm, ectopic atrial rhythm, AV block patterns, ventricular rhythm, and pacing when supported.

## Electrical axis

Axis assessment is a major visual feature.

Show a large frontal-plane wheel using true lead orientations:
- I = 0°
- II = +60°
- III = +120°
- aVF = +90°
- aVL = -30°
- aVR = -150°

Each lead has Positive / Equiphasic / Negative QRS polarity controls. The compatible axis sector narrows as findings are entered. Equiphasic leads use the perpendicular-lead method where appropriate.

Output:
- estimated angle or range
- normal / left / right / extreme axis class
- highlighted compatible sector

The wheel must be fully usable on phones and have a non-visual control alternative.

## P waves / atrial abnormalities

Capture P duration, amplitude where relevant, lead II morphology, and V1 biphasic morphology / terminal negative component.

Derive or suggest left atrial, right atrial, or biatrial abnormality when supported.

## PR / AV conduction

Capture PR interval, dropped beats, progressive PR prolongation, fixed conduction ratios, and AV dissociation.

Derived or suggested patterns include short PR, first-degree AV block, second-degree AV block patterns, high-grade AV block, and complete AV block.

Clinical prompt examples:
- bifascicular disease + syncope: consider intermittent high-grade AV block
- conduction disease in older patients: consider degenerative conduction-system disease among other causes

## QRS / conduction

Capture QRS duration, V1/V2 morphology, lateral terminal forces, delta-wave / pre-excitation features, and pacing morphology.

Derived or suggested patterns include RBBB, LBBB, incomplete RBBB, nonspecific IVCD, LAFB, LPFB, bifascicular block, pre-excitation, and paced rhythm.

Criteria are expandable rather than permanently occupying the workspace.

## R-wave progression / Q waves

Capture predominant R/S by V1–V6, transition zone, poor R-wave progression, early transition, and pathological Q waves by lead.

Interpretation may consider prior infarction/necrosis patterns, cardiomyopathy, abnormal activation, and lead placement mimics.

## Voltage / hypertrophy

Support numerical entry for amplitudes needed for:
- Sokolow-Lyon
- Cornell voltage
- Cornell product when inputs permit
- structured RVH criteria

Voltage criteria alone should not be presented as definitive chamber hypertrophy.

## ST segments

Primary entry uses anatomical groups:
- Inferior: II, III, aVF
- Septal/anterior: V1–V4
- Lateral: I, aVL, V5–V6
- Additional: aVR and optional right-sided / posterior leads

Capture normal/elevation/depression, involved leads, magnitude where useful, morphology where useful, and reciprocal changes.

Pattern examples:
- inferior ST elevation + reciprocal aVL depression -> inferior STEMI pattern
- V1–V3 ST depression + dominant R waves -> consider posterior infarction
- diffuse ST elevation + PR depression -> consider pericarditis
- widespread ST depression + aVR elevation -> diffuse subendocardial ischemia / supply-demand imbalance without automatically labeling left-main disease

## T waves

Capture normal, inverted, biphasic, tall/peaked, flattened, and hyperacute-looking morphology with lead distribution.

Interpret in relation to QRS morphology to distinguish primary from secondary repolarization changes when possible.

Recognizable patterns may include Wellens-type morphology, RV strain, LVH strain, bundle-branch-related secondary changes, and hyperkalaemic repolarization changes.

## QT / QTc

Capture QT and RR or rate.

Calculate:
- Bazett
- Fridericia
- Framingham
- Hodges

Always show raw QT with corrected values. Where heart rate makes Bazett less useful, visually emphasize a more appropriate alternative without hiding the others.

## Other findings

Support compact entry for ectopy, pacing, low voltage, electrical alternans, Brugada-type morphology, U waves, dextrocardia / lead-reversal pattern, and unusual free-text findings.

## Pattern engine

Reason across combinations rather than emitting isolated comments.

Example combination:
- sinus tachycardia
- right-axis deviation
- RBBB
- right-precordial T-wave inversion
- S1Q3T3

Suggested combined interpretation:
- acute/right-ventricular strain pattern
- consider pulmonary embolism among causes of acute RV pressure overload

Low-specificity isolated findings should not trigger alarm-style conclusions. S1Q3T3 alone must not produce a PE conclusion.

## Pathophysiology and clinical prompts

For meaningful abnormalities, provide an expandable explanation with:
1. Finding
2. Electrical/structural mechanism
3. Common or important associations
4. Concise clinical prompts

Examples:
- Acute RV strain: consider pulmonary embolism, pulmonary hypertension, and other acute RV afterload causes. Prompt: With acute dyspnoea, hypoxaemia, pleuritic pain or unexplained tachycardia, consider pulmonary embolism.
- Bifascicular conduction disease: In unexplained syncope, consider intermittent high-grade AV block.
- LVH + strain: consider hypertension, aortic stenosis, and other structural causes of LV pressure load.
- Peaked T waves + progressive QRS widening: consider hyperkalaemia urgently.
- Electrical alternans + low voltage: consider a large pericardial effusion / tamponade physiology.

The user does not enter clinical context. Context appears only as conditional prompts generated from ECG findings.

## Uncertainty philosophy

This is a clinician-facing tool. Avoid repetitive disclaimers, generic correlate-clinically text, large medico-legal banners, and alarm styling for routine abnormalities.

Show uncertainty only when clinically useful:
- important mimics
- low-specificity patterns
- internally contradictory data
- morphology poorly captured by structured controls
- time-critical distinctions

Use concise terms such as compatible with, suggests, consider, and commonly associated with.

## Consistency checks

Flag contradictions without blocking progress.

Examples:
- No visible P waves + constant PR interval
- QRS 86 ms + complete LBBB selected
- manually selected normal axis + wheel-derived marked right-axis deviation

Warnings are compact and actionable.

## Live interpretation

Maintain a continuously updated Interpretation so far. Prefer replacing generic normal statements with relevant abnormalities rather than appending an exhaustive checklist.

## Final report

Final screen contains three editable/copyable sections:
- ECG interpretation
- Pathophysiology
- Clinical considerations prompted by this ECG

The report updates automatically when findings change.

## Review and reset

Before finalizing:
- show incomplete sections
- allow direct jump to incomplete sections
- show contradictions
- allow Not assessed where appropriate

A clear New ECG / reset action clears all entered and derived state after confirmation.

## Visual design

Use the approved clinical visual direction:
- clean light interface
- dark navy desktop navigation
- restrained accent color
- green for completed/normal
- red reserved for genuinely abnormal findings
- amber for incomplete / review
- dense but uncluttered desktop layout
- touch-first mobile controls
- vector-based responsive clinical diagrams

## Accessibility

- native controls where practical
- keyboard navigation
- visible focus states
- sufficient contrast
- status never encoded by color alone
- approximately 44 px minimum touch targets
- accessible alternative to axis-wheel interaction
- dynamic updates without excessive live-region chatter

## State and persistence

Initial release keeps one ECG assessment in local application state only. No account, patient identifier, or server-side persistence is required.

## Testing expectations

Test at minimum:
- dashboard and wizard share one state
- 320 px mobile layout and common phone widths
- desktop three-column layout
- axis polarity logic
- QTc formula calculations
- voltage calculations
- conditional branching
- consistency warnings
- acute RV strain pattern
- bundle branch blocks
- AV block patterns
- ischemic ST patterns
- hyperkalaemic morphology
- long-QT prompts
- final report copy behavior
- reset behavior
- keyboard navigation and touch targets

## Non-goals

- automated ECG image/PDF analysis
- OCR
- EHR integration
- symptom/history intake
- laboratory integration
- risk scores requiring non-ECG data
- cloud persistence/accounts

## Success criteria

A clinician can:
1. Start an ECG assessment on phone or desktop.
2. Work non-linearly from the dashboard or switch to a systematic wizard.
3. Enter all major ECG findings without excessive typing.
4. Use an interactive axis wheel to determine frontal QRS axis.
5. Receive correct objective calculations and coherent pattern suggestions.
6. See likely electrical/pathophysiological mechanisms.
7. Receive concise ECG-triggered clinical considerations such as PE in an RV-strain pattern or intermittent AV block in syncope with bifascicular disease.
8. Generate a concise, editable, copy-ready interpretation without repetitive disclaimers.
