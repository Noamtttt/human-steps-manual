# Using real mascot art instead of the hand-drawn SVG

Only relevant if real image files for the mascot exist. The current design uses the
hand-drawn inline SVG in `SKILL.md` — this file only applies if that changes.

## If the user shares real pose images

The user may have a reference character (e.g. a 3D-cartoon-style guy) and share pose
variations over time (fresh bagel, half-eaten, pointing left/right/down,
celebrating, etc).

- Store them under `.claude/skills/human-steps-manual/assets/bagel-guy/`, one file
  per pose (e.g. `default.png`, `point-left.png`, `celebrating.png`,
  `half-bagel.png`). Ask the user to drop the file at that path (or share a
  reachable URL) — there's no tool in this environment that pulls a
  pasted-in-chat image onto disk directly.
- In the widget, embed the chosen pose as a `data:` URI (base64-encode the file
  contents) rather than a bare local path — the widget iframe cannot resolve
  local filesystem paths.
- If a pose file for the exact gesture you need doesn't exist yet, fall back to
  the closest available pose rather than stretching/rotating the art in ways
  that would look broken, and mention to the user which additional pose would
  help.
- Once real art exists for a pose, prefer it over the hand-drawn inline SVG for
  that pose — the SVG stays only as the fallback for poses with no real art yet.

## Image generation

This environment has no direct Gemini/image-gen connector. The Figma MCP server
exposes Weave tools (`weave_list_tools`, `weave_run_tool`) which *may* include an
image-gen workflow, but using it requires the user to first link their Figma
account to Weave at `https://app.weavy.ai/settings?section=profile` (an
account/auth action — don't do this for them, ask them to do it) — after that,
check `weave_list_tools` for a suitable recipe before assuming one exists.

## Token cost for real images (only applies once real art is in use)

Real mascot art can quietly balloon a single tool call — don't pay that cost more
than once per widget:

- **Embed the mascot image's data URI exactly once**, in a single `<style>` rule
  (e.g. `.mascot-img{background-image:url(data:...);background-size:contain}`),
  then apply that class to every panel that needs the mascot. Never paste the
  same base64 string into multiple `<img src="data:...">` tags in one widget —
  that multiplies the payload by panel count for zero visual gain.
- **Keep the source image small before encoding**: resize to roughly 120–200px on
  the long edge and compress (e.g. `sips -Z 160 -s formatOptions 55 in.png --out
  out.jpg`) before base64-ing it. Check the encoded length and re-compress
  smaller if it's not already a few KB.
- **When generating the data URI, don't view it more than once.** Write it
  straight into the file/variable you'll use to build the widget call in one
  Bash step — don't `Read`/`cat` it to inspect it and then paste it again
  separately; each viewing round-trips the full string through the model's
  context for no benefit.
