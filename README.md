# JS Supply Chain Attack Atlas

An interactive, self-contained HTML data story about JavaScript supply-chain attack exposure.

The project explores a simple question:

> When popular JS/TS package versions are staged in isolated package-manager environments, which known malicious package/version indicators appear?

The main demo artifact is:

- [`v2.html`](./v2.html): redesigned interactive story with campaign atlas, timeline, heatmap, evidence drill-downs, proof links, and caveats.

There is also:

- [`index.html`](./index.html): broader scan-matrix version of the story.
- [`METHODOLOGY.md`](./METHODOLOGY.md): detailed methodology, limitations, safety controls, and interpretation guide.

## How To View

Open `v2.html` directly in a browser, or serve the folder locally:

```sh
python3 -m http.server 8000
```

Then visit:

```text
http://localhost:8000/v2.html
```

No build step is required. The HTML files are self-contained with inline CSS, inline JavaScript, and embedded JSON data.

## What It Shows

The data story includes:

- confirmed supply-chain attack campaigns
- staged package/version exposure evidence
- a campaign constellation visualization
- an attack timeline
- a staged exposure heatmap
- exact package/version claim text
- source/advisory links
- evidence filters and CSV export
- caveats about lockfile-only evidence

## Important Caveat

This project does **not** claim that popular GitHub repositories were compromised.

It compares staged package-manager state against known malicious package/version indicators. A lockfile or staged-resolution match is exposure evidence, not proof that malicious code executed.

Use this framing:

- “known bad package-version indicator”
- “observed staged hit”
- “possible dependency exposure”
- “confirmed advisory indicator”

Avoid this framing:

- “this repo was hacked”
- “this app executed malware”
- “no hit means safe”

## Scanner

The analysis is based on:

- `supply-chain-attack@0.1.10`
- advisory snapshot date: `2026-05-12`

The scanner matches local package-manager state against known supply-chain attack indicators. See [`METHODOLOGY.md`](./METHODOLOGY.md) for the full workflow and limitations.

## Safety

The research workflow assumes isolated package-manager environments and disabled lifecycle scripts. Known poisoned artifacts should be inspected with safe tarball workflows such as `npm pack`, not installed into a normal developer environment.

## License

Add your preferred license before publishing if this repository will be shared publicly.
