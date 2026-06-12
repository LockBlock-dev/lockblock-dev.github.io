# Blog

Hugo static blog using the PaperMod theme as a Git submodule.

## Setup

```sh
git submodule update --init --recursive
```

Install Hugo Extended, then run locally:

```sh
hugo server
```

Build static files:

```sh
hugo
```

The theme is configured in `hugo.yaml` and lives in `themes/PaperMod`.
