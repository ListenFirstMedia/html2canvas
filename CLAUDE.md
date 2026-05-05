# CLAUDE.md — `html2canvas` (LFM fork)

> **One-line purpose:** LFM's fork of Niklas von Hertzen's MIT-licensed `html2canvas` (DOM-to-canvas screenshot library). The fork's only purpose is to commit a built `dist/` so it can be consumed via git URL.

## 1. Repo identity

- **Repo type:** `library` (TypeScript / JavaScript browser library; LFM fork of OSS upstream).
- **Primary language(s):** TypeScript (265 `.ts` files in `src/`), JavaScript (156 `.js` files, mostly tests + dist + www/Gatsby site). Node `>= 8.0.0` (per `package.json` `engines`).
- **Primary framework(s):** Pure DOM library — no framework. Built with Rollup 2 (UMD + ESM output) + TypeScript 4.3 + Babel 7. Tests use Jest 27 (unit) + Karma 6 (browser, multi-browser via Sauce Labs). Documentation site uses Gatsby (`www/`).
- **Lifecycle status:** `maintenance-only` — single LFM commit (build dist/ files for git-URL consumption) on top of upstream v1.3.2. Upstream is at https://github.com/niklasvh/html2canvas (Niklas von Hertzen, MIT-licensed).
- **Repo URL:** https://github.com/ListenFirstMedia/html2canvas (default branch: `master`).
- **On-call / owners:** _<TODO: confirm — the only LFM merge is by `Paulette Tawil <paulette.tawil@listenfirstmedia.com>` (2021-11-14). Frontend platform team likely owns it.>_
- **Slack channel:** _<TODO>_
- **Confluence / docs root:** Upstream homepage at https://html2canvas.hertzen.com (linked from `README.md`); upstream docs at `docs/`.

## 2. Why this repo exists

LFM forked `niklasvh/html2canvas` for one practical reason: **to commit a built `dist/` to the repo** so that `lfm-js-commons` can consume the library via a `github:` npm dep URL (`"html2canvas": "github:ListenFirstMedia/html2canvas#master"`). The upstream project doesn't commit `dist/`; it publishes to npm — but LFM's frontend lib (`lfm-js-commons`) treats every dep as a git URL with committed dist files, and `html2canvas` is no exception.

The single LFM merge commit (`98058c5`, _"Merge pull request #6 from ListenFirstMedia/revert-5-revert-4-APPS-24484-run-build"_, 2021-11-14, Paulette Tawil) added the upstream source + a freshly-built `dist/` directory to the LFM tree. The merge title's `revert-5-revert-4-APPS-24484` history hints that the LFM team did some build-vs-no-build flip-flopping before landing on "yes, commit the build."

There is **no LFM-specific business logic** — this is a vendored fork purely for distribution mechanics.

## 3. Where it sits in the platform

A leaf library. Used only at build time / runtime by frontend code.

- **Upstream services this repo calls** (runtime HTTP/RPC): _N/A — this is a browser library. At runtime it walks the DOM in the user's browser and rasterizes it. It does not initiate calls itself, though host pages may use the `proxy` option to route cross-origin image fetches through an HTTP proxy (any URL the consumer supplies)._
- **Downstream consumers** (LFM repos that bundle THIS as a library, per the dependency map):
  - `lfm-js-commons` — `package.json` declares `html2canvas: github:ListenFirstMedia/html2canvas#master` as both a peer dep and a dev dep. `lfm-js-commons` re-exports it; downstream apps (`explorer`, `reporting-redesign`, `tiles`) consume it transitively when generating tile/dashboard image exports.
- **Library deps** (compile-time, from `package.json`):
  - Runtime: `css-line-break 2.0.1`, `text-segmentation ^1.0.2`. That's it — only two runtime deps.
  - DevDeps: 60+ for testing, building, and the Gatsby docs site (Rollup, Babel, TypeScript, Karma, Jest, Mocha, Express, Webpack, ESLint, Prettier, etc.).

> Phase 1 dependency map for this repo: `~/Documents/claude-github/phase1/html2canvas.json`.

## 4. HTTP / RPC interface

_N/A — this is a browser library; it exposes no HTTP routes._

It does ship a **dev server** for the upstream test suite (`npm start` → `ts-node tests/server --port=8080 --cors=8081`), used for running rendering tests against a local Express server. Not a production surface.

## 5. APIs this repo CONSUMES

_N/A — html2canvas runs entirely in the browser DOM. The consumer page may pass an HTTP proxy URL via the `proxy: '<url>'` option to fetch cross-origin images, but that's the consumer's concern, not html2canvas's._

