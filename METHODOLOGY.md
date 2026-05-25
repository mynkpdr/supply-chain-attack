# Methodology

This document describes how the JS supply-chain attack data story was built, what the embedded data means, and what claims the final HTML artifacts can and cannot support.

The short version: this project does **not** claim that popular GitHub repositories were compromised. It stages selected npm package/version environments, compares the resolved package state against the `supply-chain-attack@0.1.10` advisory snapshot, and presents the result as exposure evidence with explicit confidence boundaries.

## Artifacts

The repository currently contains two self-contained HTML artifacts:

- `index.html`: the broader data-heavy artifact. It embeds the scan matrix, advisory metadata, scan rows, source notes, and visualizations.
- `v2.html`: the redesigned demo artifact. It emphasizes self-guided storytelling, a campaign atlas, a timeline, a staged exposure heatmap, exact proof links, evidence drill-downs, and caveats.

Both HTML files are standalone: inline CSS, inline JavaScript, embedded JSON, and no runtime CDN dependency.

## Scanner Scope

The project centers on `supply-chain-attack@0.1.10`.

Important constraint: the scanner does not scan arbitrary GitHub repository source trees as source code. It inspects local package-manager state and matches package/version artifacts against its advisory snapshot. For this reason, the data story phrases results as package-artifact exposure evidence, not repository compromise.

The scanner snapshot used by the current artifacts is:

- Scanner: `supply-chain-attack`
- Version: `0.1.10`
- Snapshot date: `2026-05-12`
- Advisory/campaign count represented in `index.html`: `8`
- Tracked artifact count represented in `index.html`: `438`
- Tracked artifact count represented in `v2.html`: `438`

## Research Question

The project asks:

> If selected popular JS/TS package versions are staged in isolated package-manager environments, which known malicious package/version indicators appear in their resolved dependency state?

Related questions:

- How large is the dependency surface for selected packages over time?
- Which known supply-chain campaigns are confirmed by the scanner snapshot?
- Which confirmed indicators were observed in the staged matrix?
- Which rows are direct incident anchors rather than popular-package transitive hits?
- How should teams interpret lockfile-only evidence versus stronger local machine evidence?

## Data Units

The main unit is a package-version artifact, not a Git commit.

Examples:

- `next@4.2.2`
- `storybook@7.6.6`
- `handlebars@4.7.9`
- `@tanstack/react-router@1.166.6`
- `@rspack/core@1.1.7`

Popular GitHub repositories are represented through their npm package artifacts. For example, `vercel/next.js` is represented through selected `next` package versions.

## Repository And Package Selection

The original plan selected popular JS/TS ecosystem projects and incident anchor packages.

Popular package/repository surfaces represented in the current embedded data include packages such as:

- `react`
- `@angular/core`
- `vue`
- `next`
- `typescript`
- `vite`
- `svelte`
- `express`
- `axios`
- `@babel/core`
- `webpack`
- `storybook`
- `eslint`
- `tailwindcss`
- `@nestjs/core`

Incident anchor packages include known-bad or advisory-referenced package/version artifacts, such as:

- Axios malicious releases
- Nx compromised package versions
- Rspack compromised package versions
- TanStack compromised package versions
- Mini Shai-Hulud related package artifacts

This mix is intentional. Popular packages show ecosystem dependency surface; incident anchors ensure the story contains confirmed hits even when popular-package rows are sparse.

## Version Sampling

For each popular package, versions were sampled around release and incident windows:

- oldest usable baseline
- latest before `2024-01-01`
- latest before the Nx incident window, `2025-08-27`
- latest before the Axios incident window, `2026-03-31`
- latest before the TanStack incident window, `2026-05-11`
- latest stable at collection time
- incident-anchor rows where relevant

The embedded `index.html` dataset currently contains:

- `91` staged rows
- `18` represented repositories/packages
- `85` `resolved_lockfile` rows
- `6` `resolution_failed` rows
- `12` finding records
- `11` rows with at least one finding

The `v2.html` storytelling dataset summarizes:

- `91` staged package/version rows
- `5,054` resolved package pairs
- `12` observed staged hits
- `5` observed campaigns
- `12` popular package surfaces
- `8` confirmed campaigns
- `44` representative confirmed indicators

## Staging Procedure

The intended safe staging workflow is:

