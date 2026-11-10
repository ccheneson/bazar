# Node / TypeScript

Detected by: `package.json` at the root.

Package manager: `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `bun.lockb` → bun,
otherwise npm. Below, `{pm}` is that manager.

- test_cmd: `{pm} test` (or `{pm} run test`)
- lint_cmd: `{pm} run lint`, plus `{pm} run typecheck` or `npx tsc --noEmit` if there is no typecheck script
- build_cmd: `{pm} run build`
- ui_test_cmd: `{pm} run e2e` if the script exists (optional)
- run_check: start the dev server (`{pm} run dev`) and check in a browser with the
  browser skill. Skip for pure libraries or CLIs and say so in the report.
- dep_file: `package.json` (use `{pm} add`, commit the lockfile)
- allowlist: `{pm} *`, `npx *`

Notes:
- Use only scripts that exist in `package.json`. If one is missing, say so in the baseline instead of inventing it.
- Treat type errors as failures, not just test failures.
