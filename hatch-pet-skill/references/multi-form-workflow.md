# Multi-form pet workflow

## Contents

1. Mode contract
2. State-to-form mapping
3. Designed-form priority
4. Generation and assembly
5. Cache-safe revisions
6. Acceptance checks

## Mode contract

Use multi-form mode only when the user asks for distinct transformations or state-specific equipment. Keep species, face, markings, palette, body proportions, collar/core identity, and rendering style invariant. Change pose, attached props, energy state, expression, and silhouette accents only as requested.

Before image generation, write a form matrix with one row per app state. Give each form a short stable id such as `base`, `thinking-console`, `parallel-compute`, `ready-evolved`, or `failed-glitch`.

## State-to-form mapping

The Codex atlas uses fixed rows:

| Row | State | Runtime meaning | Multi-form guidance |
| --- | --- | --- | --- |
| 0 | idle | long-running fallback loop | signature designed form; never an unwanted generic default |
| 1 | running-right | pet dragged right | compact mobile form facing right |
| 2 | running-left | pet dragged left | compact mobile form facing left |
| 3 | waving | greeting | friendly signature or ready form |
| 4 | jumping | pointer-hover jump | signature or powered form with body-position motion |
| 5 | failed | task failure | attached glitch/error form; keep identity readable |
| 6 | waiting | needs user input | expectant/questioning form, not “thinking” |
| 7 | running | Codex is thinking/working | computer, scanner, typing, processing, or focused work form |
| 8 | review | result ready | success, evolved, celebratory, or review-ready form |

Do not confuse `waiting` with thinking: runtime `waiting` means **Needs input**. Runtime `running` is the Thinking/Working state. Runtime `review` is Ready/Completed.

## Designed-form priority

The app plays non-idle states for a limited number of cycles, then appends row 0 and loops idle. Therefore row 0 controls what users see most of the time.

- Put an approved designed signature form in idle.
- Never leave idle as an obsolete generic default when the user asks to prioritize designed forms.
- When explicitly requested, idle may rotate 2–3 identity-preserving approved forms. Keep transitions readable and avoid rapid size or silhouette popping.
- If the user wants strict state semantics, use one signature idle form and reserve transformations for rows 5–8.
- Do not use a visually unrelated base or placeholder in any used row.

For a three-form workflow, a practical mapping is:

- idle: signature form, or slow `thinking → working → ready` rotation when requested
- waiting: multi-panel expectant form
- running: computer/typing form with code reflected in eyes
- review: illuminated evolved success form
- failed: dimmed or glitch version of the computer form
- movement/wave/jump: signature, working, or ready forms chosen for clean silhouettes

## Generation and assembly

1. Define canonical identity invariants and the form matrix.
2. Generate a canonical base reference.
3. Generate each distinct form as its own grounded reference on a removable chroma background.
4. Generate every animation row using its assigned form reference plus the canonical base.
5. Keep effects attached to the pet. Avoid detached UI fragments that become separate sprite components.
6. Assemble the fixed 8×9 atlas at 192×208 per cell.
7. Leave every unused cell fully transparent.
8. Inspect both the contact sheet and motion previews. Reject form drift, frame popping, clipped props, and state/form mismatches.

Do not create multi-form rows by merely scaling, flipping, or tiling a single master. Each semantically distinct row must have generated visual grounding. Deterministic transforms remain limited to safe atlas processing and an explicitly approved left/right mirror.

## Cache-safe revisions

Custom pet data may remain cached after a same-path atlas replacement. For every installed revision:

1. Increment the filename: `spritesheet-v2.webp`, `spritesheet-v3.webp`, and so on.
2. Update `pet.json.spritesheetPath` to that exact filename.
3. Copy manifest and atlas together.
4. Verify the installed file hash matches the staged file hash.
5. Refresh custom pets, then reselect the pet once if the overlay still holds the previous data URL.

Prefer WebP with alpha to reduce data-URL size and first-wake latency. Keep a PNG only as a build/debug artifact; never let `pet.json` ambiguously reference an obsolete file.

## Acceptance checks

- `pet.json` resolves to an existing versioned atlas.
- Atlas is exactly 1536×1872 and alpha-capable.
- Row 7 visibly contains the requested thinking/working form.
- Row 6 visibly communicates needs-input rather than thinking.
- Row 8 visibly contains the ready/success form.
- Row 0 contains the intended long-lived signature or approved form rotation.
- No used cell contains an obsolete generic default.
- Unused cells are fully transparent.
- Installed and staged atlas hashes match.
- Contact sheet labels reflect runtime meanings, not guessed meanings.
