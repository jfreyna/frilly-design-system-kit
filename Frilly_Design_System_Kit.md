# Try it yourself: a design system from your website, in minutes

*Fill in the three lines below, then copy-paste THIS ENTIRE DOCUMENT — your answers and everything under the line — into a new Claude or ChatGPT chat. That's it.*

## Your three inputs

1. **Your website:** ___________________________
2. **Two fonts** — headings + body (Google Fonts names work best,
   or write "keep our current fonts"): ___________________________
3. **A reference site you admire** — you're not copying it, you're pointing
   at a mood (when we built badgerstartup.com, ours was obama.org):
   ___________________________

**Claude (or your LLM of choice) will:** extract your real brand colors and type into a complete design system, show you a spec sheet, and build a sample homepage in the new direction. Don't like it? Change a font or the reference site and paste again — each variation takes minutes, so explore freely.

*Want the real thing — a full site built on a system like this, ready for the AI-agent web out of the box? That's what we do: Frilly, frillyconcierge.com.*

---

**Instructions for Claude:** The person above has given you their website, an optional font pairing, and a reference site. First, run the skill below against their website to extract its design system faithfully. Then create ONE redesign variation: apply their chosen fonts (if given) and let the reference site guide aesthetic decisions (spacing, weight, restraint) — label carried-over values vs. redesign values per the skill's rules. Present the spec sheet, then offer to build a sample homepage from the new tokens.

**Output files (always these names):** the spec sheet is `design-system.html`; the sample homepage is `index.html`. Make both fully self-contained — inline CSS, and inline SVG or CSS shapes for imagery, so they work with no other files (real webfont links are the only exception). If you have file accessn (Claude Code / Desktop), save both next to this document, plus an `img/` folder for any assets you generate. If you're in a chat without file access, deliver each as a downloadable file with those exact names and tell the person: "save both files into one folder, then double-click index.html."

# Design System from an Existing Site

Turn a live website (or a folder of HTML/CSS) into the beginnings of a Material Design-based design system. The output is ONE self-contained HTML file that is simultaneously a visual spec sheet a founder can approve and a token source an AI coding agent can build from.

Core principle: **extract, don't invent.** Every anchor value in the system must be traceable to something the site actually uses. The system organizes the brand; it does not redesign it.

## Step 1 — Mine the site for raw material

Fetch the homepage and main CSS (crawl sub-pages if styles differ). Mine in this priority order:

1. **CSS custom properties first.** Themes that define `--color-primary`,
   `--color-content-text`, `--font-family` etc. have already named their
   semantics — that mapping is ground truth. Grep for `--[a-z-]+:` and read the
   names, not just the values.
2. **Frequency-count hex values** across CSS + HTML (`#[0-9a-f]{6}`, lowercase,
   sorted by count). The brand colors surface at the top. **Discard third-party
   colors**: social-network brand hexes (Twitter blue, WhatsApp green, LinkedIn
   blue…), syntax-highlighter themes, and plugin palettes are noise that
   frequency-counting will happily surface.
3. **Classify what remains by usage semantics**, not by hue: primary (links,
   buttons, hover states), secondary (link hover, gradient endpoints), accent
   (overlays, highlights). Two or three brand colors is typical; more usually
   means you're looking at noise.
4. **Collect every gray in use, with its role**: heading color, body text,
   muted/faded text, borders, alternate section backgrounds, footer background.
   Record the exact hex of each — these become the neutral ramp (Step 3).
5. **Fonts**: check Google Fonts `<link>`s, `@font-face` blocks, and
   `font-family` declarations (ignore icon fonts). Record families, loaded
   weights, base font size/line-height (desktop AND mobile breakpoints), and
   per-heading sizes/weights/line-heights if the theme defines them.

## Step 2 — Build tonal ramps anchored at 600

Pin each brand color **verbatim** as the `600` step (per the Material convention for a saturated brand anchor). Generate the rest by mixing: tints toward white, shades toward black, using these fractions:

| Step | Mix | Fraction |
|------|-----|----------|
| 50   | white | 0.94 |
| 100  | white | 0.86 |
| 200  | white | 0.72 |
| 300  | white | 0.55 |
| 400  | white | 0.35 |
| 500  | white | 0.16 |
| 600  | — | anchor, verbatim |
| 700  | black | 0.18 |
| 800  | black | 0.34 |
| 900  | black | 0.50 |
| 950  | black | 0.66 |

