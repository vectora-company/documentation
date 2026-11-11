# Vectora Documentation

Official documentation site for the open-source projects maintained by the
[Vectora organization](https://github.com/vectora-company). The site is built with
[Hugo](https://gohugo.io/) and [Hextra](https://imfing.github.io/hextra/) and is published at
<https://docs.vectora.company/> through GitHub Pages.

## Local development

Requirements: Go version from `go.mod` and Hugo Extended `0.163.3`.

```bash
hugo server --buildDrafts
hugo --gc --minify --environment production
```

Content belongs in `content/`. Keep the English and Brazilian Portuguese versions aligned. Do not
document private repositories or services here; this site is for public open-source projects.

## Publishing

- Pull requests and pushes to `main` run the Hugo build in `.github/workflows/ci.yml`.
- A push to `main` deploys the generated site through `.github/workflows/deploy-pages.yml`.
- GitHub Pages must use **GitHub Actions** as its publishing source. The custom domain is
  `docs.vectora.company`.

## License

See [`LICENSE`](LICENSE).
