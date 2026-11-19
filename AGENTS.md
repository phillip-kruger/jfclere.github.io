# Working on this site

This is a Quarkus Roq static site, published to GitHub Pages. Keep it that way.

## Layout

- Content in `content/` (front matter + Markdown or HTML), layouts and partials in
  `templates/`, styles in `public/css/main.css`, images in `public/images/`.
- The deploy workflow puts `main.css` inside every published page for first paint
  (`.github/InlineStyles.java`); keep editing the file and linking it normally.
- Site settings are the front matter of `content/index.html` (Roq's "site data"): its
  `title` is the site title (it names the RSS feed and llms.txt too) and its
  `description` the site description. Optional keys the layout reads:
  `analytics: { ga4: XXXXXXXXXX }` (the GA4 Measurement ID WITHOUT the `G-` prefix; the
  `{#ga4 /}` tag puts the snippet on every page), `googleSiteVerification: <code>`
  (the Search Console HTML tag's content value), and `comments:` (see below). Never
  hand-write those snippets in the layout; set the key.

## Templating

- `quarkus.qute.alt-expr-syntax=true` is on: Qute expressions are `{=expr}`. Wrap
  inline `<script>` and `<style>` bodies in `{| ... |}` so braces are left alone.
- Links between pages use `{=site.url('path')}`; images use `{=site.image('name.png')}`
  or `{=site.url('images/name.png')}`. Never hard-code the domain.

## Posts

Posts live in `content/posts/YYYY-MM-DD-slug.md` with front matter `title`,
`description` and optional `image`. The `post` layout comes from the collection config
in `config/application.properties`, which also sets the address pattern:

```properties
site.collections.posts.layout=post
site.collections.posts.link=/:year/:month/:slug.html
```

That pattern is deliberate. It reproduces the addresses the old Blogger site served, so
existing links and search results keep working. Do not change it.

The archive page is `content/blog.html`, which groups every post by year rather than
paginating. The home page shows the most recent few.

## Comments

Posts take comments through giscus, backed by GitHub Discussions on this repository.
The embed is `templates/partials/comments.html` and the post layout includes it once
after the post body. Settings are the `comments` key in `content/index.html`:

```yaml
comments:
  repo: jfclere/jfclere.github.io
  repoId: ""
  category: Announcements
  categoryId: ""
```

While `categoryId` is empty nothing renders, so the site is safe to publish before
Discussions is turned on. Threads are keyed on the post's source file name
(`data-mapping=specific`), not its address, so they survive a move or a new domain;
renaming a post file orphans its thread.

## Generated files, leave them alone

- `content/rss.xml` is the posts feed. `content/llms.qute.txt` and
  `content/llms-full.qute.txt` generate `/llms.txt` for AI assistants. A page opts out
  with `llmstxt: false`.
- `content/sitemap.xml` is generated; a page opts out with `sitemap: false` in its front
  matter (the 404 page does).
- `content/404.html` carries `link: /404.html`, which publishes it to the root as GitHub
  Pages requires. Without it Roq writes `404/index.html`, which GitHub never serves.
  Keep that key if you touch the page.
- `public/robots.txt` carries the site's address in its `Sitemap:` line. Update it if
  the site moves.

## Search engines

Every page has a `title` (under 60 characters) and a one-sentence `description` (80 to
155 characters) in its front matter. The home page also sets `image:` to the name of a
real file in `public/images/` (no `images/` prefix).
`templates/partials/structured-data.html` holds the schema.org JSON-LD; keep it accurate.

Old addresses: a page's `aliases:` front matter list (paths only, for example
`["/2005/06/", "/about-us.html"]`) generates a redirect page at each old path. When a
page moves, add its old path to its aliases; when content merges, the surviving page
carries every old path.

## Front end

- Third-party libraries come from [mvnpm](https://mvnpm.org) as Maven dependencies, not
  a CDN or a copied file. giscus is the one exception, because it is a hosted service
  rather than a library.
- Code blocks: fenced code in Markdown is coloured by highlight.js, bundled from
  `web/app/main.js` (the `org.mvnpm:highlight.js` artifact) and loaded by the post layout
  through `{#bundle /}`. Colours are the `--hl-*` tokens in `main.css`. To support
  another language, import it from `highlight.js/lib/languages/<name>` and register it
  in `main.js`.
- Photo viewer: `web/lightbox/` is its own bundle, loaded on every page by the default
  layout (`{#bundle key="lightbox" tag="script" /}`). A photo's `alt` or `<figcaption>`
  is the description shown under it; `data-no-zoom` keeps an image from opening. Do not
  add another gallery popup.
- Table of contents: the post layout shows one (`page.tocHtml`, TOC plugin) when a post
  has three or more headings.
- Web fonts never block the page. They are served from this site under `public/fonts/`
  and preloaded in the default layout.

## Design

Brand colours are CSS custom properties at the top of `main.css`. Dark theme tokens live
in the two `data-theme` blocks; keep both themes and the `theme-toggle` partial. Token
names are a contract: change what a token is worth, never its name. Keep `.skip-link`,
`.container` and the `.logo-light`/`.logo-dark` swap, which the layouts depend on.

Every page must render well at 360px wide. Tap targets at least 44px. Respect
`prefers-reduced-motion`.

## Rules

- Do not add build tooling (npm, bundlers), server code, or third-party scripts.
- `target/` and `node_modules/` are work folders. Both are ignored; never commit them
  and never commit any file over a few megabytes.
- When checking pages in a browser, never wait for images to load in a script:
  off-screen `loading="lazy"` images never start loading and the wait never ends. Check
  for broken images by fetching their URLs with curl instead.
- Writing style: plain English, no em or en dashes, no marketing filler, no emoji.
- Verify before you finish:
  `QUARKUS_HTTP_PORT=8765 QUARKUS_ROQ_GENERATOR_BATCH=true mvn -q -B package quarkus:run`
  must succeed and `target/roq/index.html` must exist. The sitemap plugin reads each
  page's last commit date, so this only works inside a git repository with a commit.
