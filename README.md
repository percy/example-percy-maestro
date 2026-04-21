# example-percy-maestro

Example repo showing full Percy visual testing integration with the [Maestro](https://docs.maestro.dev) framework for **web flows**, using [`@percy/maestro`](../percy-maestro). Uses DOM capture under the hood — same multi-browser, multi-width rendering as `percy-selenium` and `percy-playwright`.

## Prerequisites

1. **Node 14+** and **Yarn**.
2. **Java 17** (Maestro requires a JDK):
   ```sh
   brew install openjdk@17
   ```
   Add to `~/.zshrc`:
   ```sh
   export PATH="$HOME/.maestro/bin:/opt/homebrew/opt/openjdk@17/bin:$PATH"
   export JAVA_HOME="/opt/homebrew/opt/openjdk@17"
   export MAESTRO_CLI_ANALYSIS_NOTIFICATION_DISABLED=true
   ```
3. **Maestro CLI**:
   ```sh
   curl -Ls "https://get.maestro.mobile.dev" | bash
   ```
4. **Percy project** (type: Web) and its token:
   ```sh
   export PERCY_TOKEN="your-web-project-token"
   ```

## Dev-test setup

This repo expects `@percy/maestro` to live in a sibling directory:

```
~/Desktop/percy/
├── percy-maestro/            ← the SDK under development
└── example-percy-maestro/    ← this repo
```

`package.json` references `"@percy/maestro": "file:../percy-maestro"` so SDK edits are picked up without a republish.

```sh
cd ~/Desktop/percy/example-percy-maestro
yarn install
```

After editing any SDK source, re-run `yarn install` (or `rm -rf node_modules && yarn install`) to refresh the symlinked copy.

## Run

```sh
yarn test-web
```

This runs:

```
percy exec -- bash -c '
  percy-maestro serve &                      # aux capture server on :5339
  sleep 2
  maestro test flows/web-percy.yaml          # Maestro drives Chromium, runScript posts snapshots
  kill $!                                    # stop capture server
'
```

After it finishes, `percy exec` prints a build URL. Open it to review — snapshots render across Safari/Chrome/Firefox/Edge at every configured width.

## Flow structure

`flows/web-percy.yaml`:

```yaml
url: https://percy.io
---
- launchApp
- runScript:
    file: ../node_modules/@percy/maestro/scripts/snapshot.js
    env:
      PERCY_SNAPSHOT_NAME: "percy-home"
      PERCY_SNAPSHOT_WIDTHS: "375,1280"

- scroll
- runScript:
    file: ../node_modules/@percy/maestro/scripts/snapshot.js
    env:
      PERCY_SNAPSHOT_NAME: "percy-home-scrolled"
      PERCY_SNAPSHOT_WIDTHS: "375,1280"
```

### Key conventions

- **`url:`, not `appId:`** — Maestro v2 web driver requires `url:` for web flows.
- **Script path is relative to the YAML file** — use `../node_modules/@percy/maestro/scripts/snapshot.js`, not absolute or CWD-relative.
- **Env vars are accessed as direct variables in the `runScript` script** (`PERCY_SNAPSHOT_NAME`), not `env.PERCY_SNAPSHOT_NAME`.

## Supported snapshot options

Pass via the `env:` block of a `runScript:` step. See the full list in [`@percy/maestro` README](../percy-maestro/README.md#supported-options).

Common ones:
- `PERCY_SNAPSHOT_NAME` (required)
- `PERCY_SNAPSHOT_WIDTHS` — e.g. `"375,1280"`
- `PERCY_SNAPSHOT_MIN_HEIGHT` — minimum render height
- `PERCY_SNAPSHOT_IGNORE_REGIONS` — JSON array of region objects
- `PERCY_SNAPSHOT_CONSIDER_REGIONS` — JSON array
- `PERCY_SNAPSHOT_RESPONSIVE` — `"true"` to re-capture DOM at each width

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `Unable to locate a Java Runtime` | Install JDK (see Prerequisites) |
| `0 devices connected` | Your YAML uses `appId:` instead of `url:` for a web flow |
| `Flow file does not exist: .../flows/node_modules/...` | Script path is relative to YAML — use `../node_modules/...` |
| `TypeError: Cannot read property 'PERCY_SNAPSHOT_NAME' of undefined` | Script used `env.VAR` — change to direct variable access (just `PERCY_SNAPSHOT_NAME`) |
| `Missing required URL for snapshot` | You're using the screenshot-upload path (`percy-maestro upload`) against a Web-type Percy project — use the DOM-capture path (`percy-maestro serve` + `runScript`) instead |

## License

MIT
