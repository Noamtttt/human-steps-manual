# human-steps-manual: IKEA-instruction-booklet card style

## Context

After reverting an earlier attempted redesign (new mascot + timeline layout — see
`2026-08-17-mascot-pivot-design.md`, abandoned; the user preferred the original
bagel-mascot, plain-grid design), the user asked to explore card styling ideas
separately, to make panels "a bit more fun" without necessarily touching the mascot.

Explored via the brainstorming visual companion (four directions: icon badges,
progress trail, bubble-leads ordering, sticker corners) and several real-fidelity
`visualize` widget renders. Landed on a different, more specific direction than any
single one of those four: leaning fully into the skill's own name — a literal IKEA
instruction-booklet parody — rather than a warm colorful-card treatment.

## Decisions

### 1. Overall concept

Cards become black-line-art on a plain surface, with exactly **one accent color**
used sparingly for arrows/highlights — not the previous warm multi-tone per-panel
backgrounds. The accent color is the mascot's existing fill (`#F4C177`, amber) — no
new color introduced, ties the new card style back to art that already exists.

### 2. Panel wrapper

- `background: var(--surface-1)`
- `border: 1.5px solid var(--text-primary)` (bolder/darker than the previous
  `0.5px solid var(--border)` hairline — reads as a booklet line-drawing, not a soft
  UI card)
- `border-radius: 12px` — uniform, **not** the uneven per-corner radius explored
  during the (reverted) mascot pivot; that idea does not carry over here.
- No per-step background color cycling — every card uses the same neutral
  `--surface-1` fill regardless of step number. Color is reserved for the accent
  arrow and the mascot art only.

### 3. Step badge

Plain circle, no fill: `border: 1.5px solid var(--text-primary)`, transparent
center, bold step number set in DynaPuff inside it. Replaces both the earlier plain
number badge and the "colored icon badge" direction explored in the browser
mockups — authentic IKEA manuals number steps plainly, they don't use colored
category icons for this.

### 4. The literal arrow

A small inline SVG line-arrow (stroke only, no fill, `stroke: #D9A05B` — the accent
— `stroke-width: 4`, rounded caps/joins) points from context toward the field or
element the user needs to act on. This is new: earlier grammar used an outline
highlight ring for "click here"; the IKEA parody replaces that with a literal arrow
graphic instead, matching the reference manuals' visual language directly.

### 5. Mascot — unchanged

The original hand-drawn bagel mascot SVG (black `#2b2b2b` stroke, amber `#F4C177`
fill) carries over as-is from the pre-existing design, speech bubble included,
positioned at the bottom of the card as before. Its existing amber fill is what
defines the new "one accent color" rule — no new mascot art needed for this
direction.

### 6. Typography

- **Step titles**: DynaPuff (Google Fonts, SIL Open Font License, free for
  commercial use, no attribution required), weight 600.
- **Body copy**: Nunito (Google Fonts, same license terms), weights 400/regular and
  600 for light emphasis.
- **Literal copyable values** (tokens, flags): unchanged, `var(--font-mono)`.
- Both fonts load via `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=DynaPuff:wght@600&family=Nunito:wght@400;600&display=swap">` — `fonts.googleapis.com`/`fonts.gstatic.com` are on the
  widget renderer's CDN allowlist, so this is technically permitted.
- **Explicit, informed tradeoff**: this deliberately breaks the visualize widget's
  own style guidance ("don't introduce other fonts — only the three platform
  tokens"), which exists so widgets blend seamlessly into claude.ai's chat UI. The
  user chose this consciously, after being told about the tradeoff, in favor of a
  more distinctly playful/booklet feel. Only `human-steps-manual` panels are
  affected — this is not a platform-wide change.

### 7. Dropped ideas (explored, not used)

- **Progress trail strip** (a filling bar above the grid) — explicitly rejected by
  the user ("i don't want this progress trail").
- **Colored icon badges per step** and **per-step background color cycling** —
  superseded by the monochrome-plus-one-accent approach.
- **Speech-bubble-leads ordering** (mascot's line as the first thing read) — an
  intermediate preview used this ordering, but the final approved version reverted
  to title-first, mascot-and-bubble-at-bottom, matching the original grammar's
  panel order.
- **Sticker corner accent** — not selected, not carried forward.

## Open items for implementation

- **Warning/sensitive panel**: not yet re-previewed in this style. Existing
  principle (warning panels always use `--bg-warning`/`--border-warning`,
  overriding decorative styling, since that color needs to mean "danger"
  consistently) should still hold — needs a quick visual check during
  implementation that a warning-tinted fill still reads correctly against the new
  bold black border treatment.
- **Final "done" panel**: keeps the existing checkmark + mascot celebrating-pose SVG
  from the original grammar (unaffected by this change) — should get the same
  black-outline card treatment as any other step.
