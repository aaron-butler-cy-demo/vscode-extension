# AI Coding Rules for vscode-extension

## Project Overview

- This repository is the Cycode VS Code extension, written in TypeScript. `src/extension.ts` is the entry point and is bundled by `webpack.config.js` (using `ts-loader`) into `dist/extension.js`.
- Source is organized by role under `src/`: `cli/`, `commands/`, `listeners/`, `providers/`, `services/`, `ui/`, `utils/`, plus `constants.ts`, `ioc.ts`, and `symbols.ts`.
- Dependency injection uses `tsyringe` and `reflect-metadata`, configured in `src/ioc.ts` and `src/symbols.ts`.

## Webview and HTML Rendering Rules

- WHEN editing the violation panel renderers in `src/ui/panels/violation/renderer/` (`sast.ts`, `sca.ts`, `iac.ts`, `secret.ts`), assign plain values such as descriptions, severities, CWEs and remediation guidelines with `textContent` rather than `innerHTML`.
- WHEN a renderer must display markdown or HTML (for example output produced by `showdown`), sanitize the converted output before inserting it into the DOM. Do not insert detection data from scan results into `innerHTML` unsanitized.
- WHEN creating or editing a webview, keep scripts restricted to what the view needs and do not enable additional capabilities without a clear need.

## CLI Execution Rules

- WHEN editing `src/cli/cli-wrapper.ts`, keep launching the Cycode CLI through `spawn` with an argument array. Do not build a single command string from user-controlled values and do not set `shell: true`.
- WHEN adding code that uses `config.cliPath`, `cycode.additionalParameters`, or any path taken from settings or scan results, pass them as separate arguments and do not concatenate them into shell command strings.
- WHEN using `shelljs`, do not call `exec` with strings that contain user-controlled or scan-result-derived values.

## Secrets and Configuration Rules

- NEVER commit tokens, passwords, API keys or client secrets to source, `package.json` settings defaults, or workflow files.
- WHEN adding new authentication-related values, do not store them in contributed settings in `package.json`. Keep the existing pattern where authentication is handled through the Cycode CLI.
- WHEN adding new contributed settings in `package.json`, follow the existing `cycode.*` naming and include a `description` and `markdownDescription`.

## Dependency and Supply Chain Rules

- WHEN editing `package.json`, preserve the exact-version pinning style used for `dependencies` and `devDependencies`. Do not introduce `^` or `~` ranges for new packages.
- WHEN changing any dependency in `package.json`, update `yarn.lock` in the same change so that `yarn install --frozen-lockfile` (the `ci` script) keeps working.
- Keep `webpack`, `ts-loader`, `typescript`, `eslint`, and the `@typescript-eslint` packages on mutually compatible versions when changing one of them.
- WHEN adding a runtime dependency, make sure it is bundled by webpack and does not require files that are excluded by `.vscodeignore`.

## TypeScript and Code Style Rules

- WHEN writing new code under `src/`, comply with `tsconfig.json` (strict mode, unused locals and unused parameters checks, decorators enabled) and with the rules in `eslint.config.mjs`.
- WHEN adding new injectable services, register and resolve them through the existing `tsyringe` setup in `src/ioc.ts` and `src/symbols.ts` instead of creating ad hoc singletons.
- WHEN adding a new VS Code command, place it in `src/commands/`, and declare it in the `contributes.commands` section of `package.json` with the `Cycode` category and the `cycode.` command prefix.
- WHEN adding listeners, providers or UI components, place them in `src/listeners/`, `src/providers/`, or `src/ui/` respectively, following the existing folder split.
- WHEN changing `package.json` scripts, keep `lint` (`eslint src`), `compile`, `package` and `vscode:prepublish` working with the webpack build.

## CI/CD Workflow Rules

- WHEN editing `.github/workflows/lint_build_publish.yml`, preserve the lint step, the `.vsix` packaging step, and the artifact upload that run on every push.
- WHEN editing that workflow, keep marketplace publishing steps (VS Code Marketplace, Open VSX, GitHub Releases) restricted to `v*` tags.
- NEVER hardcode publishing tokens in the workflow file. Reference them through `${{ secrets.* }}`.

## Prohibited Repo Patterns

- Assigning scan-result or CLI-derived text to `innerHTML` without sanitization
- `spawn` or `exec` calls with `shell: true` or concatenated command strings built from settings or scan data
- Hardcoded credentials or tokens in any committed file
- Adding `^` or `~` version ranges to `package.json` dependencies
- Changing `package.json` dependencies without updating `yarn.lock`
