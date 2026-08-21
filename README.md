# Anas — portfolio site

A single-page portfolio for a videographer, filmmaker and editor. No framework, no build
dependencies, no third-party requests at runtime.

## Run it

Because the page loads video files, open it through a local server rather than double-clicking
the file — browsers block `file://` media requests in some cases.

```bash
cd anas-portfolio
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploy it

Drag the whole `anas-portfolio` folder onto Netlify Drop, or push it to a GitHub repo and point
Vercel, Cloudflare Pages or GitHub Pages at it. There is nothing to compile.

Before going live, set the real URL in three places in `index.html`: `<link rel="canonical">`,
`og:url`, and the `url` field inside the JSON-LD block. Right now they all say `example.com`.

## What to fill in

Every unfinished value is styled as dashed steel-blue text. Search `index.html` for **`class="ph"`**
to jump between them:

| Where | What to add |
|---|---|
| Screening room + every project modal | Year for each film |
| Project modals | Client names for the six unnamed clients |
| About → spec list | Camera and lenses, edit and colour tools, typical turnaround |
| About → portrait | Save a photo to `assets/poster/portrait.webp` and replace the placeholder block |
| Contact | Your real email — it appears twice: in the `href="mailto:"` and in the button text |
| Contact → socials | The five `href="#"` links |
| Clients grid | The event halls, the contractor and the café aren't named yet |

Project text, roles and descriptions live in the `PROJECTS` object near the top of the `<script>`
block, and in the card markup. Change both if you rename a piece.

## Adding a real showreel

The screening room currently plays *Tel Jadore Book Café* as the feature. When you cut a proper
reel:

1. Export it and save as `assets/video/showreel.mp4`
2. Grab a poster frame: `ffmpeg -ss 5 -i assets/video/showreel.mp4 -frames:v 1 assets/poster/showreel.webp`
3. In `index.html`, find `id="reelVideo"` and change `data-src` to the new path, then update the
   title, runtime and metadata in the panel beside it.

## Adding a new project

1. Encode a web copy and a silent hover loop:

```bash
SLUG=my-new-film
ffmpeg -i source.mp4 -vf scale=720:1280 -c:v libx264 -preset veryfast -crf 26 \
       -pix_fmt yuv420p -c:a aac -b:a 96k -movflags +faststart assets/video/$SLUG.mp4
ffmpeg -ss 2 -t 5 -i source.mp4 -vf scale=396:704 -an -c:v libx264 -preset veryfast \
       -crf 31 -pix_fmt yuv420p -movflags +faststart assets/preview/$SLUG.mp4
ffmpeg -ss 5 -i source.mp4 -frames:v 1 -vf scale=460:-2 -c:v libwebp -quality 58 \
       assets/poster/$SLUG.webp
```

2. Copy any `<button class="card">` block in `template.html`, change the slug, alt text, badges,
   title and blurb. Use `POSTER:my-new-film` as the image `src`.
3. Add a matching entry to the `PROJECTS` object.
4. Run `python3 build.py`.

Use `card--wide` for 16:9 pieces, `card--half` for a vertical you want shown larger, and
`card--push` to drop a card down the page for a staggered row.

## How the build works

`build.py` reads `template.html` and inlines two things into `index.html`:

- the Archivo, Martian Mono and Sora webfonts, as base64
- every poster frame, as base64 webp

That means the page paints completely on first load with zero network round-trips — no font flash,
no empty grey boxes. Videos stay as real files and load only when they're needed.

**Edit `template.html`, not `index.html`.** `index.html` is generated and gets overwritten.

## Performance notes

- Nothing autoplays until it's in view. Card previews are 5-second silent loops, ~150 KB each.
- Full films load only when a project is opened.
- The hero loop is skipped on phones and on Save-Data connections; the still frame stands in.
- `prefers-reduced-motion` disables the grain, the reveals and all previews.
- Playing media pauses when the tab is hidden.

## Files

```
anas-portfolio/
├── index.html            generated — don't edit
├── template.html         the source you edit
├── build.py              regenerates index.html
├── README.md
└── assets/
    ├── video/            11 web-encoded films (720p / 1280p, faststart)
    ├── preview/          silent hover loops + the hero ambient loop
    ├── poster/           first-frame stills + og-cover.jpg for link previews
    └── fonts/            self-hosted woff2
```

Source footage totalled 511 MB; the encoded set is about 70 MB.
