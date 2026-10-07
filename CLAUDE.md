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
6. **Chip states.** Each chip shows n/target. n counts saved labels plus unsaved boxes on the current image.
   - Below target: orange dot and class-coloured border.
   - At target: grey, but still visible and usable.
7. **Absent and negative labels are recommender-only.**
   - They are stripped from training.
   - They never block Save & Next or training.
   - Copy Master Image Annotation skips them. It also skips classes not in the target folder, and keeps the target image's existing recommender-only boxes.
8. **No label-status panel** in the annotation tool. Readiness lives only in the Run Recommender popup.
9. **Run Recommender button.** It sits top right, is always clickable, and carries an "N ready" pill. The popup groups captures as:
   - Ready to run
   - Outdated, with a reason
   - Running
   - Not ready, with per-class deficit chips and an "Annotate →" link that opens the right image with marking already on
   - Up to date

   Ready and Outdated captures can be multi-selected; the button reads "Run for N captures". Runs are async: minutes in reality, 8 s here (override with `?run=N`). There is no cancel.
10. **Recommended column, per folder.** It shows one of:
    - "—" when not run
    - a spinner with "Running"
    - the number
    - the number greyed and struck through, with an "Outdated" tag

    A capture becomes Outdated when folders are added or removed, or class config changes. If the tree changes during a run, the result lands as Outdated. Adding images or annotations does NOT outdate it. The Annotated column is unchanged.
11. **Step 2 guards.** You can't remove:
    - a folder that has uploaded images
    - the only folder without an absent class
    - the only folder with a negative class

**Demo data (capture 2):**
- Classes: odc1, odc2, odc3.
- odc3 is marked Absent.
- odc4 is the negative class of odc2.
- Folders: "odc1 & odc2 & odc3", "odc1 & odc2", "odc1 & odc4 & odc3".
- Other captures show the other recommender states.

**Prototype controls bar:** Fill capture 2 to ready, Auto-label this image, Reset.

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
