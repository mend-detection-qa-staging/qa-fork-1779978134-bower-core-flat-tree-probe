# bower-core-flat-tree-probe

This probe validates Mend's detection of Bower's flat dependency layout, covering two patterns in a
single manifest: direct dependencies with no transitives (`moment`, covering `flat-resolution-basic`)
and direct dependencies that pull transitives in Bower's flat `bower_components/` layout
(`bootstrap` pulling `jquery`, `backbone` pulling `underscore`, covering `transitive-flat`). Because
Bower installs all packages — both direct and transitive — into the same flat `bower_components/`
directory, Mend must infer parent-chain relationships from each package's own `bower.json`
`dependencies` field rather than a lockfile. This probe operates in manifest-only mode: no
`bower_components/` directory is committed, so Mend's UA resolver must resolve the tree from
`bower.json` at scan time.

## Feature exercised

Patterns covered:
- `flat-resolution-basic`: direct deps with no transitives; validates flat tree foundation.
- `transitive-flat`: direct deps that pull transitives; validates parent-chain detection in Bower's flat layout.

## Expected dependency tree

After `bower install`, 5 packages appear at the top level of `bower_components/`:

| Package | Version (likely) | Source | Group | Direct? | Parent |
|---|---|---|---|---|---|
| bootstrap | 3.4.x | registry | main | yes | (root) |
| backbone | 1.6.x | registry | main | yes | (root) |
| moment | 2.30.x | registry | main | yes | (root) |
| jquery | resolved by bootstrap's constraint | registry | main | no | bootstrap |
| underscore | resolved by backbone's constraint | registry | main | no | backbone |

- Total packages: 5
- Direct: 3 (bootstrap, backbone, moment)
- Transitive: 2 (jquery under bootstrap, underscore under backbone)
- All sources: Bower registry (registry.bower.io)
- No `resolutions` field — no version conflict exercised (separate probe)
- No `.bowerrc` — default configuration only (custom registry/directory are separate probes)
- No `devDependencies` — deferred to a separate probe

### Mend failure modes exercised

- `bower.json` not detected (Mend does not recognise Bower manifests without a lockfile).
- Transitive parent-chain not inferred: `jquery` must be attributed to `bootstrap`, not listed as a
  root-level direct dep.
- `underscore` must be attributed to `backbone`, not listed as a root-level direct dep.
- `moment` must appear as a direct dep with no children (not silently dropped).
- Flat layout misread: Mend must not report all 5 packages as direct deps.

## Probe metadata

```
patterns:             flat-resolution-basic, transitive-flat
pm:                   bower
manifest:             bower.json
lockfile:             none (Bower has no lockfile)
detection_mode:       manifest-only (no bower_components/ committed)
total_packages:       5
direct_packages:      3
transitive_packages:  2
generated:            2026-05-04
target:               remote
remote_repo:          https://github.com/mend-detection-qa/bower-core-flat-tree-probe
```