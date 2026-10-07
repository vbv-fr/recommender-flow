# Recommender flow prototype (Frinks Vision)

`index.html` in this folder is the current, working prototype and the source of truth. Edit it in place. Never rebuild it from scratch.

## What it prototypes
A redesign of recipe creation in Frinks Vision, an industrial computer-vision platform. The data recommender now takes its sample annotations from the Datasets & Annotations screen, instead of from separate upload steps.

**Old flow (the problem):** Step 2 Dataset Folders, then Step 3 Sample Images, then Step 4 Sample Labels. Users uploaded data twice: once only for the recommender, then again for training.

**New flow:**
1. **Wizard.** The recipe wizard has 2 steps and ends at Dataset Folders. Finish opens Datasets & Annotations filtered to that capture. A one-time banner explains what to do next.
2. **Annotation.** Users upload images per folder and time slot, and annotate them as usual. The recommender reads those same annotations.
3. **Readiness, per capture.** All three must hold:
   - Every class has at least 5 instances. These are training labels on saved images, in any folder or slot.
   - Every Absent-Case class has at least 3 "absent" labels.
   - Every negative class has at least 5 "negative" labels.
4. **Absent label.** A `<class> · absent` chip, shown after a divider in the annotation tool.
   - It appears ONLY on the first 5 images of the capture that sit in folders WITHOUT that class.
   - Images are ordered by dataset creation, then by image order.
   - It draws a dashed box where the class would normally sit.
5. **Negative label.** A `<neg> · negative` chip.
   - It appears ONLY on the first 5 images in folders CONTAINING the negative class.
   - It draws a dotted box.
   - A negative class is never offered as a normal class chip.
6. **Counter info icon.** Each absent/negative chip (with its n/target counter) has an ⓘ next to it; hover shows "Minimum 3 (absent) / 5 (negative) instances of this class need to be mapped for Data Recommender to run for this capture".
7. **Chip states.** Each chip shows n/target. n counts saved labels plus unsaved boxes on the current image.
   - Below target: orange dot and class-coloured border.
   - At target: grey, but still visible and usable.
8. **Absent and negative labels are recommender-only.**
   - They are stripped from training.
   - They never block Save & Next or training.
   - Copy Master Image Annotation skips them. It also skips classes not in the target folder, and keeps the target image's existing recommender-only boxes.
9. **No label-status panel** in the annotation tool. Readiness lives only in the Run Recommender popup.
10. **Run Recommender button.** It sits top right and is always clickable (no "N ready" pill). The popup is a table: checkbox | Capture | Status | Classes. Classes shows each class chip; when short, the chip gets a "· N more" suffix, and short absent/negative labels follow as dashed chips ("C · absent · 3 more"). Status: Ready to run, Outdated + reason, Missing classes (no Annotate link), Running, Up to date. Only Ready and Outdated rows can be ticked; nothing is pre-ticked; the header checkbox selects all. "Run Recommender" (footer button) is disabled until a box is ticked and runs the standard recommender. Runs are async: minutes in reality, 8 s here (override with `?run=N`). There is no cancel.
   - **Advanced Mode.** When 2+ ticked captures include at least one pair sharing a class, a "Setup Advanced Mode" button (with an ⓘ tooltip: "Gives optimised recommendations by combining captures with overlapping classes") appears left of Run Recommender. It opens a second step: captures are pre-grouped by shared classes (connected components; a capture with no overlap is its own group). × unassigns a capture into an "Unassigned" tray; an "Assign to…" dropdown puts it into an existing group or a new one. Empty groups vanish. A group whose captures don't all overlap shows an amber "Some captures share no classes" note (not blocking). "Run Advanced Mode" appears only when nothing is unassigned.
11. **Recommended column, per folder.** It shows one of:
    - "—" when not run
    - a spinner with "Running"
    - the number
    - the number greyed and struck through, with an "Outdated" tag

    A capture becomes Outdated when folders are added or removed, or class config changes. If the tree changes during a run, the result lands as Outdated. Adding images or annotations does NOT outdate it. The Annotated column is unchanged.
12. **Negative-class note.** When any class has a negative class set, Define Class Properties shows an amber Note (above "Define Mutually Exclusive Classes") naming the negative class(es) and stating the training rule: a module with that class as negative can't be trained in the same job as a module where it is normal.
13. **Step 2 guards.** You can't remove:
    - a folder that has uploaded images
    - the only folder without an absent class
    - the only folder with a negative class

**Demo data (5 captures, all variant 1):**
- capture 1: A, B (ready). capture 2: A, B, C (ready). capture 5: A, B, C, the live one (A=odc1, B=odc2, C=odc3 internally). C is marked Absent; H (odc4) is the negative class of B. Folders "A & B & C", "A & B", "A & H & C". It starts not ready; use "Fill capture 5 to ready".
- capture 3: D, E, F (ready). capture 4: E, F, G (Outdated).
- Overlaps: 1, 2 and 5 share A, B; 3↔4 share E, F. 1+3 or 2+4 have none.

**AI Model Training (prototype control).** Replica of frinksvision-frontend `src/modules/project/ai-training-new` (model list page + `BuildNTrainDrawer`: Outfit font, #ff4301, 5-step timeline, step 1 Model Name with Module Type/prefixed name/description + the real validation messages, step 2 Modules & Classes table: checkbox | Variant | Capture | Module | Model Type | Search Area | Classes; ClassChip colours use the same hash→HSL as `ClassChip.tsx`). Only "Object Detection & Classification" lists modules:
- capture 3 · ODC-1: D, E, F
- capture 4 · ODC-2: F
- capture 4 · ODC-3: E, G, with F as a NEGATIVE class (not shown as a chip; it only drives the conflict rule)
- Rule: one training job can't hold a class as both normal and negative. Selecting ODC-1/ODC-2 greys out ODC-3 and vice versa; the greyed checkbox shows a hover tooltip (real Tooltip styling) naming the class and the conflicting module(s). Steps 3–5 are not built; Next on step 2 shows a toast.

**Prototype controls bar:** Fill capture 5 to ready, AI Model Training (and Datasets & Annotations to return), Auto-label this image, Reset.

**Style:** Frinks orange #FF4A00 table headers, Quicksand font, class-coloured chips.

## How index.html is built (read before editing)
- `<template id="dc-template">` holds all markup. It uses:
  - `{{path}}` holes, which are dotted lookups only, never expressions
  - `<sc-if value="{{x}}">` for conditionals
  - `<sc-for list="{{items}}" as="item">` for loops; `{{$index}}` is in scope
  - `onClick="{{handler}}"`-style event attributes
- A ~60-line interpreter (the second `<script>`) renders the template with inlined Preact (the first `<script>`).
- `class Component extends DCLogic` (the last `<script>`) holds all state and logic. `renderVals()` returns everything the template reads, including handlers and per-item precomputed styles.
- To change UI, edit the template markup and compute values in `renderVals()`. Keep it one file, with no build step and no external JS. Google Fonts is the only external request.

## Hosting
- GitHub Pages repo: `vbv-fr/recommender-flow`, branch `main`, served from the repo root.
- Live at https://vbv-fr.github.io/recommender-flow/
- After any change, commit `index.html`, push to `main`, and confirm the live URL loads (Pages takes about a minute).