1. Create a disposable working directory.
2. Use an isolated `HOME` and package-manager cache.
3. Stage a target package/version or copied lockfile/manifests.
4. Resolve dependencies without lifecycle script execution.
5. Run the scanner against the isolated state.
6. Normalize findings into the embedded data schema.
7. Delete or archive disposable outputs after validation.

Representative command pattern:

```sh
HOME="$RUN/home" npm_config_cache="$RUN/home/.npm" NO_COLOR=1 \
npx --yes supply-chain-attack@0.1.10 --json --no-interactive --fail-on none
```

Safe install patterns:

```sh
npm ci --ignore-scripts
pnpm install --frozen-lockfile --ignore-scripts
YARN_ENABLE_SCRIPTS=false yarn install --immutable
```

For known poisoned package artifacts, the safe inspection method is `npm pack <name>@<version>`, not `npm install`, because lifecycle scripts must not execute during research.

## Safety Controls

The methodology assumes:

- disposable directories for every run
- isolated `HOME`
- isolated npm cache
- no host secrets mounted into the staging environment
- lifecycle scripts disabled during dependency resolution
- no execution of package binaries from staged artifacts
- tarball inspection through `npm pack`, not installation, for known malicious artifacts

The HTML artifacts are static and do not execute the scanned packages. They only render embedded JSON.

## Evidence Confidence

Evidence strength is intentionally separated from campaign confirmation.

Recommended interpretation ladder:

- `lockfile = possible resolution`: the package/version appeared in a resolved dependency state or lockfile-like output. This is exposure evidence, not proof of execution.
- `node_modules = installed dependency`: the package/version existed in an installed dependency tree. This is stronger than lockfile-only evidence.
- `cache/store = fetched artifact`: the artifact was fetched into a package-manager cache or store.
- `_npx/global/bin = likely executed`: the package was found in a location commonly associated with command execution.

The current public story mostly uses lockfile/staged-resolution evidence and advisory anchors. It should therefore use phrases like:

- "observed staged hit"
- "confirmed indicator"
- "possible resolution"
- "package artifact exposure"

It should avoid unsupported phrases like:

- "this GitHub repository was compromised"
- "this app executed malware"
- "all users of this package were affected"

## Campaign Confirmation

Campaigns are considered confirmed when they are represented in the scanner snapshot or cited advisory/reporting sources.

In `v2.html`, all eight campaign cards are confirmed indicators from the advisory snapshot or source reporting. Only some of them were observed in the staged package matrix.

This distinction is central:

- **Confirmed campaign**: the scanner/source identifies malicious package-version artifacts.
- **Observed staged hit**: one of those package-version artifacts appeared in the staged experiment.

## Heatmap Methodology

The `v2.html` proof heatmap is built from `stagedScan.observedRows`.

Rows are staged package/version subjects, such as:

- `next@4.2.2`
- `storybook@7.6.6`
- `axios@1.14.1`
- `@tanstack/react-query@5.90.12`

Columns are confirmed campaigns, such as:

- Nx
- Rspack
- Axios
- TanStack
- Mini Shai-Hulud May

Each colored cell means:

> At least one exact package-version indicator for that campaign was observed while staging that row.

Empty cells mean:

> No embedded observed hit exists for that row/campaign pair in this dataset.

Empty cells do not prove safety.

## Link Methodology

The story includes proof links, but package registries often remove or hide malicious versions. Because exact npm version pages can 404 after yanking, `v2.html` uses stable package-level version/history links while keeping the exact version in visible text.

For example:

- visible claim: `@tanstack/react-router@1.166.6`
- stable link: `https://www.npmjs.com/package/@tanstack/react-router?activeTab=versions`

Scoped npm packages must preserve the slash in `@scope/name`; encoding the slash as `%2F` can produce broken npm links.

Advisory/source links are used for confirmation when registry artifacts are yanked or unresolved.

## Data Normalization

Rows are normalized around the following concepts:

- `repo`: GitHub repository represented by a package
- `pkg`: npm package name
- `version`: package version
- `date`: package release date, when available
- `milestone`: sampling bucket
- `count`: resolved package count
- `hits`: number of scanner/advisory findings in that row
- `status`: resolution state
- `findings`: package/version indicators observed in that row

Findings include:

- package name
- package version
- advisory/campaign ID
- campaign title
- source URL
- confidence label
- evidence text

The `v2.html` dataset additionally separates:

- confirmed campaign sample indicators
- staged observed rows
- popular package surface rows
- response actions
- source notes

## What The Metrics Mean

Dependency count:

> Number of resolved packages in the staged dependency state for the selected package/version row.

