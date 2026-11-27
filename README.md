# Arkaya Labs website

The site at arkaya-labs.com: plain static HTML on Vercel with no build step (`vercel.json`). `.vercelignore` keeps
`src/`, `tools/` and this file off the site. Footer: "Copyright 2026 the Arkaya Labs authors". Contact:
hello@arkaya-labs.com.

Four pages, each with its own painting: Home (Taktsang, Paro), Projects (Kailasa, Ellora), Research (Delphi) and
About (Chittorgarh). Every page has the sticky bar, a blurred copy of its painting fixed behind the content under a
paper wash, and the full painting at the bottom.

## Open it

Run `python3 -m http.server 8000` in this folder and open `http://localhost:8000`.

Opening `index.html` straight from disk also works, but browsers refuse to give `file://` images to WebGL, so the
closing paintings then fall back to a plain day/night crossfade (no breathing lamps, stars or wind). The paintings'
fade masks are inlined in `styles.css` as data: URIs for the same reason: from disk the browser refuses mask image
files, and a mask that fails to load hides the whole painting (take 30 had this bug).

## Day and night (from take 9)

- `theme.js` runs in `<head>` so the first frame is already the right palette. First visit follows the system
  setting (or the host page's theme); clicking the sun/moon remembers the choice under `ak-theme`, shared across
  pages and tabs, and it always wins over a host that sets its own theme. Where storage is blocked, the choice
  still carries from page to page in the same tab.
- Everything turns on one clock and one curve: `--dusk` (3.2 s) and `--ease` in `styles.css`.
- The brand mark and the seal swap to their dark-paper version at night.
- The closing painting on each page is a canvas (`site.js`): a WebGL pass crossfades day into night on the same
  --dusk tween. After dark the flames and lit windows breathe and a few stars twinkle. Nothing in the picture moves or
  bends; the lamp layer holds only the lamps themselves (see below), so lit walls and trees stay still. The two still images stay underneath, so with reduced motion or without WebGL the
  picture still crossfades.

## The seal and mission

The seal, "Intelligence should be owned, not rented." and the Sui juris motto live only on the About page, as the
centrepiece of its mission section. Since take 35 the seal is drawn by `tools/seal.py` (run it, then rebuild) and is Indian in its vocabulary, kept to a
few bold rings: a lobed, cusped edge like a Mughal or Rajput medallion with a gold jewel in each lobe, a dotted rangoli
ring, the inscription (ARKAYA LABS · MMXXVI over the top, SUI JURIS under the bottom) with lotus buds at three and
nine, and a two-layer gilt lotus rosette, as carved on a temple ceiling, around an enamel medallion holding the mark.
Its colours (ink, `--gold`, the enamel; the petals are gold by day and bronze by night) are set in `styles.css` under
"The seal", so it follows the theme. It shows up to 440 px wide, on a soft disc of page colour with a low shadow.

## Page colours

By day each page's background is the sky of its own painting, sampled from the band where the picture fades into
the page. By night the page keeps the deep blue (`#0e1322`) and, just above the closing painting, eases into that
painting's moonlit sky over a long fade (`.ark-lead` and `--ark-lead` near the end of `styles.css`), so there is no
edge. The header, demo frame and lines are mixed from the page colour; the ledger's dark band fades in and out.

The fade is not one flat colour any more. Up to take 27 it was (`--ark-sky`), and wherever the top of a painting is
darker than that colour (Paro's pines and cliff, Delphi's left sky and right-hand cliffs) the page went dark, then a
lit band, then dark again. Now each painting has two small strips, `assets/<name>.lead.png` (day) and
`<name>.night.lead.png` (night), made by `tools/lead_strips.py` from the painting's own top edge: column by column the
fade arrives at the colour actually at the top of the picture, never lighter than the old sky colour, and dark masses
spread softly outward as they rise instead of standing as stripes. In every column the colour moves one way only,
from the page into the picture. The night strip fades in over the day one on the --dusk clock, and both share one mask
(the old mask also ended on a sudden stop that drew a faint light line at the top of the picture; it is now flat at
both ends).

The paintings have open sky across their top fifth, and the footer sits on it: the picture is drawn up under the
footer (`.ark-reveal` in `styles.css`), a little further on short, wide windows, so that at the bottom of the page
the footer always clears the top bar (checked from 1280 × 720 to 2560 × 1440, and on phones).

## Editing

The four pages are built from `src/pages/*.html` plus shared pieces in `src/partials/` (seal, stack diagram) and the
header, footer and painting blocks in `src/build.py`. After editing, run `python3 src/build.py`. Editing the built
HTML directly also works, but the next build overwrites it.

- The demo carousel moved to the Larch site (Arkaya-Labs/larch-website). Its styles are still in `styles.css`,
  unused here.
- Release-notes signup: set `data-endpoint` on `<form id="signup">` in `src/pages/projects.html` (Buttondown,
  Formspree, or your own URL). It posts JSON `{email}`. Until then it says release notes are not set up yet and
  points people to GitHub.
- Backdrop strength: `WASH` in `site.js` (default .73, so ~27% of the painting shows through).
- The ledger: edit the lists at the top of `src/ledger.py`, then rebuild. Engine rows are drawn as the difference
  from the engine named on the row (right of centre = better), on one shared −60 % to +60 % axis; anything bigger is
  clipped with a break mark and labelled in words. Blue = ahead, lotus pink = behind (checked for colour-blind
  separation in both modes). The exact numbers are under "Show the numbers as a table".

## Search and AI crawlers

- Each page's `<head>` (from `src/build.py`) carries a canonical URL, Open Graph and Twitter card tags, and an
  Organization/WebSite JSON-LD block. Titles and descriptions are in `PAGES`, canonical URLs in `CANON`.
- Social cards are `assets/og/<painting>.jpg`, 1200 × 630 centre crops of each page's day painting.
- `robots.txt` allows every crawler, naming the AI search crawlers explicitly. `sitemap.xml` lists the four pages:
  update its `lastmod` dates when a page changes. `llms.txt` is a short plain-text summary for AI tools.
- Larch is one product: an AI engine with a built-in agent. Don't present "Larch engine" and "Larch" as two products.

## Swapping paintings

Put `<name>.webp` (day) and `<name>.night.webp` (night) in `assets/` at the same size, plus half-width
`<name>.small.webp` / `<name>.night.small.webp` for the blurred backdrop. Then make the lamp/star layer:

    python3 tools/masks.py <name> --sun <moon x>,<moon y> --sky <how far down the sky reaches>   # stars
    python3 tools/lamps.py <name>                                                               # flames

`lamps.py` keeps only small, bright, warm points that stand out from their surroundings (flames, lit windows,
lamp strings) and drops anything larger than a lamp, and (since take 29) the faint warm residue between lamps, so lit
hillsides never breathe; it writes a preview to /tmp to check by eye. Chittorgarh used
`--size 6 --min 0.12`; Paro and Delphi the defaults. The `.mask.webp` it also writes is for motion, which the site no longer uses; delete it. Add the
moon position to `PAINT` in `site.js`, and the backdrop rules (`.page-<name>`) and lead-in strip rules
(`.page-<name> .ark-lead::before/::after`) in `styles.css`, and the painting's size to `SIZES` in `src/build.py`.
Paintings should have open sky across roughly their top fifth, with every tree, cliff and cloud below it: the footer
sits on that sky and the page fades into it. If a new painting has another shape, change the backdrop's aspect-ratio
(`.ark-landscape-image`) and the footer overlap (`.ark-reveal`, which assumes a picture 55.82 % as tall as it is wide)
in `styles.css` to match.
The top-edge fade of each closing painting comes from `tools/fade_masks.py` (now a short straight fade, .14 to .20 of
the height, since the tops are open sky); add the new painting to `SHAPES` there and run it. The fade above each painting
carries the painting's canvas grain (`assets/canvas.grain.png`, from `tools/sky_grain.py`, built from the original
full-quality Chittorgarh night painting, whose open sky shows the weave most cleanly). Finally add the painting's day
page colour (its sky, sampled from the top 6 %) and its top-edge sky colours to `PAGE` and `SKY` at the top of
`tools/lead_strips.py` and run it, for the lead-in strips (`SKY` also caps how far the fade may climb toward a lighter
top edge, which keeps the footer readable). Set the same day colour as that page's `--ark-paper` in `styles.css`.
Take 29 used: Paro `--sun 0.298,0.422 --sky 0.56`, Delphi `--sun 0.314,0.388 --sky 0.5`, Chittorgarh (no moon)
`--sun 0.5,-1 --sky 0.45`; take 30's Ellora `--sun 0.612,0.326 --sky 0.45`,
and for its lamps `tools/lamps.py ellora --glow 3` (its lit windows are only a few pixels, so they get a wider halo to
breathe in).
Needs `opencv-python`, `numpy` and `Pillow`.

