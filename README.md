# typst-deploy

A reusable GitHub workflow that compiles a Typst document and deploys the PDF to GitHub Pages,
so a document repository carries a caller of a dozen lines instead of a build of its own.
It also holds the fonts the compile runs with, checked out alongside the document.

## Use

Enable Pages for the repository with GitHub Actions as the source,
and copy `templates/typst-deploy.yml` to `.github/workflows/`:

```yaml
jobs:
  deploy:
    uses: luiswirth/typst-deploy/.github/workflows/typst-deploy.yml@v1
    with:
      entry_file: src/main.typ
      output_name: paper.pdf
```

The PDF is served at the Pages URL under `output_name`,
and the landing page redirects to it.

| input | what it is |
| --- | --- |
| `entry_file` | the document to compile, relative to the repository root |
| `output_name` | the file name the PDF is published under |
| `package_path` | a local package root, for a library the repository vendors |

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
