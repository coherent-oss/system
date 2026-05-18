Migrate a [skeleton](https://github.com/jaraco/skeleton) Python project to the Coherent system.

Goals:
- Replace the project’s bespoke packaging/CI/tooling config with Coherent-managed defaults.
- Preserve the library’s runtime behavior, public API, tests, docs, and entry points.
- Make only the minimum source edits needed to fit Coherent’s conventions.

Please do the migration end-to-end:

1. Packaging/build
- Replace the existing build configuration with a minimal `pyproject.toml` (pull from coherent-oss/coherent.build).
- Preserve any non-import-detectable dependencies by moving them into `__requires__` inside the top-level package module.

2. Package layout
- Align the package layout to the [essential layout](https://github.com/coherent-oss/system/blob/main/essential-layout.md).
  - In particular, move the sole package to the root of the repo.

3. Entry points and metadata
- Preserve console scripts and similar entry points by rendering them in `(meta)/entry_points.txt`.

4. CI and repo config
- Replace the custom GitHub Actions workflow with the reusable Coherent workflow `.github.workflows/main.yml` (pull from coherent-oss/coherent.build).
- Remove local config files that are now redundant because Coherent manages them (for example: tox, pytest, ruff, mypy, coverage, pre-commit, towncrier, Read the Docs, etc.).

5. Docs and readme cleanup
- Remove badges from the README.
- Remove changelog/history wiring (NEWS.rst, docs/history.rst), remove linker config that depended on the NEWS file.

7. Validation
- Run the tests with `coh test` or `pipx run coherent.test`.

Deliverables:
- The actual file changes, staged with Git.
- A short summary of what was removed, what replaced it, and any manual follow-up needed.
- Explicit note of any issues encountered.