Reference implementation (run it, don't eyeball it):

```python
def hex2rgb(h): h = h.lstrip('#'); return tuple(int(h[i:i+2], 16) for i in (0, 2, 4))
def rgb2hex(r): return '#%02x%02x%02x' % tuple(max(0, min(255, round(c))) for c in r)
def mix(c1, c2, t):
    a, b = hex2rgb(c1), hex2rgb(c2)
    return rgb2hex(tuple(a[i] + (b[i] - a[i]) * t for i in range(3)))

TINTS  = {50: .94, 100: .86, 200: .72, 300: .55, 400: .35, 500: .16}
SHADES = {700: .18, 800: .34, 900: .50, 950: .66}

def ramp(anchor):
    out = {k: mix(anchor, '#ffffff', t) for k, t in TINTS.items()}
    out[600] = anchor
    out.update({k: mix(anchor, '#000000', t) for k, t in SHADES.items()})
    return out
```

Build one ramp per brand color (`primary`, `secondary`, `accent`). White-mixing naturally desaturates tints and black-mixing deepens shades — close enough to Material's hand-tuned palettes for a working system.

## Step 3 — The neutral ramp: map, don't generate

**Do not compute the neutral ramp.** Instead, slot the site's real grays (collected in Step 1) onto the steps they're closest to, then fill only the gaps with sensible in-between values. Aim for every gray the site actually uses landing **exactly** on a step — e.g. `100 = #f5f5f5` (section bg), `200 = #e8e8e8` (borders), `500 = #999` (muted text), `700 = #333` (body), `900 = #1a1a1a` (headings), `950 = #000` (footer). This means migrating the site to tokens later requires zero rounding compromises, and each swatch can be annotated with its real-world role. If the site's grays are chaotic (a dozen near-identical values), consolidate to the nearest step and note the merges.

## Step 4 — Typography hierarchy

Document a complete scale, preserving the site's real values:

- **H1–H6**: size, line-height, weight per level. If the site uses fluid sizes
  (`max(3rem, 3vw)`), record the rem floor as the canonical token.
- **Body large / body / body small**: the site's base size + line-height is
  body-large or body; note the mobile base if it differs.
- **Caption** and **overline** (small, and small+bold+tracked+uppercase).
- Families: if the site uses one family, keep it; if you introduce a pairing
  later, split tokens into `--font-head` (headings, captions, UI) and
  `--font-body` (body copy) rather than overloading one variable.
- **Annotate every token in BOTH px and rem** (state the 16px root). Designers
  think in px, tokens ship in rem; showing both prevents translation errors.

## Step 5 — The deliverable: one self-contained HTML spec sheet

Produce a single `.html` file (no external assets except the real webfonts)
containing, in order:

1. **Masthead** — project name, one-paragraph provenance statement ("colors extracted from X, aqua #00b5bc pinned as primary-600, fonts …").
2. **Color section** — one swatch grid per ramp (all 11 steps, step number + hex on each swatch, dark/light label color chosen by luminance), with the 600 anchor visually marked (outline + ★) and a caption noting the *site usage* each ramp came from. For the neutral ramp, caption which real site roles land on which steps. Close with usage chips (primary/secondary/accent buttons, a tonal chip, a brand gradient) rendered from the tokens.
3. **Typography section** — one specimen row per token: left column meta (name, family, px / rem, line-height, weight, Tailwind utility), right column live specimen text at the real size in the real font.
4. **Tailwind setup section** — copy-paste blocks for BOTH Tailwind v4 (`@theme` CSS with `--color-*-50…950` and `--text-*` size/line-height variables) and v3 (`tailwind.config.js` `extend` with `colors`, `fontFamily`, `fontSize`), plus the exact font `<link>` snippet. Mark the 600 anchors with a `★ brand` comment and put px equivalents in comments.

Define the tokens once as CSS custom properties in the file's own `<style>` and style the sheet with them — the spec sheet dogfoods its own system. Render swatches with a small JS loop from a single ramp object so hex labels can never drift from the actual swatch color.

## Step 6 — Verify before presenting

- Open the file in a browser; screenshot it.
- Confirm the 600 swatches display the verbatim brand hex values.
- Confirm specimens render in the real fonts at the stated sizes (spot-check with computed styles: `getComputedStyle(el).fontSize/fontFamily`).
- Confirm both Tailwind blocks contain identical values to the swatch grids.

## Rules

- The 600 anchor is sacred: verbatim from the site, never "improved."
- Every value gets provenance — a reader should be able to trace any token back to where the site uses it.
- Redesign decisions (new sizes, new font pairings) may layer on later, but label them as redesign values vs. carried-over values so the sheet stays an honest record of both.
- The spec sheet is the single source of truth: later pages, components, and AI-agent builds copy tokens from it, never the other way around.
