# AGENTS.md

## Cursor Cloud specific instructions

### Prerequisites

- **Node.js >= 22** (pre-installed via nvm)
- **Yarn 4.12.0** (bundled in `.yarn/releases/`, uses `nodeLinker: node-modules`)
- **Git LFS** (pre-installed; run `git lfs pull` if LFS objects are missing)

### Quick reference

All dev commands, lint/test/build scripts, and architecture rules are documented in `CLAUDE.md` (root) and `.codex/AGENTS.md`. Refer to those instead of duplicating here.

### Running the web app (primary dev target in Cloud)

```bash
yarn app:web          # webpack dev server on http://localhost:3000 (~90s first compile)
```

The initial webpack compilation takes ~60-90 seconds. After the first compile, hot-reload is fast. The web app connects to OneKey's remote API services (no local backend needed).

### Gotchas discovered during setup

- **First `yarn install` may fail transiently** on a fresh VM due to native module builds timing out. Re-running `yarn install` usually succeeds on the second attempt.
- **`yarn install` runs postinstall automatically** which executes `yarn setup:env` (copies `.env.example` to `.env` if missing), `patch-package`, and `copy:inject`. No manual `.env` setup is needed.
- **oxlint has ~32 pre-existing prettier-related errors** in the repository. These are not regressions; `yarn tsc:staged` (type checking) passes cleanly.
- **The web dev server process** appears to exit in the background shell output but actually keeps running as a child webpack-dev-server process. Verify with `curl http://localhost:3000` rather than checking the parent PID.
- **iOS/Android targets** require Xcode/Android SDK which are not available in Cloud VMs. Use `yarn app:web` or `yarn app:ext` for Cloud development.
- **`yarn test`** runs 54 Jest suites (~1261 tests) and takes ~2.5 minutes.
