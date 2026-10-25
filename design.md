# Design

This guide explains *why* Coherent projects are shaped the way they are and which invariants to preserve when working on one. For a hands-on walkthrough — building, testing, releasing, and creating projects — see the [User Guide](user-guide.md). For the broader vision, see the [README](README.md) and [Jason's vision](https://blog.jaraco.com/vision-for-scalable-OSS-development/).

## Source is primary

For any given Git repo, the developer's primary concern is the source code, tests, and (maybe) documentation. Everything else — packaging, CI, linting, type checking, release automation — is a *systematic concern* shared across projects and handled by the system rather than by config committed to each repo. Metadata and config concerns that must live in the tree are collected under a separate `(meta)` path, keeping the repo root free for source.

See the [essential layout](essential-layout.md) for how the source in the repo is shaped to match the code as it appears when installed, enabling composition across projects.

## Metadata is derived, not stored

Coherent avoids storing metadata in a static form that drifts out of date. Instead it derives metadata from reusable sources and reflects existing state:

- **Version** — from the SCM (commit history and tags); no hardcoded version. No tags yields an `0.1` prerelease.
- **Dependencies** — inferred from the imports in the source. Requirements that can't be inferred (binary deps, dynamic imports, version constraints) are declared in `__requires__`.
- **Name** — from the repo/root path (a dotted name denotes a namespace package).
- **Description** — from the GitHub project description.
- **Author** — from the commit history.
- **Supported Pythons** — the set of non-EOL Python versions, adapting automatically as versions sunset.
- **License** — the SPDX expression in `__license__`; the full text is resolved and bundled at build time by [coherent.licensed](https://pypi.org/project/coherent.licensed).
- **Typing** — packages are presumed typed, so a PEP 561 `py.typed` marker is emitted automatically. A knowingly untyped package opts out with `__typed__ = False`.

Because these are derived, a single commit of source is enough to bootstrap a viable project. See [discovery.py](https://github.com/coherent-oss/coherent.build/blob/main/discovery.py) for exactly what is discovered and how, and the [User Guide](user-guide.md) for how to declare the few things the system can't infer.

## What not to add

The corollary of the above: do **not** add config that the system already manages or derives. A Coherent project has no `tox.ini`, `setup.cfg`, `ruff.toml`, `mypy.ini`, coverage config, `.pre-commit-config.yaml`, Towncrier config, or Read the Docs config; no `.gitignore` of language/tool/editor artifacts (see [Ignoring artifacts](user-guide.md#ignoring-artifacts)); no duplicated `LICENSE` file (the SPDX grant in metadata plus the bundled text suffice); no hardcoded version or dependency list; and no `AGENTS.md` (see below). Adding any of these reintroduces exactly the boilerplate the system exists to eliminate.

## Composition

By adopting the essential layout, source is shaped consistently with its runtime manifestation, so packages compose cleanly across projects and redundant, repetitive naming is removed within and across projects. Coherent embraces namespace packages: name a repo with the dotted path to the module within the namespace (e.g. `coherent.build`) and the system does the rest.

## Agent guidance lives centrally

Rather than committing an `AGENTS.md` into every project, Coherent keeps a single, central set of agent prompts under [`(meta)/agents/`](<(meta)/agents/>) in this repository. Agents working on any Coherent project are expected to read those prompts, which point back to this guide and the [User Guide](user-guide.md). This keeps the guidance consistent and maintained in one place — and keeps individual project repos free of yet another boilerplate file. See the [README](README.md#agent-guidance) for the convention itself.

## Compromises

In some cases, practicality beats purity, and the Coherent System has had to compromise on its principle of "no boilerplate".

### GitHub workflows

By necessity, GitHub requires a workflow definition for each repo. The Coherent System requires a boilerplate workflow that references a shared, reusable workflow across projects.

### pyproject.toml

While the Coherent System does not require a `pyproject.toml`, many tools including the basic builders and installers require one to know to use the `coherent.build` backend. While Jason has [aspirations to obviate this need](https://hackmd.io/gsTGbh9TRbKFj5Y7nkuNzA), it's still convenient to add a minimal `pyproject.toml` to each repo.