## 6. Tech stack details

- **Runtime versions:**
  - Node `>= 8.0.0` (per `package.json` `engines`); CI uses Node 12.
- **Package manager(s):** npm. `package-lock.json` is committed (1.6 MB).
- **Build/test commands:**
  - Install: `npm install`
  - Build (UMD + ESM + minified + types): `npm run build`
  - Watch: `npm run watch`
  - Lint: `npm run lint` (ESLint with TypeScript + Prettier; `--max-warnings 0`)
  - Format: `npm run format`
  - Unit tests (Jest): `npm run unittest`
  - Browser tests (Karma, multi-browser): `npm run karma`
  - Full test (lint + unit + karma): `npm test`
  - Reftest diff: `npm run reftests-diff`
  - Reftest preview: `npm run build:reftest-preview` + `npm run start`
  - Release: `npm run release` (uses `standard-version`)
- **Notable internal LFM gems / packages bundled here:** None — pure upstream tree.
- **Why we forked:** Distribution. LFM consumes via `github:` git URL (`lfm-js-commons` `package.json`); the upstream npm-published artifact isn't suitable because LFM wants to pin a specific tree-ish AND have `dist/` available immediately. The `revert-5-revert-4-APPS-24484-run-build` PR title indicates the team went back and forth on whether to commit `dist/` before landing on "commit it."
- **Build outputs (`dist/`):**
  - `html2canvas.js` (436 KB, UMD)
  - `html2canvas.esm.js` (406 KB, ESM module — the `package.json` `module` field)
  - `html2canvas.min.js` (197 KB, UMD minified — the `package.json` `main` and `browser` fields)
  - `html2canvas.{js,esm.js}.map` (sourcemaps)
  - `lib/` (TypeScript intermediate compilation outputs)
  - `types/` (`.d.ts` declarations — the `package.json` `typings` field points at `dist/types/index.d.ts`)
  - The `dist/` directory **is committed to the repo** as the LFM fork's reason for being.

## 7. Local development

### 7.1 Required services

- None for build/test cycles.
- For browser/Karma tests: a local browser (Chrome/Firefox/Safari/Edge) or Sauce Labs credentials for cross-browser testing.
- For the Express dev server (`npm start`): port 8080 (server) + 8081 (CORS).

### 7.2 Required env vars

| Name | Purpose | Example | Where it's set / read |
|---|---|---|---|
| (none for the library at runtime) | The library reads no env vars — it's pure browser DOM code | — | — |
| `${{ secrets.PAT_TOKEN }}` | Used in `.github/workflows/release.yml` to allow the release workflow to push commits + tags back to the repo | GitHub Actions secret | upstream-style release workflow |
| `${{ secrets.GITHUB_TOKEN }}` | Standard GitHub Actions token | (auto-provisioned) | `ci.yml` |

The library has no LFM-specific env vars.

### 7.3 First-time setup

```
1. brew install nvm (or any Node ≥ 8 manager)
2. nvm install 12 && nvm use 12   # to match upstream CI
3. cd html2canvas
4. npm install
5. npm run build                  # produces / refreshes dist/
6. npm run unittest               # Jest tests
7. # optional: npm run karma      # multi-browser tests (requires Karma + browsers)
```

## 8. Tests

- **How tests are organized:**
  - Unit tests: `src/**/__tests__/*.ts` (Jest, jsdom)
  - Reference (image-diff) tests: `tests/reftests/` driven by `tests/karma.ts` against real browsers
  - End-to-end testing infra: `tests/server.ts` (Express + CORS), `tests/karma/` (Karma config)
- **How to run a single test:** `npx jest <path>` for unit; `npm run karma -- --grep '<name>'` for browser.
- **CI green criteria:** `.github/workflows/ci.yml` runs `npm test` (lint + unit + karma) on push/PR to `master` and on tags.
- **Coverage target / current %:** Not configured.
- **Mocks vs. real services:** Browser tests render real DOM in real browsers and image-diff against committed reference snapshots in `tests/reftests/`. Strict reftest checking is upstream policy — the bar is "rendering output didn't change at all."

## 9. Deployment

- **Where it deploys:** Nowhere directly. This is a library; "deployment" = pushing a new commit to `master` and having `lfm-js-commons` pick it up on its next `npm install` (since `lfm-js-commons` consumes via `github:ListenFirstMedia/html2canvas#master`).
- **How a deploy happens:**
  - Manual: edit src, run `npm run build`, commit dist/, push to `master`.
  - Or trigger `.github/workflows/release.yml` (workflow_dispatch with version input: major/minor/patch/prerelease). Uses `standard-version` and pushes a version tag with `${{ secrets.PAT_TOKEN }}`. _<TODO: confirm whether the LFM team uses this release workflow or just pushes to master manually.>_
