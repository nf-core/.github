# Working in this repo

A handful of files, no code, no CI. The risk here is not complexity — it is
**blast radius**. `renovate-config.json` is read by every nf-core repo from the
default branch, so a merge is live org-wide on Renovate's next run. There is no
pipeline to catch a mistake and no staging copy to catch it in.

Treat every change to the preset as a production deploy of a config you cannot
observe from inside this repo.

## Validate it yourself

`renovate-config-validator` is weaker than it looks. It passes on both failure
modes that actually break this repo:

```console
$ renovate-config-validator renovate.json   # extends a preset that 404s
 INFO: Config validated successfully
$ renovate-config-validator c.json          # comments + trailing comma in a .json
 INFO: Config validated successfully
```

It checks schema, not syntax strictness and not remote resolution. That gap is
what let [nf-core/tools#4446](https://github.com/nf-core/tools/issues/4446)
through: the config validated while the preset it pointed at 404ed for every
public consumer.

Checks 1 and 2 run for you — `pre-commit` locally, and the `pre-commit` workflow
on every PR. Run all three by hand when you want the answer before pushing:

```bash
# 1. schema
npx --package renovate renovate-config-validator renovate-config.json

# 2. strict JSON — the validator accepts JSON5 here, Renovate's auto-discovery does not
python3 -c "import json; json.load(open('renovate-config.json'))"

# 3. publicly reachable, unauthenticated (the 4446 failure)
curl -s -o /dev/null -w "%{http_code}\n" \
  https://raw.githubusercontent.com/nf-core/.github/master/renovate-config.json
# expect 200. While a preset change is still on a branch, master legitimately
# 404s — swap master for your branch name to check the file you actually edited.
```

Check 3 is the one nothing can automate: CI validates the file in the PR, but
only a fetch of the default branch proves consumers can actually read it.

## Test before merging

Renovate reads the preset from the default branch, so there is no dry run after
the fact. Extend it from your branch instead:

```json
{ "extends": ["github>nf-core/.github:renovate-config#my-branch"] }
```

Use the `github>` form when testing, not `local>`. A local run rejects `local>`
outright — _"The platform you're using (local) does not support local presets"_ —
and the two are equivalent on github.com.

To confirm a preset resolves, look for the fetch rather than trusting a green
validator:

```bash
LOG_LEVEL=debug renovate --dry-run=lookup 2>&1 | grep -i preset
```

Expect a `contents/renovate-config.json` fetch and no `Preset fetch error`.

Install the CLI once rather than reaching for `npx` per invocation, which is slow
enough to derail a debugging loop:

```bash
npm i --prefix /tmp/rnv renovate   # then /tmp/rnv/node_modules/.bin/renovate
```

## Open a PR

`master` and `main` require a PR with one approving review. Members of the `core`
team can bypass that ruleset, so a direct push from `core` will succeed — open a
PR regardless. The preset is read by every nf-core repo, which is more reach than
an unreviewed push should have.

## Scope

`process_updaters.json5` is opt-in and nothing extends it. Chaining it from
`renovate-config.json` would switch on three custom managers for every repo in
the org at once, so leave it unreferenced unless that is the explicit goal.

Keep changes to the preset separate from changes to `profile/README.md`. They
share a repo and nothing else.

## Deeper context

`README.md` covers what the preset is and how repos consume it.
[`docs/agents/renovate.md`](https://github.com/nf-core/ops/blob/main/docs/agents/renovate.md)
in `nf-core/ops` carries the migration state: the full consumer list, the command
to re-derive it, and why the preset must not move back into `ops`.
