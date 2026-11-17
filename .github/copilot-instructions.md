# Copilot instructions

## Project at a glance

`chrome-devtools-mcp` is a TypeScript/Node.js MCP server and CLI that lets coding agents inspect and automate a live Google Chrome/Chrome for Testing browser through Chrome DevTools Protocol and Puppeteer. It also integrates DevTools tracing/Lighthouse for performance analysis. This is a medium-sized npm project (about 11 MB reported GitHub repository size) with a large `third_party/devtools-frontend` submodule; most first-party code is in `src/` and tests in `tests/`.

Use Node.js 24 as pinned by `.nvmrc` (supported engine range is Node `^20.19.0 || ^22.12.0 || >=23`; CI exercises 22, 24, and 26). Use npm 11.10.0 or newer: `.npmrc` configures `min-release-age`, which requires that npm version. TypeScript is ESM; the project uses TypeScript, Node's test runner, Puppeteer, ESLint, Prettier, Rollup, and Zod.

## Bootstrap and validation

Run commands from the repository root, in order:

1. Select Node 24 (`nvm use` where nvm is available), then confirm `node --version` and `npm --version` (npm >=11.10.0).
2. Always install from the lockfile before building: `npm ci`. This runs `scripts/prepare.ts`, which ensures the sparse `third_party/devtools-frontend` checkout and applies a type-declaration workaround in installed dependencies. It needs Git/network access if the submodule is absent. Do not edit `third_party/devtools-frontend`; it is a submodule/mirror.
3. Build/type-check with `npm run build` (`tsc` then `scripts/post-build.ts`). `npm run typecheck` runs `tsc --noEmit` when only type-checking is needed.
4. Run all standard tests with `npm run test` (builds first, then runs tests; third-party notices are excluded). For a focused test, use `npm run test tests/<name>.test.ts`, for example `npm run test tests/McpContext.test.ts`. Tests use Node's test runner and have a 120-second per-test timeout.
5. Run formatting/lint validation with `npm run check-format` (ESLint and Prettier checks). `npm run format` fixes files; inspect its diff before committing.

For browser-dependent tests or the full CI-equivalent package validation, ensure Chrome is installed: `npx puppeteer browsers install chrome`. CI checks out submodules, runs `npm ci` with `PUPPETEER_SKIP_DOWNLOAD=true`, installs Chrome, runs `npm run bundle` with `NODE_OPTIONS=--max_old_space_size=4096`, then runs `npm run test:no-build` (Windows uses test concurrency 1) and `npm run test:notices:no-build`. To reproduce the bundle/notices path locally, run `npm run bundle` before `npm run test:notices:no-build`; `npm run test:notices` itself bundles first. `npm run bundle` cleans and rebuilds `build/`, uses Rollup to bundle dependencies, and creates third-party notices. Build/test entry points and supported flags are defined in `package.json` and `scripts/test.js`.

When changing tool names/descriptions or adding tools, update generated references by running `npm run gen` and include its generated changes. CI's `check-docs` job runs that command and fails if it leaves a diff. Do not run generation just to inspect it: it can modify generated files and formatting.

The repository workflows in `.github/workflows/` include required format/docs checks (`presubmit.yml`), cross-platform Node/browser tests (`run-tests.yml`), and Conventional Commit PR-title validation (`conventional-commit.yml`). PR titles should use Conventional Commit prefixes (`feat:`, `fix:`, `chore:`, etc.). Full tests run on Ubuntu, Windows, and macOS with Node 22/24/26; local success on one platform does not replace CI coverage.

## Where to make changes

- `src/bin/`: MCP server and CLI entry points; `src/index.ts`: package API.
- `src/tools/`: MCP tool definitions and handlers. Keep schemas aligned with handler parsing and generated tool-reference docs. Custom ESLint rules enforce parsed arguments and Zod schema constraints: do not use `.nullable()` or `.object()`; represent complex object results as short formatted strings.
- `src/McpContext.ts`, `src/McpPage.ts`, `src/McpResponse.ts`, `src/BrowserManager.ts`: browser/context/page/response orchestration. Supporting directories include `collectors/`, `processors/`, `formatters/`, `devtools/`, `telemetry/`, `daemon/`, and `utils/`.
- `tests/`: corresponding unit/integration suites, with `tests/tools/` for tool handlers and `tests/mocks.ts` for reusable mocks. Prefer mocked unit tests; use real-browser tests only when browser/DevTools integration is genuinely under test. Use `sinon.createStubInstance`, Sinon assertions, and restore Sinon in `afterEach`.
- `scripts/`: build/post-build, tests, generation, and custom ESLint rules. `rollup.config.js` assembles bundles and validates bundled licenses.
- Root configs: `package.json` and `package-lock.json` (scripts/dependencies), `.nvmrc`/`.npmrc` (runtime/npm requirements), `tsconfig.json`, `eslint.config.js`, `prettier.config.js`, and `puppeteer.config.js`. `docs/` contains tool, configuration, troubleshooting, and design documentation; `CONTRIBUTING.md` has release, generated-doc, and Lighthouse update guidance.

## Coding rules

Follow `AGENTS.md` in addition to this file: no `any`, type assertions (`as`/`!`), or TypeScript suppression comments; prefer `for...of` to `forEach`. Keep tests focused on real behavior and avoid duplicating production logic in mocks. Feature changes should update relevant docs/tool references and skills as applicable. Trust these repository instructions and use further searches only when they are incomplete or demonstrably out of date; consult `AGENTS.md`, `CONTRIBUTING.md`, `package.json`, and the relevant workflow before assuming a command or convention.
