# Repository guidance

This repository contains the public documentation site for Vectora Ltda.'s open-source projects. It
is a Hugo + Hextra site; it is not the Engine source repository and does not host documentation for
private repositories.

## Content and links

- Write user-facing content in English and Brazilian Portuguese.
- Keep paired pages synchronized and link between pages using Hugo's content references where
  possible.
- Only document public open-source repositories. Do not mention private products, repositories,
  services, or internal endpoints.
- Keep the home pages as minimal placeholders until approved documentation content is added.
- Use the canonical site URL `https://docs.vectora.company/` in metadata and absolute links.

## Build and deployment

- Use Go as pinned by `go.mod` and Hugo Extended `0.163.3`.
- `hugo --gc --minify --environment production` builds the site into `public/`.
- `public/`, `resources/_gen/`, and `.hugo_build.lock` are generated outputs and must remain
  untracked.
- Pull requests and pushes to `main` run the Hugo build. Pushes to `main` deploy through GitHub
  Pages using the workflow in `.github/workflows/deploy-pages.yml`.
- Do not add app runtimes, Rust bindings, or project-specific build steps here.
