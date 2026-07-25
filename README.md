# FIMI × Ghost Note — Website Redesign Proposal

Single-file, password-gated proposal deck for the Food is Medicine Institute at Tufts University.
Horizontal click-through on screen, one-slide-per-page landscape PDF on export.

---

## Before you send

Three things must be handled. They are deliberately impossible to miss.

1. **Three red `FILL BEFORE SENDING` boxes** on the *Selected work* slide. Each needs one
   concrete outcome for that engagement. They print in red on purpose. If you can see a red
   box in the PDF, the PDF is not ready.
2. **The founders photo** on the *Meet Ghost Note* slide was not resolving. Confirm
   `Founders_01_Ghost-Note-1274-b_w-scaled.jpg` loads before you export.
3. **The findability footnote** currently reads "required in the RFP scope of work." Swap in
   the real section number once you have the RFP in front of you.

---

## Publishing

Deploy as a static site. No build step, no dependencies.

**GitHub Pages**
```
Settings → Pages → Source: Deploy from a branch → main → / (root)
```
Rename `FIMI_Ghost-Note_Proposal.html` to `index.html` if you want a clean URL.

**Netlify**
```
Drag the folder into the Netlify dashboard, or connect the repo.
Build command: none. Publish directory: /
```

---

## Exporting the PDF

Open the deck and click **Save as PDF** in the top bar. That button is the only supported
export path.

It preloads every image in the deck before opening the print dialog, then prints. There is a
10-second ceiling so a dead image URL cannot hang it. You do not need to click through the
slides first, and browser zoom does not matter.

In the Chrome dialog:

| Setting | Value |
| --- | --- |
| Destination | Save as PDF |
| Layout | Landscape (the `@page` rule sets 11 × 8.5in) |
| Scale | Default — **not** Custom |
| Background graphics | On |
| Headers and footers | Off |

**Expected output: 38 pages.** One page per slide, no blanks. If you get more, something
overflowed — see below.

---

## Fit check

Print slides are a hard `8.5in` with `overflow:hidden`. That guarantees one page per slide,
but it also means content that runs long gets clipped silently rather than spilling onto a
phantom page.

Run the checker after any copy edit:

```
open the deck with #fit on the URL     →  deck.html#fit
or call fitCheck() in the console
```

It measures every slide against the real print box (1056 × 816px, 56/44px padding) and tables
anything that would clip.

---

## Print architecture

Three rules do the work. Do not remove them.

**1. Desktop grids are re-declared inside `@media print`.**
The `@media(max-width:760px)` block was winning during export and collapsing every
multi-column grid into a single stack, which pushed images and bullets onto extra pages.
Every grid — `.split`, `.cols2`, `.statrow`, `.caps-cols`, `.team-cards`, `.mockwrap`,
`.cs-grid`, and the Gantt columns — is pinned for print.

**2. `.slide` is `height:8.5in` with `overflow:hidden`.**
It used to be `min-height`, which allowed growth and fragmentation. A fixed height plus
hidden overflow makes each slide an atomic box that cannot break across pages.

**3. `.slide:last-child` resets `page-break-after`.**
`page-break-after:always` on the final slide was emitting a trailing blank page.

If you add a slide, it inherits all three automatically. If you add a new **grid** component,
add its desktop columns to the print block or it will collapse on export.

---

## Structure

Everything is in one file: markup, styles, and script.

```
<style>   design tokens → components → mobile query → print query
<body>    38 <section class="slide"> elements in order
<script>  navigation, team modals, password gate, savePDF(), fitCheck()
```

Slides are read from the DOM at runtime. Nothing is hardcoded to a slide count, so you can
add or remove sections freely — the menu, counter, and progress bar all follow.

## Password gate

Client-side only. It keeps the deck off casual view; it is not security. Do not put anything
in here you would not email.

The field has a show/hide toggle (the eye icon, left of the arrow). It swaps the input between
`password` and `text`, preserves the caret position, and updates its own `aria-label` and
`aria-pressed`. Useful when you're reading a password off a phone or dictating it on a call.
Native browser reveal icons are suppressed so there's only ever one control.
