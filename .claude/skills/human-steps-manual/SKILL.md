---
name: human-steps-manual
description: Turn a multi-step manual task the USER (not Claude) must do by hand into a fun, illustrated IKEA-manual-style visual instead of a wall of text. Triggers on authentication/login flows, retrieving or pasting API keys/credentials, paywall or billing setup, account/service configuration, granting OAuth/app permissions, or any other point where Claude would otherwise say "now you do X, then Y, then Z" across 3+ non-trivial manual steps. Do NOT trigger for a single one-line ask ("please run this command"), for steps Claude can do itself, or when the user has asked for terse/plain text output.
---

# Human Steps Manual

Renders a human-intervention task as a comic-panel, IKEA-manual-style visual with a
recurring bagel-eating mascot pointing at each step, using the `visualize` MCP tools
already available in this environment. Replaces a wall-of-text numbered list.

## When this fires

Any moment where you (Claude) are about to hand the user a multi-step manual task you
cannot do yourself — logging into a third-party site, generating/copying an API key,
setting up billing/paywall, granting OAuth scopes, clicking through a confusing settings
UI. Skip it for single-step asks, steps you can automate, or when the user has asked for
plain/terse output.

## Do it, don't describe it

The user does not want to read instructions for how to get somewhere — minimize
reading, maximize action:
- If the destination is a **local file/folder** (a settings file, a downloads folder,
  a generated report), **open it directly** with `open <path>` (Bash) so it pops up
  on their screen — don't just print the path and tell them to navigate there.
- If the destination is a **URL** and the environment has a browser tool available,
  consider navigating to it directly rather than only handing back a link, when doing
  so clearly serves the task (see the tool-use safety rules on this — anything past
  "open a page for them to look at" like submitting forms/logging in still needs the
  user, since only they hold the credentials).
- Reserve typed-out step-by-step prose for the rare case where nothing can be opened
  for the user (e.g. a physical action, or a page only they can reach because it needs
  their own login).
- The visual widget is still the default explainer for *what's about to happen* and
  *why* — but pair it with actually opening the thing whenever that's possible, so the
  user lands on the right screen instead of hunting for it.

## How to build it

1. Break the task into **3–7 discrete steps**. If there are more, group into stages
   (e.g. "Stage 1: Create the key", "Stage 2: Wire it into the app").
2. Call `mcp__visualize__read_me` with `modules: ["mockup", "interactive"]` once per
   session before the first `show_widget` call.
3. Call `mcp__visualize__show_widget` with an HTML widget built from the panel grammar
   below — one comic panel per step, laid out in a wrapping grid/flow.
