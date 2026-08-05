# typst-deploy

A reusable GitHub workflow that compiles a Typst document and deploys the PDF to GitHub Pages,
so a document repository carries a caller of a dozen lines instead of a build of its own.
It also holds the fonts the compile runs with, checked out alongside the document.

## Use

Enable Pages for the repository with GitHub Actions as the source,
which takes one call and has to happen before the first run,

```bash
gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow
```

and copy `templates/typst-deploy.yml` to `.github/workflows/`:

```yaml
jobs:
  deploy:
    uses: luiswirth/typst-deploy/.github/workflows/typst-deploy.yml@v2
    with:
      documents: |
        src/main.typ paper.pdf
```

| input | what it is |
| --- | --- |
| `documents` | the documents to compile, one per line |
| `package_path` | a local package root, for a library the repository vendors |

A line names a Typst file and the name it is published under,
the name defaulting to the file's own with a pdf extension,
so `test/showcase.typ` alone becomes `showcase.pdf`.
One document is served at the Pages URL itself, through a landing page that
redirects to it, so that the URL is the document.
Several are listed on that page instead.

A Pages site is public even when its repository is private,
private Pages being an Enterprise Cloud feature,
so making the repository private hides the sources and publishes the PDF.
A document that may not be read yet is one that is not deployed yet.

The compile runs with the repository root as the Typst root,
so a document may read any file it contains.
Submodules and LFS objects are checked out before it.

## Local packages

A repository that imports a library as `@local/<name>:<version>`
must make that library present in the checkout, a submodule being the usual way,
and name the directory holding `local/<name>/<version>` in `package_path`:

```yaml
      package_path: lib/dottyp/pkg
```

The workflow exports it as `TYPST_PACKAGE_PATH`,
which is the same variable a local build sets,
so remote and local resolve the import the same way.
Left empty, nothing is exported and only Typst Universe packages resolve.

## Fonts

Every font in `fonts/` is on the font path of the compile.
A document needing one that is not there adds it here.
