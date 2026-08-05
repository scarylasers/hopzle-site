# hopzle-site

The landing page at **hopzle.com** for the **Hopzle Toolkit** — free tools for
streamers, aimed particularly at people streaming from inside VR, where reaching
a keyboard isn't an option.

One static HTML file whose whole job is to point people at the
[Hopzle Discord](https://discord.gg/r4z4EVnt9U) and list the tools as they ship.

This replaces the old Hopzle web app, which is retired.

The hero wordmark art only reads "HOPZLE" — "TOOLKIT" is set in Bungee beneath it
(`.hero-kind`) so the lockup reads as the full name without needing new art.

## Cost

$0/month. It's a Render **Static Site** — free on any plan, custom domain and TLS
included. The only recurring cost is the domain renewal.

## Layout

```
index.html          the entire site — markup + CSS inline, no build step
assets/
  logo.png                    hardhat bunny, from the original app
  wordmark.png                HOPZLE text logo
  hopperdock.png              HopperDock bunny
  bunnydeck.png               Bunny Deck paws
  streamhopper.png            Stream Hopper headset
  gpumonitor.png              GPU Monitor chip glyph
  voicemeetermini.png         Voicemeeter Mini fader panel
  videodownloader.png         Video Downloader
  triggerword-alpha.webm      animated TriggerWord logo, VP9 + real alpha
  triggerword.mp4               └ H.264 fallback, black background baked in
  triggerword-still.webp        └ poster + frozen frame for prefers-reduced-motion
  cliphopper-anim.webp        animated film-strip split
  cliphopper-still.webp         └ frozen frame, for prefers-reduced-motion
  cliphopper-wordmark.webp    CLIP HOPPER logo — not currently on the page
  favicon.ico
  apple-touch-icon.png
```

No framework, no dependencies, no build step. **First paint is ~116 KB** (HTML
plus hero art); the tool art below the fold is all `loading="lazy"`, so it never
blocks the Discord button.

## Adding a tool

Copy a `<div class="card">` in the grid and change three things:

1. **Art** — *one* visible element. If it's animated, ship a matching still frame
   and give the pair `class="anim"` and `class="anim-still"`. Works for `<video>`
   as well as `<img>`; a video also needs `autoplay muted loop playsinline`.
2. **Badge** — `live` / `beta` / `soon`, and add `is-live` to the card itself for
   the green border.
3. **Links** — released tools get a `.btn-row` with Download (`.primary`), Source
   and Guide. Unreleased ones get a `<p class="pending">` line instead, which
   keeps the card heights even. Only link a repo that's actually public — a
   private one 404s for every visitor.

Commit, push to `master`, Render redeploys in about 40 seconds.

## The doubled-logo trap

`.card-art` holds two elements for animated tools and hides one with CSS. That
only works while the hiding rule out-specifies the sizing rule:

```css
.card-art img, .card-art video { display:block }   /* (0,1,1) */
.card-art .anim-still          { display:none  }   /* (0,2,0) — wins */
```

Keep the show/hide selectors **class-only**. `(0,2,0)` beats `(0,1,1)` on class
count, and staying off the type selector means the same rule works whether the
art is an `<img>` or a `<video>`.

An earlier version hid the still with a bare `.anim-still{display:none}` — one
class — which **lost** to `.showcase-art img{display:block}` and rendered both
frames stacked on top of each other.

## The TriggerWord video

It's the only animation here that's video rather than WebP, and it ships twice: a
VP9 WebM carrying real alpha, and an H.264 MP4 fallback with the background baked
to black.

Chrome, Edge and Firefox composite VP9 alpha in WebM. **Safari does not.** The
`<source>` order puts the WebM first so most visitors get a logo that floats on
the card; Safari falls through to the MP4 and still looks right, because the
colour data under the transparent pixels is pure black — so it lands on the
rounded black screen that `.screen` is styled for.

Only one file is ever downloaded per visitor: ~744 KB on WebM browsers, ~681 KB
on the fallback path.

`.card-art .screen` sets `border-radius` and deliberately **no background** — a
background would sit behind the transparent pixels and undo the alpha.

Animated WebP was not viable for this clip: 1028 KB with alpha, versus 211 KB for
VP9. WebP only wins on short clips, which is why Clip Hopper is still a WebP.

## Things worth knowing

- The Discord invite appears **twice** — the CTA button and the footer. Change
  both together.
- Use a **permanent** invite (Expire After: Never, Max uses: No limit) or the
  page quietly becomes a dead end.
- `og:image` and `og:url` hardcode `https://hopzle.com` — update if the domain
  changes.
- There's no `favicon.svg`; the original was a 318 KB traced SVG, and the `.ico`
  plus apple-touch-icon already cover every browser.
- Regeneration recipes for the animations, and the mapping from each card to its
  source project, live in `NOTES.md` — which is gitignored, because this repo is
  public.