4. **Every panel that has a real destination gets a real link.** If the step happens
   on a knowable page (a service's dashboard, settings screen, docs page), add a
   clickable `<a href="https://...">Open dashboard</a>`-style link inside that panel,
   pointing at the actual URL — not a placeholder, not just "go to the website" in
   prose. Use the exact URL if you know it (e.g. a docs link the user already gave
   you, or a well-known settings deep link like `https://dashboard.stripe.com/apikeys`);
   if unsure of the exact deep link, link to the closest known page (e.g. the
   service's dashboard root) rather than guessing a URL that might 404. Links render
   as normal `<a href>` in the widget — clicks go through the host's link-confirmation
   dialog, so no extra confirmation needed here.
5. Keep the chat reply itself to 1–2 sentences pointing at the visual. Any exact
   copy-pasteable value (a flag name, a literal command, a value with no clickable
   destination) goes as **plain text below the widget** — never bury a string the
   user must copy accurately only inside the graphic.
6. **Fallback:** if `show_widget` fails or isn't available, degrade to a normal
   numbered-list text reply, with the same real links inline as markdown. Don't block
   the task on the visual.
7. Never put secrets (real API keys, tokens, passwords) into the widget HTML — use
   placeholder text like `sk-••••••••` even if you already have the real value in
   context.

## Panel grammar (reuse across every manual)

**IKEA-instruction-booklet parody.** Cards are black line-art on a plain surface,
with exactly **one accent color** (the mascot's own amber fill, `#F4C177` — no new
color introduced) used sparingly for arrows/highlights. No per-step background
color, no warm multi-tone cards — color is reserved for the accent arrow and the
mascot art only. Consistent visual language so different scenarios (auth vs API key
vs paywall) still feel like the same "manual" — only icons/copy change.

- **Panel wrapper**: `background:var(--surface-1)`, `border:1.5px solid
  var(--text-primary)` (bold, dark outline — deliberately heavier than a normal
  0.5px hairline, reads as a booklet line-drawing), `border-radius:12px` (uniform,
  no uneven/per-corner radius), `padding:1rem`. No rotation.
- **Step badge**: a plain circle, no fill — `border:1.5px solid var(--text-primary)`,
  transparent center, bold step number set in DynaPuff (see Typography below)
  inside it. Not a colored icon badge — real IKEA manuals number steps plainly.
- **The literal arrow**: a small inline SVG line-arrow (stroke only, no fill,
  `stroke:#D9A05B`, `stroke-width:4`, rounded caps/joins) pointing from context
  toward the field or element the user needs to act on. This replaces any
  highlight-ring approach — draw an actual arrow graphic, matching real IKEA
  manuals' visual language.
- **Mockup elements inside a panel** (generic, stylized — never a real screenshot of
  the user's actual third-party site): a small "browser chrome" bar (three dots + a
  fake URL pill), an input field outline (`border:1.5px solid var(--text-primary)`,
  transparent fill, matching the booklet line-art treatment), a button pill, a
  toggle, a key/lock/shield icon (Tabler outline) — built from plain CSS/SVG shapes,
  not real UI captures.
- **Warning/sensitive panel**: the one exception to the monochrome rule — still
  `background:var(--bg-warning)`, `border:0.5px solid var(--border-warning)` (the
  normal hairline weight, not the bold 1.5px line-art border), since that color
  needs to mean "danger" consistently, not fit the booklet aesthetic.
- **Final panel**: a "done" state — checkmark, mascot in its happy/celebrating pose,
  same bold-outline card treatment as any other step.

## The mascot

A small recurring cartoon character — round body, big simple eyes, holding a bagel —
appears in every panel to point at the relevant mockup element and crack a short,
dry one-liner in a speech bubble. This carries the *fun* tone; it never carries
safety-critical copy (that stays as a plain callout per the panel grammar above).
Its existing amber fill (`#F4C177`) is what defines the panel grammar's "one accent
color" — no separate mascot art needed for the IKEA-booklet card style.

Running gag: the bagel gets progressively eaten across panels (whole → half →
crumbs → gone) as the user moves through steps, finishing right as the last panel
hits "done".

Reusable inline SVG (drop this into the widget HTML, reuse verbatim per panel, only
swap the `data-pose` wrapper class and the bagel `<g>` bite-state):

```html
<svg class="mascot" data-pose="point-right" viewBox="0 0 100 100" width="72" height="72">
  <!-- body -->
  <circle cx="50" cy="55" r="30" fill="var(--mascot-body, #F4C177)" stroke="#2b2b2b" stroke-width="3"/>
  <!-- eyes -->
  <circle cx="40" cy="48" r="5" fill="#2b2b2b"/>
  <circle cx="62" cy="48" r="5" fill="#2b2b2b"/>
  <!-- smile -->
  <path d="M40 63 Q50 72 62 63" stroke="#2b2b2b" stroke-width="3" fill="none" stroke-linecap="round"/>
  <!-- pointing arm (flip via CSS transform for point-left pose) -->
  <path d="M78 60 L95 45" stroke="#2b2b2b" stroke-width="5" stroke-linecap="round"/>
  <!-- bagel in other hand: swap this <g> for bite-state (whole/half/crumbs/gone) -->
  <g class="bagel" data-bite="whole">
    <circle cx="22" cy="65" r="12" fill="#D9A05B" stroke="#2b2b2b" stroke-width="2.5"/>
    <circle cx="22" cy="65" r="4" fill="none" stroke="#2b2b2b" stroke-width="2.5"/>
  </g>
</svg>
```

Pose variants: `point-right`, `point-left` (mirror with `transform: scaleX(-1)` on the
arm group), `point-down` (rotate the arm path), `celebrating` (both arms up, no bagel —
used on the final panel). Bagel bite states, swap the `.bagel` group:
- `whole`: full ring as above
- `half`: same circle with a wedge cut (`<path>` overlay in the panel's background
  color covering half the ring)
- `crumbs`: replace the ring with 3-4 small dots
- `gone`: omit the `<g class="bagel">` entirely, mascot's hand is empty/happy

Speech bubble: a simple rounded-rect `<div>` with a small triangle "tail" pointing at
the mascot, short one-liner text, positioned near the mascot in each panel.

### Real mascot art

Current design uses the hand-drawn SVG above — that's the common case, no extra
reading needed. Only if the user has shared real pose image files for the mascot,
read `reference/real-mascot-art.md` first for how to store/embed/compress them.

## Jargon terms — floating bubble, not inline expansion

The user may not know what "repo", "token", "API key", "OAuth", "scope", "endpoint",
".env", "webhook", "deploy", "branch", "commit", "pull request", "SSH key", "CLI",
"terminal", "environment variable", "cache", "DNS", "SSL/HTTPS certificate", "2FA",
"session", "cookie", or "CDN" mean. Wrap any such term with a click-to-open floating
bubble — never an inline `<details>` block, which pushes the layout down and reads as
broken. The bubble floats above the word and doesn't reflow anything around it.

```html
<span class="jt">token<span class="jt-bubble">like a password, but for this service instead of your login</span></span>
```

```css
.jt{position:relative;display:inline-block;border-bottom:1px dotted var(--text-secondary);cursor:pointer}
.jt-bubble{position:absolute;left:0;bottom:calc(100% + 6px);background:var(--surface-2);border:0.5px solid var(--border-strong);border-radius:8px;padding:6px 10px;font-size:12px;color:var(--text-secondary);width:max-content;max-width:220px;display:none;z-index:20}
.jt.open .jt-bubble{display:block}
```

```html
<script>
document.querySelectorAll('.jt').forEach(function(el){
  el.addEventListener('click', function(e){
    e.stopPropagation();
    var wasOpen = el.classList.contains('open');
    document.querySelectorAll('.jt.open').forEach(function(o){o.classList.remove('open');});
    if(!wasOpen) el.classList.add('open');
  });
});
document.addEventListener('click', function(){
  document.querySelectorAll('.jt.open').forEach(function(o){o.classList.remove('open');});
});
</script>
```

- One short line, analogy over definition ("like a password, but for this service"
  beats "a credential used for authentication").
- Only wrap the *first* occurrence of a given term per widget.
- Don't wrap words the user already used themselves in this conversation.
- If in doubt whether a term needs it, err on the side of explaining.

### Log which terms actually get explained

Every time a widget explains a jargon term, append one line to
`.claude/skills/human-steps-manual/glossary-log.md` in this repo (create it if
missing) so the glossary improves from real usage over time instead of guesswork:

```
- term | scenario (e.g. "github token setup") | explanation used | YYYY-MM-DD
```

- Log the **term and the plain-language explanation you used** — never the actual
  secret/token/URL/screenshot content, never anything personally identifying about
  the user. The point is tracking which *concepts* people need explained, not what
  they're doing with them.
- Use the current date from context if available; omit the date rather than
  guessing/computing one if it isn't.
- This is a lightweight append, not a blocking step — skip it if it would meaningfully
  delay the actual task, but default to doing it.

## Screenshots for troubleshooting

One of the most useful things about this skill: if the user gets stuck or the real
page doesn't match what the manual describes, they can just **share a screenshot** of
their actual screen and you look at it directly — far faster than them describing
what they see. Mention this once, briefly, near the end of the manual (not per panel —
that's noise): something like "stuck at any point — screenshot it and I'll tell you
exactly what to click." When they do share one, treat it as normal image input: locate
where they are, tell them the next concrete action, don't ask them to describe it back
to you.

## Keep token cost down

This skill's cost (vs. a plain text reply) is almost entirely the widget generation
itself — keep it lean since the whole point is helping non-technical users without
making every manual slow or expensive:

- Keep the mascot SVG's path data as terse as the shape allows — it repeats per
  panel, so it adds up on long manuals (8+ steps).
- Don't re-render the same widget multiple times while iterating/testing — sanity
  check the HTML logic first, render once.
- If real mascot images are ever used instead of the SVG, see the token-cost
  guidance in `reference/real-mascot-art.md` — embedding images can be far more
  expensive than the hand-drawn SVG if not compressed first.

## Style notes

- Keep panels compact enough that 3-5 fit on screen without heavy scrolling; wrap to
  multiple rows for longer sequences.
- Background transparent, no outer page padding (per the visualize widget's own
  convention) so it composes cleanly in the side panel.
- **Typography — a deliberate, informed exception to the platform's font rule**:
  step titles use **DynaPuff** (weight 600), body copy uses **Nunito** (400/600) —
  both Google Fonts, SIL Open Font License, free for commercial use, no attribution
  required. Load via `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=DynaPuff:wght@600&family=Nunito:wght@400;600&display=swap">`
  (`fonts.googleapis.com`/`fonts.gstatic.com` are on the widget renderer's CDN
  allowlist). This intentionally breaks the visualize widget's general "only the
  three platform font tokens, so widgets blend into claude.ai" guidance — the
  tradeoff was surfaced to and chosen by the user for this skill specifically, for a
  more distinctly playful instruction-booklet feel. `--font-mono` is still used
  for literal copyable values (tokens, flags, commands) so they read unambiguously
  as "type this exactly."

## Aligning with the visualize widget's design system

Call `mcp__visualize__read_me` before every use — its rules are mandatory and take
precedence over anything here if they conflict. In particular:
- No gradients, drop-shadows, blur, or glow — panel borders and rotation transforms
  are fine (not shadows), but keep fills flat.
- Colors: use CSS variables (`var(--surface-1)`, `var(--text-primary)`,
  `var(--bg-warning)`, `var(--border-warning)`, etc.) so panels work in dark mode,
  not hardcoded hex, with one deliberate exception — the accent arrow's amber
  (`#D9A05B`/`#F4C177`, matching the mascot) is a fixed literal color, not a CSS
  variable, since it's meant to be the one consistent accent regardless of
  light/dark mode. The warning panel is the only other exception, using
  `--bg-warning`/`--border-warning` instead of the monochrome+amber system.
- Functional icons (lock, key, mail, check) should be Tabler outline webfont
  (`<i class="ti ti-lock">`), already loaded — don't hand-draw those. The **mascot**
  itself is the one deliberate exception: a small bespoke illustrative SVG character
  is the whole point (personality + the bagel gag), not a UI icon.
- Sentence case for all copy, including the mascot's speech-bubble one-liners.
- No `<style>` blocks over ~15 lines; prefer inline `style="..."`.