- **Branching model:** `master` only.
- **Infra-as-code:** N/A.
- **Rollback:** Revert the dist commit on `master` and force-push, then have `lfm-js-commons` reinstall.
- **Health-check endpoint:** N/A.

## 10. Configuration & secrets

- **Secret store:** None for the library itself. CI uses GitHub Actions secrets.
- **How config is loaded at runtime:** N/A — the library exposes a `Partial<Options>` type (`Options = CloneOptions & WindowOptions & RenderOptions & ContextOptions & {…}`); the consumer passes options at the call site.
- **Where to find example values:** `docs/configuration.md`.
- **Feature flags:** N/A.

## 11. Data stores & state

- **No data stores accessed.**
- **Browser state:**
  - The library reads the entire DOM tree of the target element + ancestor styles.
  - Internally it instantiates a `CacheStorage` (`src/core/cache-storage.ts`) keyed off `window` to deduplicate image fetches across multiple `html2canvas()` calls.
  - It clones the target document into a hidden iframe (`DocumentCloner` in `src/dom/document-cloner.ts`) before rendering, so the original page isn't disturbed.

## 12. File / directory map

```
html2canvas/
├── README.md                      — upstream README (3.7 KB) — describes usage, browser support, building
├── CLAUDE.md                      — this file
├── CHANGELOG.md                   — upstream `standard-version` changelog through v1.3.2 (2021-08-15)
├── LICENSE                        — MIT (Niklas von Hertzen)
├── package.json                   — upstream package metadata (still authored as Niklas's; npm name: `html2canvas`)
├── package-lock.json              — 1.6 MB
├── tsconfig.json                  — strict TS, target es5
├── rollup.config.ts               — UMD + ESM bundle config
├── jest.config.js                 — Jest config (one line: `module.exports = { preset: 'ts-jest' };`)
├── karma.conf.js                  — Karma multi-browser config (Chrome, Firefox, Safari, Edge, IE, Sauce launchers)
├── .babelrc                       — Babel config (preset-env, preset-flow)
├── .eslintrc                      — ESLint with TypeScript + Prettier integration
├── .prettierrc                    — Prettier config (single quotes, etc.)
├── .editorconfig
├── .npmignore                     — controls what's published if anyone publishes to npm (LFM doesn't)
├── .github/
│   ├── ISSUE_TEMPLATE.md, PULL_REQUEST_TEMPLATE.md
│   ├── no-response.yml            — upstream stale-issue bot config
│   └── workflows/
│       ├── ci.yml                 — Build + test on push/PR to master, tags v*
│       └── release.yml            — Manual workflow_dispatch: bump version + push tag (uses PAT_TOKEN)
├── src/
│   ├── index.ts                   — entry point: exports default `html2canvas(element, options)`; sets up CacheStorage; selects renderer (Canvas vs ForeignObject)
│   ├── global.d.ts                — ambient type declarations
│   ├── invariant.ts               — assertion helper
│   ├── core/                      — Context, cache-storage, features, logger
│   ├── css/                       — CSS parsing (types/, layout/, property-descriptors/)
│   ├── dom/                       — DOM walking (node-parser, document-cloner, element-container/, replaced-elements/, text-container)
│   ├── render/                    — canvas-renderer + foreign-object-renderer (two render strategies)
│   └── __tests__/                 — co-located unit tests (Jest)
├── dist/                          — committed build outputs: html2canvas.{js,esm.js,min.js} + sourcemaps + lib/ + types/. **This is the LFM fork's whole point.**
├── tests/                         — Karma + reftest infrastructure
├── docs/                          — Markdown docs: configuration, documentation, faq, features, getting-started, proxy
├── examples/                      — small HTML examples
├── configs/                       — base.json, preview.json, scripts.json (build configs)
├── scripts/                       — build helpers (create-reftest-list.ts, etc.)
└── www/                           — Gatsby site for html2canvas.hertzen.com (separate package.json + dist tree; not built by LFM)
```

Not a monorepo — single npm package at root, with `www/` (a Gatsby docs site) and `tests/` (Karma harness) as auxiliary trees.

## 13. Conventions & gotchas

### ⚠️ Committed credentials

**No new committed credentials found.** The repo is clean: no `.npmrc` token (unlike `app-library`), no Gemfile, no `.env*` files, no AWS keys, no `ghp_*` tokens, no OAuth client secrets. The only "secrets" are GitHub Actions `secrets.GITHUB_TOKEN` (auto-provisioned) and `secrets.PAT_TOKEN` (used by the release workflow).

