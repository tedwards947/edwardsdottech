# Agent context — edwardsdottech

Static personal site for Tony Edwards: portfolio/CV + "hobby build" project
pages. Plain HTML/CSS, no build step, hosted on GitHub Pages with a custom
domain (`CNAME` → `edwards.tech`). Repo: `tedwards947/edwardsdottech`.

## Structure

```
index.html              — home: hero/about + "From the Workshop" project list
cv.html                  — CV, built from Tony's real resume content
style.css                — single shared stylesheet, all pages link it
images/                  — site-wide images (currently just the hero portrait)
projects/<slug>/
  index.html             — one page per hobby project
  images/                — that project's photos, kept alongside its HTML
```

Each project lives in its own folder so HTML + images stay together.
Current projects: `nitrollm`, `flwlamp`, `teddy-tank`, `danse-macabre`,
`matrixbox` (folder/slug must match the real GitHub repo name where one
exists — e.g. `matrixbox` because that's what it's called on GitHub, not
"tudum-box" which was a placeholder name used early on).

## Linking convention

All internal links (nav, CV, back-links, stylesheet) are **root-relative**
(`/style.css`, `/cv.html`, `/index.html#workshop`), not page-relative. This
was a deliberate choice so pages nested under `projects/<slug>/` don't need
`../../` chains. Local dev must be served from the site root (see below) for
these to resolve.

Project detail pages use directory-style URLs: link to `/projects/<slug>/`
(trailing slash, no `.html`) since each project folder's `index.html` is
served automatically.

## Design system (style.css)

- Cool/minimal palette (CSS vars in `:root`): light slate `--paper`
  background, near-black `--ink`, deep teal `--accent` (#1f6f78). This
  replaced an earlier warm cream/terracotta palette that looked too much
  like Anthropic's site — cool slate was chosen specifically to move away
  from that.
- Headings use **Fraunces** (serif) — keep this; it was an explicit choice
  to preserve during the palette rework.
- Small "label" text (kickers/eyebrows, nav links, chips/tags, dates,
  "View project →", "← Back" links, footer social links) uses
  **IBM Plex Mono** via the `--mono` var — an intentional engineering/
  technical accent, distinct from the serif headings and Inter body text.
- `.status-tag` + `.status-done` / `.status-wip` / `.status-paused` — pill
  badges for project status, color-coded (green/amber/gray). Used inline
  next to the title via a `.title-row` wrapper (flex row, gap, wraps h1/h3 +
  tag together) on both the homepage cards and each project's own page.
  Keep these two in sync when a project's status changes.
- `.repo-link` — dark pill CTA button (GitHub mark SVG + "See this project
  on GitHub") for linking out to a project's GitHub repo. Most hobby
  projects will eventually have one. Place it in `.project-hero`, right
  after the `.lead` subtitle paragraph — not buried in `.project-body` —
  so it reads as a prominent CTA near the title rather than a plain text
  link at the bottom. See `projects/nitrollm/index.html` for the reference
  markup (inline SVG + `target="_blank" rel="noopener"`). Only add this to
  a project page once there's a real repo URL — don't invent one.

## Image handling patterns

Mixed photo orientations (e.g. a tall vertical lamp photo vs. wide shots)
led to a two-tier approach — don't reflexively reach for `object-fit: cover`
everywhere:

- **Homepage list thumbnails** (`.workshop-photo`): fixed 150×150 square,
  `object-fit: contain` — shows the whole image (letterboxed/pillarboxed) so
  nothing gets cropped, while every tile in the list stays the same size and
  the text column stays aligned.
- **Project hero photo** (`.project-media`, not inside `.gallery-row`): no
  forced aspect ratio — image displays at natural aspect, capped at
  `max-height: 640px`, centered. Full quality, no cropping.
- **Project gallery row** (two photos side by side, `.gallery-row
  .project-media`): fixed 4:3 boxes, `object-fit: contain` — keeps mixed
  orientation pairs the same height and aligned as a row.

### Captions

Wrap any `.project-media` or `.video-embed` block in `<figure class="media-figure">`
with a `<figcaption>` to add a caption — small mono text, centered, muted color,
consistent with the rest of the site's label styling. Works standalone or inside
a `.gallery-row` cell. Example (see `projects/nitrollm/index.html` for a live one):

```html
<figure class="media-figure">
    <div class="project-media"><img src="images/total.jpeg" /></div>
    <figcaption>The whole setup, mid-build.</figcaption>
</figure>
```

Don't wrap `.project-media`/`.video-embed` in `<figure>` unless there's an actual
caption to add — no need to add empty figure wrappers everywhere.

## Video embeds

- Normal case: YouTube unlisted/public videos via `.video-embed > iframe`
  (16:9 box, `object-fit` doesn't apply to iframes but the box handles it).
- `matrixbox` is a special case: the demo video is hosted as a GitHub
  "user-attachment" (dragged into a PR/issue), not YouTube. Those URLs
  (`github.com/<user>/<repo>/assets/<id>/<uuid>`) are **not
  iframe-embeddable** (GitHub sends `X-Frame-Options: deny` and the URL
  itself 302-redirects to a signed, expiring S3 URL, not an HTML page).
  They work fine with a plain `<video><source src="...github.com/assets/...">`
  tag instead — the browser follows the redirect itself. Caveat: these
  attachments are often `.mov` (QuickTime) served as `video/quicktime`,
  which Safari plays but Chrome/Firefox may refuse regardless of the actual
  codec — confirmed working in this case, but worth remembering if another
  GitHub-hosted video gets added and doesn't play.

## Contact info / privacy

Tony asked not to expose a raw email (spam risk) — the site intentionally
has **no visible email address anywhere**. Footer/contact links use GitHub
and LinkedIn only. Don't add a `mailto:` link unless he asks for it again.

## Git / deploy

- Remote: `origin` → `https://github.com/tedwards947/edwardsdottech.git`,
  branch `main`. Pushing to `main` triggers GitHub Pages redeploy
  automatically (custom domain via `CNAME`).
- Tony sometimes edits files directly on GitHub's web UI (has happened at
  least once — a `<title>` tweak) or drops in his own local edits between
  agent turns — check `git status`/`git fetch` before assuming the local
  tree or remote history is exactly what you last left it. If `git push` is
  rejected, `git fetch` + inspect the diff before merging/rebasing; prefer
  `git rebase origin/main` to keep history linear when the remote change is
  small and non-conflicting.
- `.gitignore` excludes `.claude/` and `.DS_Store` — not site content.
- Only commit when explicitly asked; this has been the working pattern all
  along (add specific files, not `-A` blindly, though `-A` has been used
  once when the working tree was known-clean of anything else).

## Local dev

Tony runs `python3 -m http.server` from the repo root and views the site at
`http://localhost:8000/`. Because links are root-relative, `/` must resolve
to `index.html` (it does, by default, with `http.server`) — viewing
`index.html` directly via `file://` will break every internal link.

## Known placeholders / TODO-ish state

- `danse-macabre` and `teddy-tank` still have some placeholder body copy
  ("Placeholder paragraph one...") — real project write-ups pending.
- CV (`cv.html`) content was transcribed from Tony's real resume PDF and
  should be treated as accurate/real, not placeholder — don't regenerate it
  from scratch without checking with him first.
