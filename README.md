# example-percy-maestro

Minimal working example of [`@percy/maestro`](../percy-maestro) for Maestro web flows. Navigates percy.io, captures 2 snapshots, uploads to a Percy Web project with full multi-browser / multi-width rendering.

## Prerequisites

1. **Node 14+** and **Yarn**.
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

Same shape as Selenium/Playwright users' `percy exec -- playwright test`. `percy-maestro exec` handles the DOM capture server + Percy lifecycle internally — nothing else to manage.

When it finishes, `percy exec` prints a build URL. Open it: every snapshot renders in Chrome/Safari/Firefox/Edge at both widths, same review experience as Selenium/Playwright builds.

## Flow file

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

**Note:** `runScript:` is the only thing that differs from Selenium/Playwright usage. Selenium/Playwright let you call `await percySnapshot(driver, name)` directly from your JS test code; Maestro tests are YAML with no user-written JavaScript, so the `runScript:` invocation is unavoidable.

## Troubleshooting

| Symptom | Cause |
|---|---|
| `Unable to locate a Java Runtime` | Install JDK (see Prerequisites) |
| `0 devices connected` | YAML uses `appId:` instead of `url:` for a web flow |
| `Flow file does not exist: .../flows/node_modules/...` | Script path is relative to YAML — use `../node_modules/...` |
| `TypeError: Cannot read property 'PERCY_SNAPSHOT_NAME' of undefined` | Script used `env.VAR` — change to direct global access (`PERCY_SNAPSHOT_NAME`) |
| `Missing required URL for snapshot` | Wrong Percy project type — make sure the project is **Web** |

## License

MIT