Observed hit:

> A known package-version indicator appeared in the staged row or incident anchor row.

Confirmed indicator:

> A package/version appears in a source-backed campaign or scanner advisory snapshot.

Resolved pair:

> A package/version pair represented in the staged dependency surface.

Resolution failed:

> The package/version could not be fully resolved at collection time. Incident-anchor failures can happen because malicious versions are yanked or no longer resolvable.

## Limitations

This methodology has important limits:

- It is not a full source-code audit of GitHub repositories.
- It is not a claim that a repository's main branch contains malicious code.
- It does not prove local execution unless evidence location supports execution.
- It can miss packages not included in the scanner snapshot.
- It can miss incidents not represented in the selected version windows.
- It can be affected by yanked package versions and registry behavior changes.
- It may undercount dependency paths when lockfile paths are not embedded.
- It may overstate risk if readers treat lockfile-only exposure as execution.

## Review Checklist

Before presenting or publishing the data story, verify:

- Embedded JSON parses in both HTML artifacts.
- The visible counts match the embedded data.
- Every positive hit has package, version, campaign/advisory, confidence, and source.
- Heatmap cells match `stagedScan.observedRows`.
- Proof links do not use broken scoped npm encoding.
- Exact package versions remain visible even when registry links point to package history pages.
- All caveats are visible before readers reach detailed evidence.
- No external runtime assets are required.
- No `fetch()` or CDN script is needed for the story to run.

## Browser QA

Recommended manual checks:

1. Open `v2.html` from a local server.
2. Confirm there are no console errors.
3. Check the hero, campaign atlas, timeline, surface bars, heatmap, mechanism section, evidence explorer, response cards, and sources.
4. Click a campaign node and a package dot.
5. Click a heatmap cell and confirm the popup contains exact version text plus source links.
6. Search the atlas and evidence explorer.
7. Toggle the theme.
8. Download CSV and confirm URL columns are present.
9. Test mobile width.
10. Test reduced-motion mode.

## Static QA

Useful checks:

```sh
node - <<'NODE'
const fs = require('fs');
for (const file of ['index.html', 'v2.html']) {
  const html = fs.readFileSync(file, 'utf8');
  const data = JSON.parse(html.match(/<script type="application\/json" id="data-json">([\s\S]*?)<\/script>/)[1]);
  for (const [, script] of html.matchAll(/<script>([\s\S]*?)<\/script>/g)) new Function(script);
  console.log(file, 'ok', Object.keys(data));
}
NODE

rg -n "<script src|<link[^>]+stylesheet|fonts.googleapis|fonts.gstatic|cdn|fetch\(|import |src=\"https|url\(https" index.html v2.html
```

For `v2.html`, also check generated package links:

```sh
rg -n "npmjs.com/package/%40|/v/" v2.html
```

The expected result is no broken scoped-package encoding and no brittle exact npm `/v/` links in generated proof UI.

## Source Policy

Primary source categories:

- scanner repository and package metadata
- GitHub Security Advisories
- npm documentation
- Socket campaign reports
- maintainer postmortems where available

The story should cite sources as evidence for campaigns, but it should not quote long passages from them.

## Reproducibility Notes

The final HTML files embed normalized data rather than raw scan outputs. Helper scripts and raw outputs may have existed during research, but the public artifact is intentionally one-file HTML.

To reproduce the data, rebuild staged environments using the same scanner version and date-aware package/version sampling. Expect some differences over time because npm registry state changes, malicious versions may be yanked, and advisory snapshots can update.

## Claim Language

Use this language:

- "This staged row resolved a known indicator."
- "This is exposure evidence."
- "This package/version is confirmed by the advisory snapshot."
- "The exact version may no longer resolve from npm."
- "The source repository can be clean while a registry artifact is poisoned."

Avoid this language:

- "The repository was hacked."
- "The app was compromised."
- "The dependency executed."
- "No hit means safe."
- "The npm page is the sole proof."

## Current Embedded Counts

At the time this document was written, the embedded artifacts reported:

| Artifact | Rows | Findings / Observed Hits | Campaigns | Notes |
| --- | ---: | ---: | ---: | --- |
| `index.html` | 91 | 12 findings | 8 advisories | broader scan matrix |
| `v2.html` | 91 staged rows | 12 observed hits | 8 campaigns | redesigned narrative and heatmap |

`v2.html` also reports `5,054` resolved package pairs and `44` representative confirmed indicators.
