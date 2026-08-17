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

Consistent visual language so different scenarios (auth vs API key vs paywall) still
feel like the same "manual" — only icons/copy change. Keep it calm and legible first,
charming second — the mascot and copy carry the personality, the card chrome should
not fight for attention.

- **Panel wrapper**: `background:var(--surface-1)`, `border:0.5px solid var(--border)`,
  `border-radius:12px`, `padding:1rem`. **No rotation, no thick borders** — those read
  as sloppy rather than hand-made. A small step-number badge sits inline at the top of
  the card's content (not absolutely positioned/overlapping the corner).
- **Mockup elements inside a panel** (generic, stylized — never a real screenshot of
  the user's actual third-party site): a small "browser chrome" bar (three dots + a
  fake URL pill), an input field outline, a button pill, a toggle, a key/lock/shield
  icon (Tabler outline) — built from plain CSS/SVG shapes, not real UI captures.
- **Warning/sensitive panel**: same card shape, `background:var(--bg-warning)`,
  `border:0.5px solid var(--border-warning)` — no separate thick border treatment,
  just the role-token fill + border so it still reads as "part of the set."
- **Final panel**: a "done" state — checkmark, mascot in its happy/celebrating pose.
- One consistent card skin per manual: either plain `--surface-1` cards, or the
  wood-toned skin (below) — don't mix both styles in the same manual.

## The mascot

A small recurring cartoon character — round body, big simple eyes, holding a bagel —
appears in every panel to point at the relevant mockup element and crack a short,
dry one-liner in a speech bubble. This carries the *fun* tone; it never carries
safety-critical copy (that stays as a plain callout per the panel grammar above).

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

### Real mascot art ("bagel guy")

The user has a reference character — a 3D-cartoon-style guy with a mustache, green
shirt, holding a bagel — and will share more pose variations over time (fresh bagel,
half-eaten, pointing left/right/down, celebrating, etc). When real image files exist:

- Store them under `.claude/skills/human-steps-manual/assets/bagel-guy/`, one file per
  pose (e.g. `default.png`, `point-left.png`, `celebrating.png`, `half-bagel.png`).
  Ask the user to drop the file at that path (or share a reachable URL) — there's no
  tool in this environment that pulls a pasted-in-chat image onto disk directly.
- In the widget, embed the chosen pose as a `data:` URI (base64-encode the file
  contents) rather than a bare local path — the widget iframe cannot resolve local
  filesystem paths. Keep the source image reasonably small (compress/resize to roughly
  200–300px tall) so the data URI doesn't bloat the widget payload.
- If a pose file for the exact gesture you need doesn't exist yet, fall back to the
  closest available pose rather than stretching/rotating the art in ways that would
  look broken, and mention to the user which additional pose would help.
- Once real art exists for a pose, prefer it over the hand-drawn inline SVG above for
  that pose — the SVG stays only as the fallback for poses with no real art yet, or
  for sessions where no image assets have been supplied.
- Image generation: this environment has no direct Gemini/image-gen connector. The
  Figma MCP server exposes Weave tools (`weave_list_tools`, `weave_run_tool`) which
  *may* include an image-gen workflow, but using it requires the user to first link
  their Figma account to Weave at `https://app.weavy.ai/settings?section=profile`
  (an account/auth action — don't do this for them, ask them to do it) — after that,
  check `weave_list_tools` for a suitable recipe before assuming one exists.

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

Real mascot art and multi-panel layouts can quietly balloon a single tool call. Don't
pay that cost more than once per widget:

- **Embed the mascot image's data URI exactly once**, in a single `<style>` rule
  (e.g. `.mascot-img{background-image:url(data:...);background-size:contain}`), then
  apply that class to every panel that needs the mascot. Never paste the same base64
  string into multiple `<img src="data:...">` tags in one widget — that multiplies
  the payload by panel count for zero visual gain.
- **Keep the source image small before encoding**: resize to roughly 120–200px on
  the long edge and compress (e.g. `sips -Z 160 -s formatOptions 55 in.png --out
  out.jpg`) before base64-ing it. Check the encoded length and re-compress smaller if
  it's not already a few KB.
- **When generating the data URI, don't view it more than once.** Write it straight
  into the file/variable you'll use to build the widget call in one Bash step —
  don't `Read`/`cat` it to inspect it and then paste it again separately; each
  viewing round-trips the full string through the model's context for no benefit.
- For panels using the hand-drawn SVG fallback (no real art yet), keep the path data
  as terse as the shape allows — it's cheap already, but repeated per panel it adds
  up on long manuals (8+ steps).
- Don't re-render the same widget multiple times while iterating/testing — sanity
  check the HTML logic first, render once.

## Style notes

- Warm, friendly palette — panels can optionally use a wood-toned card (e.g.
  `background:#E8D9C3` light / `#3E2F23` dark via a `prefers-color-scheme: dark`
  override, `border:1px solid #8B5E3C`) instead of plain `--surface-1`, for an
  IKEA-manual-booklet feel. Keep the border thin (1px) and skip rotation, same as the
  plain-card skin — the wood tone alone carries the "little illustrated booklet"
  feeling; piling on thick borders and tilt reads as messy, not charming.
- Keep panels compact enough that 3-5 fit on screen without heavy scrolling; wrap to
  multiple rows for longer sequences.
- Background transparent, no outer page padding (per the visualize widget's own
  convention) so it composes cleanly in the side panel.
- Font: default is `--font-sans` (the app's normal typeface) for all panel copy —
  keep it. `--font-voice` (serif) is reserved for editorial/quote moments elsewhere
  in the app, not this skill's UI-ish panels. `--font-mono` is fine for literal values
  the user copies (a flag name, a command) so they read unambiguously as "type this
  exactly." Don't introduce other fonts — only these three tokens are available.

## Aligning with the visualize widget's design system

Call `mcp__visualize__read_me` before every use — its rules are mandatory and take
precedence over anything here if they conflict. In particular:
- No gradients, drop-shadows, blur, or glow — panel borders and rotation transforms
  are fine (not shadows), but keep fills flat.
- Colors: use CSS variables (`var(--surface-1)`, `var(--text-primary)`,
  `var(--bg-warning)`, `var(--border-warning)`, etc.) so panels work in dark mode,
  not hardcoded hex — pick 1-2 accent ramps max (e.g. amber for warmth, red/amber
  role tokens for the warning panel) rather than a rainbow per panel.
- Functional icons (lock, key, mail, check) should be Tabler outline webfont
  (`<i class="ti ti-lock">`), already loaded — don't hand-draw those. The **mascot**
  itself is the one deliberate exception: a small bespoke illustrative SVG character
  is the whole point (personality + the bagel gag), not a UI icon.
- Sentence case for all copy, including the mascot's speech-bubble one-liners.
- No `<style>` blocks over ~15 lines; prefer inline `style="..."`.
