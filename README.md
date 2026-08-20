# .github

Files for the @nf-core GitHub organisation.

## Contents

| Path | Purpose |
| --- | --- |
| `profile/README.md` | The org profile page shown at [github.com/nf-core](https://github.com/nf-core). See [Customizing your organization's profile](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile). |
| `renovate-config.json` | The shared Renovate preset every nf-core repo extends. See below. |
| `process_updaters.json5` | Optional Renovate custom managers for Nextflow process containers, conda, and pypi. Opt-in; nothing extends it by default. |

## Renovate preset

`renovate-config.json` is the org-wide Renovate configuration. Repos pick it up
in one of two ways:

- **Automatically.** Renovate looks for a `renovate-config.json` in an org's
  `.github` repo during onboarding and suggests it as the sole extended preset.
  New nf-core repos therefore inherit it with no action required.
- **Explicitly**, for repos already onboarded:

  ```json
  {
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": ["local>nf-core/.github:renovate-config"]
  }
  ```

`local>` resolves against whatever platform Renovate is running on, which on
github.com behaves exactly like `github>`. See the Renovate docs on
[group/organization level presets](https://docs.renovatebot.com/config-presets/#grouporganization-level-presets).

### Why it lives here

It used to live in `nf-core/ops`, referenced as
`github>nf-core/ops//.github/renovate/default.json5`. When `nf-core/ops` became
private that reference stopped resolving for public repos, Renovate treated it
as a config error, and dependency PRs stopped org-wide (see
[nf-core/tools#4446](https://github.com/nf-core/tools/issues/4446)).

A Renovate preset is only useful if every consumer can read it, so it belongs in
a public repo. This one is public, already exists for every org, and is the
location Renovate probes by convention — so the preset is reachable and
auto-discovered without making the rest of `ops` public.

### Why it is `.json` and not `.json5`

Auto-discovery looks for the `.json` extension specifically. The preset was
ported from JSON5 to strict JSON to keep that behaviour; it had no comments, so
only syntax sugar was lost. Presets referenced explicitly by name may use
`.json5`, which is why `process_updaters.json5` keeps its extension.

### Changing the preset

Edits here affect every nf-core repo, so open a PR rather than pushing to
`master`. `renovate-config.json` must stay valid strict JSON — no comments, no
trailing commas. To check it locally:

```bash
npx --package renovate renovate-config-validator renovate-config.json
```

Renovate reads the preset from the **default branch**, so changes take effect as
soon as they merge. To try a change before merging, extend it from a branch:
`github>nf-core/.github:renovate-config#my-branch`.
