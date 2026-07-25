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
4. **Paste your alert endpoint** (see Visit alerts below) if you want the open notifications.

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

**Expected output: 37 pages.** One page per slide, no blanks. If you get more, something
overflowed — see below.

---

## Visit alerts

Emails you when someone opens the deck, and again when they leave with a read-through summary.

**Setup, one time, about two minutes.** Sign up at formspree.io, create a form, set the
notification address to `alfred@ghostnoteagency.com`. Copy the endpoint it gives you and paste
it into `ALERT_ENDPOINT` near the top of the `<script>` block.

```js
const ALERT_ENDPOINT = 'https://formspree.io/f/xxxxxxxx';
```

Leave it empty and nothing sends. Nothing breaks either. Any endpoint that accepts a JSON POST
works — Zapier, Make, a Netlify function, your own webhook. Formspree is just the fastest.

**Email one, on open.** Fires the moment the password clears. Timestamp, their timezone,
whether they've opened it before, where they came from, screen size, device.

**Email two, on exit.** Fires when they close or background the tab, and only if they stayed
longer than eight seconds so accidental opens stay quiet.

| Field | What it tells you |
| --- | --- |
| `time_on_deck` | Total time with the deck open |
| `slides_reached` | Deepest slide, e.g. `31 of 37` |
| `exported_pdf` | Whether they hit Save as PDF |
| `most_time_on` | Top five slides by dwell time |
| `full_path` | The slides they hit, in order |

`most_time_on` is the useful one. If somebody sits on Investment for ninety seconds, that is a
different follow-up call than if they sit on the project page concept.

Worth knowing: this is client-side, so an ad blocker or a locked-down corporate browser can
stop it. Treat a silent inbox as inconclusive rather than as nobody having looked.

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

**Footnotes.** `.fn` is absolutely pinned to the bottom of the page on desktop and in print.
On mobile it drops into normal flow with a rule above it, because an absolutely positioned
element inside a scrolling slide floats up over the copy as you scroll. The print block
re-asserts the pinned version so the mobile rule can never reach the PDF.
If you add a slide, it inherits all three automatically. If you add a new **grid** component,
add its desktop columns to the print block or it will collapse on export.

---

## Favicon

Points at the Ghost Note mark on the live site:
`wp-content/uploads/2025/05/GN-White-350x350.png`.

It's the **white** mark, so it reads on dark browser chrome and can disappear against a light
tab strip. If there's a dark or full-colour variant on the site, swap the URL in the two `<link
rel="icon">` tags in the head. One line each, nothing else depends on it.

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
