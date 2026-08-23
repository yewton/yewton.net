# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`yewton.net` is a personal blog repository, not a software product. Content is authored directly in
Markdown as Hugo page bundles and published via Hugo + [HugoBlox](https://hugoblox.com/)
(Tailwind CSS edition), hosted on Netlify. See `openspec/project.md` for the full project context
(purpose, tech stack, writing style/tone conventions).

## Content workflow (critical)

New articles are written and edited directly in Markdown — the Emacs Org-mode / `ox-hugo` pipeline
is no longer used for authoring. (Legacy articles still carry Org sources in
`content-org/all-posts.org`; that file is kept for reference only and is not the source of truth for
new work. The Org sources for site pages and author data are gone — top-level pages are
`content/_index.md` / `content/blog/_index.md` / `content/cv/_index.md`, and author data is
`data/authors/<slug>.yaml`.)

- **Where content lives**: each post is a Hugo page bundle at `content/post/<YYYY-MM-DD_slug>/index.md`,
  with images and other assets in the same directory. Edit `index.md` directly; there is no Org export
  step to run.
- **Front matter template** for a new post:

  ```yaml
  ---
  title: "記事タイトル"
  author: ["yewton"]
  date: 2026-08-25
  # lastmod: 2026-08-30T11:00:00+09:00   # add ONLY when you intentionally revise the content
  slug: "url-slug"
  tags: ["tag1", "tag2"]
  categories: ["雑記"]   # pick from existing categories (e.g. 技術系)
  draft: true            # set to false to publish; flipping to DONE-equivalent is a human decision
  image:
    caption: 画像のクレジット/キャプション
  ---
  ```

- **Last-updated date policy**: set the standard `lastmod:` front matter key **only when you
  intentionally revise a published article's content**. Do not bump it for mechanical/trivial edits.
  `enableGitInfo` is deliberately not enabled (git commit dates would move `lastmod` on mechanical
  edits). A local override (`layouts/_partials/components/last-edited.html`) shows the "最終更新" line
  only when `lastmod` is later than `date`, so unrevised posts don't display a redundant one.
- **Table of contents**: do not hand-write a TOC list. HugoBlox generates one automatically from
  headings, but only as a **desktop** right-sidebar (`xl:` and up); on mobile there is no TOC by
  default. To give mobile readers navigation, add the `{{< toc >}}` shortcode near the top of the body
  (just after the front matter) — it renders a collapsible `<details>` that shows on mobile and is
  hidden on desktop (`xl:hidden`), so it doesn't duplicate the sidebar. Add it per-article as needed.
- **Body syntax**: plain Markdown. Inline code is `` `code` `` (backticks). For callouts use Markdown
  alert syntax (`> [!NOTE]`, `> [!WARNING]`), not the legacy `{{% callout %}}` shortcode. For image
  credits and embeds, call the Hugo shortcodes directly (`{{< unsplash ... >}}`, `{{< pexels ... >}}`,
  `{{< pixabay >}}...{{< /pixabay >}}`) — the `{{{...}}}` Org macros are not available in Markdown.
- **No `mylastmod`**: the legacy `mylastmod` front matter key and the `{{< lastmod >}}` shortcode that
  rendered it are gone. `lastmod` is the only last-updated field. (`content-org/all-posts.org` still
  contains the ox-hugo directive that produced `mylastmod`, so re-exporting from Org would reintroduce
  it — another reason that file is reference-only.)

## Commands

Tool versions are pinned via `.tool-versions` (mise): `hugo-extended 0.159.1`, `golang 1.24.3`.

- `hugo server` — local dev server.
- **Review preview over Tailscale** — when work is ready for the author (or an agent) to review,
  serve it on this machine's Tailscale IP and hand back the URL:
  `hugo server --bind 0.0.0.0 --baseURL http://$(tailscale ip -4):1313/ -D` (`-D` includes drafts).
  Then report the reviewable URL (e.g. the specific post path) so it can be opened from any device on
  the tailnet. Do this by default at the end of a content/theme change instead of only building.
- `hugo --gc --minify` — production build (matches Netlify's default build command).
- `hugo mod ...` — manage the Hugo Go module dependency on `github.com/HugoBlox/kit/modules/blox`
  (declared in `go.mod`; no vendoring — see Architecture below).
- `npx pagefind --source 'public'` — build the search index after a Hugo build (Netlify's
  `split1`/deploy-preview/branch-deploy contexts run this after `hugo`; see `netlify.toml`).
- No `npm` scripts are defined; `package.json` only pins `tailwindcss`, `@tailwindcss/cli`,
  `@tailwindcss/typography`, and `preact` as build-time dependencies consumed by HugoBlox's
  own asset pipeline (`templates.Defer`-based `css.html`), not invoked directly.

## Architecture

This is a HugoBlox (Tailwind CSS) site, migrated from the older Wowchemy/Bootstrap theme (see
`MIGRATION_REPORT.md` for the full migration history, rationale, and gotchas — read it before
touching theme/config internals, as it documents many non-obvious HugoBlox behaviors).

- **Theme via Hugo Modules**: `config/_default/hugo.yaml` imports
  `github.com/HugoBlox/kit/modules/blox`. No `_vendor/` directory is committed — modules are
  downloaded at build time (Netlify has Go available via mise).
- **`hugo_stats.json` is required for styling**: Tailwind CSS v4's JIT scanner reads
  `hugo_stats.json` (`@source "hugo_stats.json"`) to discover classes used inside the HugoBlox
  module's own templates, which aren't otherwise visible to Tailwind. This requires
  `build.buildStats.enable: true` in `config/_default/hugo.yaml` — do not remove it, or large
  swaths of CSS silently go missing. The file is gitignored and regenerated every build.
- **Page composition**: top-level pages (`content/_index.md`, `content/cv/_index.md`) use
  `type: landing` with a `sections:` list of blocks (`hero`, `collection`, `resume-biography`,
  `resume-experience`, `resume-skills`, `resume-awards`, `markdown`, ...). The legacy
  `type: widget_page` / `content/home/*.md` widget system is gone.
- **Author/profile data is data-driven, not content-driven**: `data/authors/<slug>.yaml`
  (schema `hugoblox/author/v1`) is what `resume-*` blocks and `get_author_profile` read —
  `content/authors/` is not used. Avatar images live at `assets/media/authors/<slug>.*`.
- **Params namespace**: `config/_default/params.yaml` must live entirely under a `hugoblox:` key
  (the old Wowchemy `features.*`/`locale.*`/`marketing.*` top-level keys are silently ignored).
  Per-section behavior (comments, related posts, breadcrumbs) for `/post/**` is set via `cascade`
  in `config/_default/hugo.yaml`, not in `params.yaml`.
- **CSS**: only `assets/css/custom.css` is loaded by the theme; `assets/scss/custom.scss` is
  dead/unused. Keep custom overrides in the `.css` file, flat (no SCSS nesting).
- **Block/partial overrides**: local overrides of HugoBlox blocks live under
  `layouts/_partials/hbx/blocks/<block-name>/block.html` (currently `resume-awards`,
  `resume-biography`). Custom shortcodes live in `layouts/shortcodes/`
  (`pexels`, `pixabay`, `unsplash`).
- **Config layout**: `config/_default/` holds the base config (`hugo.yaml`, `menus.yaml`,
  `params.yaml`); `config/production/config.toml` only overrides `baseURL` for production.

## OpenSpec workflow

This repo uses [OpenSpec](https://github.com/) for spec-driven change management — see the
managed block in `AGENTS.md` and the full instructions in `openspec/AGENTS.md`. Key points specific
to this repo (from `openspec/project.md`):
- All OpenSpec artifacts (`proposal.md`, `tasks.md`, `spec.md`, etc.) must be written in Japanese.
- Do not start implementation until a proposal has been reviewed and explicitly approved by the user.
- Active/archived changes live under `openspec/changes/`; capability specs under `openspec/specs/`.

## Writing style (for blog content)

- です・ます調 (polite form), applied from the draft stage onward.
- Technical content: logical and concise. Personal opinions/impressions should be distinguishable
  from objective facts.
- First person is 私, but may be omitted depending on context.
