# torb/hello-app

A one-file [TorbScript](https://torb.dev) command-line app: it prints a greeting for the name it is given, or
"World" if none. Fork it and turn it into your own tool.

## Using this template

1. **Fork or copy this repository** under your own owner - on
   [GitHub](https://github.com/TorbScript/app-template) with the "Use this template" button, or on
   [git.torb.dev](https://git.torb.dev/torb/app-template) by forking it there.
2. **Rename the app.** In `project.trb`, change `name = "torb/hello-app"` to `name = "<your-owner>/<your-app>"`. The
   binary `torb build` writes is named after the part after the `/`.
3. **Rewrite `src/main.trb` and `src/greeting.trb`** with your own logic; `tests/greeting.test.trb` shows the pattern
   this template uses to keep logic testable without running the whole program - a plain function in its own module,
   imported by both `main.trb` and the tests.
4. **Replace the copyright line of `LICENSE`** with your own name and the current year.
5. **Adjust `.github/workflows/release.yml` and `.forgejo/workflows/release.yml`** for the targets you need - see
   "Releasing" below.

## Developing

```console
$ torb format --check .    # the layout torb format writes
$ torb check .              # types, exhaustiveness, everything torb check finds
$ torb test                  # tests/greeting.test.trb
$ torb doc --check .          # every doc comment link resolves (there are no examples to run here)
$ torb run                    # interpreted, in the VM
$ torb run --native . Grace   # compiled once, then run: prints "Hello, Grace!"
$ torb build                  # the release binary, build/release/hello-app(.exe)
```

`torb format .` writes the layout instead of only checking it. `.github/workflows/ci.yml` and
`.forgejo/workflows/ci.yml` run format, check, test, doc and build on every push and pull request.

## Releasing

Pushing a tag `v0.1.0` runs `.github/workflows/release.yml` and `.forgejo/workflows/release.yml`: each builds the
`linux-x64` release binary with `torb build` and attaches `hello-app-linux-x64.tar.gz` to a release of the forge it
runs on. Add another entry to the `matrix` of the GitHub workflow (`macos-latest`, `windows-latest`) once you need
another target - `torb build` always builds for the machine the job runs on, so cross-compiling for a target the
runner is not is a separate step this template does not need yet. Neither workflow needs a secret: `contents: write`
is enough to create a release and attach a file to it.

## Publishing

**This app is not published to the registry.** `torb publish` technically accepts a package that is only
`src/main.trb` - there is no gate that refuses an app - but the one thing publishing is for is `use owner/name from
"..."`, and an entry file can never be imported (`src/main.trb` "yes" under Top-level code, "no" under Importable,
[docs/design/PROJECT.md](https://git.torb.dev/torbscript/language/src/branch/main/docs/design/PROJECT.md) section 3):
nobody could `use` what this template publishes. The binary is the thing to distribute, and "Releasing" above is how.

If your app grows a library part worth importing - a parser, a client, anything another package could `use` - split
it into `src/lib.trb` and turn to [package-template](https://github.com/TorbScript/package-template) for how to
publish that half.

## Adding a dependency

This template depends on nothing but the standard library. To add a package once one exists on the registry (for
example, a published `package-template`):

```trb
dependencies {
  runtime "torb/hello:^0.1.0"
}
```

in `project.trb`, then `use greeting from "torb/hello"` in `src/`. `torb check` resolves and locks it into
`project.lock.trb` the first time it runs.

## Files

```text
app-template/
├─ project.trb              name, version, description, license
├─ src/
│  ├─ main.trb                the program: "torb/hello-app" builds hello-app
│  └─ greeting.trb            the one testable function main.trb calls
├─ tests/
│  └─ greeting.test.trb      torb test finds *.test.trb anywhere below the package
├─ .github/workflows/         ci.yml (push, pull request), release.yml (tag v*)
├─ .forgejo/workflows/        the same, for git.torb.dev
├─ LICENSE                    MIT - replace the copyright line
└─ .gitignore
```

## Related

- [docs/design/PROJECT.md](https://git.torb.dev/torbscript/language/src/branch/main/docs/design/PROJECT.md) - what a
  package produces, and what makes a file a program.
- [torb build](https://torb.dev/docs/tooling/torb-build) and
  [torb run](https://torb.dev/docs/tooling/torb-run) - profiles, targets and what a program name means.
