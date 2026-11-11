# Contributing to Vectora Documentation

Thanks for helping improve the public documentation site. The project uses
Hugo Extended with Hextra and publishes with GitHub Pages.

## Prerequisites

- Go version declared in `go.mod`.
- Hugo Extended `0.163.3`.

## Preview and build

```bash
hugo server --buildDrafts
hugo --gc --minify --environment production
```

The generated `public/` directory is a build artifact; do not commit it.

## Content guidelines

- Write and maintain pages in English and Brazilian Portuguese.
- Keep the two language versions consistent.
- Document only public open-source repositories in the Vectora organization.
- Do not add material about private repositories, internal services, secrets,
  or non-public endpoints.
- Keep the home page placeholders empty until content is approved.

## Pull requests

- Use Conventional Commit titles in Portuguese.
- Describe the content or site behavior changed and note the build command run.
- Keep changes focused and avoid committing generated Hugo output.