## Image size

The paintings are 2576 × 1438. That is sharp up to about a 1700 px wide window on a 1.5× display and is fine
everywhere else for a painterly image. For crisp results on large Retina/4K screens, supply them at **3840 × 2144**
(same 1.79 : 1 ratio) and re-run the conversion and masks. Export the night skies at high quality: compression shows
as faint blocks and banding in a smooth dark sky, right where it meets the fade.

## Checked

JavaScript syntax; all four pages in light and dark at 1440 × 900 and 390 × 844 in headless Chrome; WebGL canvases
start on every page; theme toggle lands on the right colours and is remembered across pages. Google Fonts were not
reachable in the test environment, so the screenshots used fallback fonts.

## Navigation

The bar has three links, Projects, Research and About, plus the sun/moon switch. The Arkaya Labs mark goes home. On
phones narrower than 400 px the wordmark shortens to "Arkaya", and below 340 px to the mark alone, so the bar fits.

## Performance on modest machines

- The sun/moon switch uses View Transitions where the browser has them (Chrome, Edge, Safari 18+, recent
  Firefox): the page swaps palette at once and the GPU crossfades one snapshot over 2.4 s, while the icon animates
  live. Older browsers fall back to the CSS fade on `--dusk`.
- The background paintings have their blur baked into `*.small.webp` (no live CSS blur), the backdrop sits on its
  own GPU layer, and there is no frosted-glass blur under the header or the ledger panel.
- The top bar has no band at rest: a soft veil appears behind the nav only while the page is scrolling and fades
  0.7 s after it stops (the delay is in `site.js`).

- Home hero buttons: "Get Larch" (to larch-engine.com) and "Read the research". The demo carousel
  has no Pause motion button; it still stops for keyboard focus and reduced motion.