**No update to `~/Documents/claude-github/phase1/SECURITY_FINDINGS.md` is required for this repo.**

### General conventions

- **This is a vendored fork.** LFM's only patch is "commit the dist/." Any meaningful bug fix should go upstream first (`niklasvh/html2canvas`), then be merged here once released.
- **The `dist/` is committed deliberately.** Do not `.gitignore` it. If you change `src/`, you must `npm run build` AND commit the resulting `dist/` files in the same PR — otherwise consumers (`lfm-js-commons` via `github:` URL) get stale code.
- **The package.json metadata still points at upstream** — `repository.url: git@github.com:niklasvh/html2canvas.git`, `bugs.url: https://github.com/niklasvh/html2canvas/issues`, `homepage: https://html2canvas.hertzen.com`, `author.email: niklasvh@gmail.com`. This is correct (MIT attribution); don't change.
- **Node 12 in CI is ancient.** Upstream hasn't updated. LFM hasn't either. If you need a newer Node feature in `src/`, bump CI's `setup-node` and verify build still produces byte-identical output.
- **Karma + Sauce Labs cross-browser testing is upstream-stock.** LFM doesn't run it (no Sauce credentials in any LFM workflow). The local `npm test` will fail on the karma step unless you have a browser available; CI shells out to a single Chrome.
- **Reference (reftest) snapshots are committed in `tests/reftests/`.** If you change rendering output, you'll fail reftests. Either fix the regression or update the snapshot images and document why.
- **Two renderers, picked at runtime:** `CanvasRenderer` (default) and `ForeignObjectRenderer` (used when `foreignObjectRendering: true` option is passed). The latter is faster but has cross-origin restrictions.
- **`CacheStorage.setContext(window)`** is called at module load. Don't move that call — it must run before any `html2canvas()` invocation.
- **Cross-origin image rendering requires a proxy.** This is documented in `docs/proxy.md`. The consumer is responsible for setting up a proxy (LFM doesn't ship one).
- **Per the upstream README:** "This script is still in a very experimental state." LFM has been using it in production for years, but rendering edge cases (CSS transforms, position-fixed children, web fonts, transitions) regularly cause issues. When users report a "this dashboard doesn't export correctly," check html2canvas first.

## 14. Common tasks (cookbook)

- **Run all tests:** `npm test`
- **Run only unit tests (fast):** `npm run unittest`
- **Run lint:** `npm run lint`
- **Format code:** `npm run format`
- **Rebuild dist:** `npm run build` (then commit `dist/`)
- **Update LFM's `dist/` after pulling new upstream:**
  ```
  git remote add upstream https://github.com/niklasvh/html2canvas.git
  git fetch upstream
  git merge upstream/master    # resolve conflicts (likely just dist/)
  npm install                  # if package.json changed
  npm run build                # regenerate dist/
  git add dist/
  git commit -m "Update dist after merging upstream <ref>"
  git push
  ```
- **Trigger the upstream-style release workflow:** GitHub Actions UI → "Create Release" → workflow_dispatch with `version: patch` (or major/minor/etc.). _<TODO: confirm LFM uses this; the `secrets.PAT_TOKEN` may not exist in the LFM repo, in which case the workflow will fail.>_
- **Test a candidate change without going via dist:** in `lfm-js-commons`, `npm install ../html2canvas --no-save` (point at your local checkout); reload your dev tile.

## 15. Known issues / migrations in flight

- **Upstream stagnation.** Upstream `niklasvh/html2canvas` last released v1.3.2 in 2021-08-15. The library has open issues piling up; LFM's fork hasn't picked anything up since. _<TODO: check if upstream is now at v1.4+ / v2.x and decide whether to merge.>_
- **Node 12 in CI** is below LFS minimums (LFM otherwise uses Node 20). The fork's CI matrix should be bumped, but doing so may require updating Babel / Webpack majors.
- **The single LFM merge commit's branch name** (`revert-5-revert-4-APPS-24484-run-build`) suggests the team has had some build-vs-no-build process churn. Document the agreed approach (commit dist) so future maintainers don't revert it again.
- **`appium-ios-simulator` is in devDeps** — heavyweight. Used in upstream's iOS Karma testing, which LFM doesn't run. Consider dropping.
- **`www/` (Gatsby docs site)** has its own `package.json` and `package-lock.json`. LFM doesn't build or deploy it — it's vendored from upstream. Could be deleted to reduce repo size, but doing so diverges further from upstream.
- **Browser-compat caveats.** The README claims IE9+ / Firefox 3.5+ / Chrome / Safari 6+ / Opera 12+. Reality is that LFM only renders in modern Chrome / Firefox / Safari / Edge. If you fix a bug, prefer `caniuse.com` checks over upstream's claims.

## 16. Glossary

| Term | What it means here |
|---|---|
| `html2canvas` | The library + npm package name + UMD global. |
| `Options` | The single options object the library accepts: union of `CloneOptions`, `WindowOptions`, `RenderOptions`, `ContextOptions`, plus `backgroundColor`, `foreignObjectRendering`, `removeContainer`. |
| `CanvasRenderer` | The default rendering strategy: walks the DOM and draws each box onto an HTMLCanvasElement using 2D context calls. Slow but accurate. |
| `ForeignObjectRenderer` | Alternative rendering strategy using SVG `<foreignObject>`. Faster but has cross-origin restrictions and platform quirks. |
| `DocumentCloner` | Clones the target element + ancestors into a hidden iframe so rendering doesn't disturb the live DOM. |
| `CacheStorage` | Module-level cache for fetched resources (images, fonts) keyed by URL. |
| `proxy` option | URL of an HTTP proxy that can fetch cross-origin images for the library. |
| `data-html2canvas-debug` | Per-element debug attribute (added in v1.3.2) that the library logs about during rendering. |
| `reftests` | Reference image tests: render a known DOM, compare to a committed snapshot byte-for-byte. |

## 17. AI agent guidance (do's and don'ts)

**Do:**

- Treat this as a **vendored fork**. Stay close to upstream; meaningful changes go upstream first.
- When fixing rendering bugs, write a reftest in `tests/reftests/` so the fix is regression-tested.
- Always `npm run build && git add dist/` in the same commit as any `src/` change. Consumers (`lfm-js-commons`) pull this repo via `github:` URL and need `dist/` up to date.
- Mirror upstream coding style (Prettier + ESLint + TypeScript strict).
- When a tile/dashboard export "doesn't look right" in `explorer` / `reporting-redesign`, check this library first — most rendering issues live here, not in the consuming app.

**Don't:**

- Don't `.gitignore` `dist/` — it's the whole point of the fork.
- Don't change `package.json`'s upstream metadata (`author`, `repository.url`, `homepage`) — MIT attribution.
- Don't add LFM-specific business logic. The library should remain a generic upstream port.
- Don't bump TypeScript / Rollup / Babel / Webpack majors without verifying the dist output is byte-functional with the previous build.
- Don't bump Node CI version unless you also verify build output reproducibility.
- Don't delete `www/` or `tests/reftests/` to "trim repo size" — both are upstream contributions and dropping them makes future merges harder.

## 18. Drift markers

- **Last verified against code:** 2026-04-30 by `Claude` (Phase 2 deep read).
- **What I read in full:**
  - Repo root: `README.md`, `CHANGELOG.md` (head), `package.json`, `tsconfig.json`, `rollup.config.ts`, `jest.config.js`, `LICENSE`, `.babelrc`, `.eslintrc`, `.prettierrc`, `.editorconfig`, `.gitignore`, `.npmignore`
  - `.github/workflows/ci.yml` (head), `.github/workflows/release.yml` (full)
  - `src/index.ts` (head, including the `Options` type union and the renderer-selection logic)
  - Full directory listings of `src/`, `dist/`, `docs/`, `examples/`, `configs/`, `scripts/`, `tests/`, `www/`
  - `git log` (only one commit visible due to shallow clone — `98058c5`, the LFM merge)
- **What I did NOT read in full:**
  - 264 other `.ts` source files (only `index.ts` head; `core/`, `css/`, `dom/`, `render/` are upstream-stock and well-documented at https://html2canvas.hertzen.com).
  - 156 `.js` files (mostly built dist output + Gatsby www site + tests).
  - 101 `.html` files (mostly reftest fixtures in `tests/reftests/`).
  - `package-lock.json`, `dist/` bundle bodies (output, not human-readable).
  - `www/` Gatsby site (vendored from upstream; not built by LFM).
- **Sections most likely to drift first:** §6 (when upstream releases a new version and LFM merges it; we'd need to also update `dist/`), §15 (the "single LFM commit" claim — if more LFM patches land).
- **Auto-checks (when wired up):**
  - [ ] `npm run lint` clean
  - [ ] `npm test` green
  - [ ] After every src/ change, `dist/html2canvas.{js,esm.js,min.js}` are committed in the same commit
  - [ ] LFM's `dist/` matches what `npm run build` produces (i.e., consumers can rebuild from source if they want)
  - [ ] no committed credentials in tracked files

---
