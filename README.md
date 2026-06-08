# typedoc-pages-action

Build versioned [TypeDoc](https://typedoc.org) documentation with
[`@shipgirl/typedoc-plugin-versions`](https://github.com/Vampire-Hunter/typedoc-plugin-versions)
and publish it to GitHub Pages — **preserving the version dropdown across releases**.

Each release adds a new version to the site instead of overwriting it. The action restores the
previously published docs before building, so the versions plugin appends rather than replaces.

## Usage

Your repository provides a `typedoc.json` that uses the versions plugin and writes to the output
directory (default `docs`), plus a `typedoc` npm script:

```jsonc
// typedoc.json
{
  "entryPoints": ["./src/index.ts"],
  "plugin": ["@shipgirl/typedoc-plugin-versions"],
  "includeVersion": true,
  "out": "docs"
}
```

Then the workflow is a thin wrapper — check out the repo, then call the action:

```yaml
name: Publish Docs
on:
  workflow_dispatch:
  workflow_call:
permissions:
  contents: write
concurrency:
  group: pages
  cancel-in-progress: false
jobs:
  docs:
    name: 📚 Publish Docs
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: Bugs5382/typedoc-pages-action@v0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

Set the repository's **Pages source to the `gh-pages` branch** (Settings → Pages). The versions
plugin needs that persistent branch to accumulate history, so this action does not use the
"GitHub Actions" Pages source.

## Inputs

| Input | Default | Description |
|---|---|---|
| `github-token` | _(required)_ | Token used to read the publish branch and deploy — usually `secrets.GITHUB_TOKEN`. |
| `versions` | `true` | Preserve version history (restore prior docs before building, for `@shipgirl/typedoc-plugin-versions`). Set `false` for a single-version site that is replaced on each deploy. |
| `node-version` | `lts/*` | Node.js version used to build the docs. |
| `docs-dir` | `docs` | Directory TypeDoc writes to (must match `out` in `typedoc.json`). |
| `publish-branch` | `gh-pages` | Branch the rendered site is published to. |
| `install-command` | `npm ci --ignore-scripts --legacy-peer-deps` | Dependency install command. |
| `build-command` | `npm run typedoc` | Command that runs TypeDoc. |

## How it works

1. Restore the docs already published on `publish-branch` into `docs-dir` — skipped on first run,
   and skipped entirely when `versions: false`.
2. Install dependencies and run TypeDoc; with the versions plugin enabled this adds the current
   version alongside the restored ones.
3. Deploy `docs-dir` to `publish-branch` via `peaceiris/actions-gh-pages`.

With `versions: false` the action just builds and deploys the current docs (each deploy replaces
the previous site) — use this when you don't run the versions plugin.

## Acknowledgments

Thanks to [`@shipgirl/typedoc-plugin-versions`](https://www.npmjs.com/package/@shipgirl/typedoc-plugin-versions)
— the version dropdown and multi-version history this action publishes are powered by that plugin.

## License

MIT
