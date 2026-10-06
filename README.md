# typst-deploy

A reusable GitHub workflow that compiles a Typst document and deploys the PDF to Cloudflare Pages,
so a document repository carries a caller of a dozen lines instead of a build of its own.
It also holds the fonts the compile runs with, checked out alongside the document.

## Use

Create a Cloudflare Pages project named after the repository,
and give the repository the two secrets the deploy authenticates with,
a token allowed to edit Cloudflare Pages and the account it belongs to.
Both have to happen before the first run:

```bash
wrangler pages project create <repo> --production-branch=main
gh secret set CLOUDFLARE_API_TOKEN -R <owner>/<repo>
gh secret set CLOUDFLARE_ACCOUNT_ID -R <owner>/<repo>
```

Then copy `templates/typst-deploy.yml` to `.github/workflows/`:

```yaml
jobs:
  deploy:
    uses: luiswirth/typst-deploy/.github/workflows/typst-deploy.yml@v4
    with:
      documents: |
        src/main.typ paper.pdf
    secrets: inherit
```

| input | what it is |
| --- | --- |
| `documents` | the documents to compile, one per line |

A line names a Typst file and the name it is published under,
the name defaulting to the file's own with a pdf extension,
so `test/showcase.typ` alone becomes `showcase.pdf`.
One document is served at the site's URL itself, which redirects to it,
so that the URL is the document.
Several are listed on that page instead.

The site is public even when its repository is private,
so making the repository private hides the sources and publishes the PDF.
A document that may not be read yet is one that is not deployed yet.

The compile runs with the repository root as the Typst root,
so a document may read any file it contains.
LFS objects are checked out before it.

## The document's environment

The compile runs inside the repository's own `devShell`,
so the Typst it is deployed with is the one it is written with,
and a library the flake puts on `TYPST_PACKAGE_PATH` resolves remotely
as it does locally.

A repository deployed this way therefore needs a flake
whose default `devShell` carries Typst.

## Fonts

Every font in `fonts/` is on the font path of the compile.
A document needing one that is not there adds it here.
