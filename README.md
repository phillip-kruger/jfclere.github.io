# Website

Static site built with [Quarkus Roq](https://iamroq.dev/) and published on GitHub Pages
by [Great Southern Websites](https://greatsouthernwebsites.com.au).

## Local development

Requires Java 21+.

```bash
./mvnw quarkus:dev          # or: mvn quarkus:dev  -> http://localhost:8080
```

## Generate the static site

```bash
QUARKUS_HTTP_PORT=8765 QUARKUS_ROQ_GENERATOR_BATCH=true mvn -B package quarkus:run
```

Output lands in `target/roq/`.

## Editing content

| What | Where |
| --- | --- |
| Pages | `content/*.md` and `content/*.html` (front matter + Markdown/HTML) |
| News posts | `content/posts/YYYY-MM-DD-slug.md` |
| Layouts and partials | `templates/` |
| Styles | `public/css/main.css` |
| Images | `public/images/` |
| Site config | `config/application.properties` |

## Comments

Posts take comments through [giscus](https://giscus.app), which stores each thread as a
GitHub Discussion on this repository. Nothing is self-hosted and no visitor is tracked.
The embed lives in `templates/partials/comments.html` and the post layout includes it.

Settings are the `comments` key in the front matter of `content/index.html`, beside
`analytics`. **While `categoryId` is empty no comment section is rendered at all**, so the
site is safe to publish before any of this is done:

```yaml
comments:
  repo: jfclere/jfclere-blogspot-com
  repoId: ""
  category: Announcements
  categoryId: ""
```

To turn comments on, the repository owner does this once:

1. Settings > General > Features > tick **Discussions**.
2. Install the [giscus app](https://github.com/apps/giscus) on this repository.
3. Keep or create the **Announcements** discussion category.
4. Run through [giscus.app](https://giscus.app) and copy the `data-repo-id` and
   `data-category-id` it gives you into `repoId` and `categoryId` above.

Threads are keyed on the post's source file name (`data-mapping=specific`), not its
address, so they survive moving the repository, a GitHub Pages path and a later custom
domain. Renaming a post file orphans its thread. Moderation is done in the repository's
Discussions tab.

## Deployment

Pushing to `main` runs `.github/workflows/deploy.yml`, which builds the site with the Roq
GitHub Action and publishes it to GitHub Pages. The workflow also runs daily so
future-dated posts publish on time.
