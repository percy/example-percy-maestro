# example-percy-maestro

Minimal working example of [`@percy/maestro`](../percy-maestro) for Maestro web flows. Navigates percy.io, captures snapshots, uploads to a Percy Web project with full multi-browser / multi-width rendering.

## Prerequisites

1. **Node 16+** and **Yarn**.
2. **Java 17** (Maestro is JVM-based):
   ```sh
   brew install openjdk@17
   ```
3. **Maestro CLI**:
   ```sh
   curl -Ls "https://get.maestro.mobile.dev" | bash
   ```
4. Add to `~/.zshrc`:
   ```sh
   export PATH="$HOME/.maestro/bin:/opt/homebrew/opt/openjdk@17/bin:$PATH"
   export JAVA_HOME="/opt/homebrew/opt/openjdk@17"
   export MAESTRO_CLI_ANALYSIS_NOTIFICATION_DISABLED=true
   ```
5. Percy project (type: **Web**), export its token:
   ```sh
   export PERCY_TOKEN="<your-web-project-token>"
   ```

## Setup

```sh
cd ~/Desktop/percy/example-percy-maestro
yarn install
```

The SDK is linked from `../percy-maestro` via `file:` in `package.json`, so any edit to the SDK source is picked up on the next `yarn install`.

## Run

```sh
yarn test-web
```

which runs:

```sh
percy-maestro exec -- maestro test flows/web-percy.yaml
```

Same shape as Selenium/Playwright users' `percy exec -- playwright test`. `percy-maestro exec` manages the DOM capture server + Percy lifecycle internally.

When it finishes, `percy exec` prints a build URL. Open it: every snapshot renders in Chrome / Safari / Firefox / Edge at both widths, same review experience as Selenium/Playwright builds.

## Project layout

```
example-percy-maestro/
├── .percy.yml             ← project-level defaults (widths, minHeight)
├── package.json           ← deps + test-web script
└── flows/
    └── web-percy.yaml     ← Maestro flow with runScript: snapshot calls
```

### `.percy.yml`

Widths, minHeight, percyCSS, discovery options — the standard Percy config file used by every Percy SDK. Our SDK reads it and applies the `snapshot:*` keys as defaults to every snapshot.

```yaml
version: 2
snapshot:
  widths: [375, 1280]
  minHeight: 1024
```

### Flow file

`flows/web-percy.yaml`:

```yaml
url: https://percy.io
---
- launchApp
- runScript:
    file: ../node_modules/@percy/maestro/scripts/snapshot.js
    env:
      NAME: "percy-home"

- scroll
- runScript:
    file: ../node_modules/@percy/maestro/scripts/snapshot.js
    env:
      NAME: "percy-home-scrolled"
      TEST_CASE: "scroll-suite"
      LABELS: "smoke,scroll"
```

Only `NAME` is required — widths come from `.percy.yml`. Per-snapshot overrides (`TEST_CASE`, `LABELS`, `PERCY_CSS`, `REGIONS`, etc.) appear inline only when they differ from the project default.

### Key conventions

- **`url:`, not `appId:`** — Maestro v2 web driver requires `url:` for web flows.
- **Script path is relative to the YAML file** — `../node_modules/@percy/maestro/scripts/snapshot.js`.
- **Env vars are direct variables in GraalJS** (`NAME`, not `env.NAME`).
- **Short form or long form** — `NAME` and `PERCY_SNAPSHOT_NAME` both work. Pick one and be consistent.

## Why the YAML looks different from Playwright/Selenium tests

Playwright/Selenium users call `await percySnapshot(page, 'Home')` — one line of JavaScript. Maestro tests are YAML with no user-written JavaScript, so Percy is invoked via a `runScript:` block. Four YAML lines to pass a parameterized snapshot name is the minimum Maestro's syntax allows. The rest of the Percy experience (payload, rendering, review UI, diff algorithms, `.percy.yml` config) is identical. See the [main SDK README](../percy-maestro/README.md#why-the-call-site-looks-different-from-playwright--selenium) for the full explanation.

## Supported snapshot options

See the full list in [`@percy/maestro` README](../percy-maestro/README.md#per-snapshot-overrides). Common ones you'll reach for:

- `NAME` (required)
- `WIDTHS` — comma-separated override
- `MIN_HEIGHT` — overrides `.percy.yml`
- `TEST_CASE` — groups snapshots in the review UI
- `LABELS` — comma-separated label chips
- `REGIONS` — JSON array of unified regions
- `PERCY_CSS` — inline CSS injected at render time
- `SCOPE` — CSS selector to capture a subtree only
- `RESPONSIVE` — `"true"` to re-capture DOM at each width

## Troubleshooting

| Symptom | Cause |
|---|---|
| `Unable to locate a Java Runtime` | Install JDK 17 (see Prerequisites) |
| `0 devices connected` | YAML uses `appId:` instead of `url:` for a web flow |
| `Flow file does not exist: .../flows/node_modules/...` | Script path is relative to YAML — use `../node_modules/...` |
| `TypeError: Cannot read property 'NAME' of undefined` | Script used `env.NAME` — change to direct global access (`NAME`) |
| `Missing required URL for snapshot` | Wrong Percy project type — make sure the project is **Web** |

## License

MIT
