# User Guide

The Coherent Software Development System ("the system") lets you maintain a Python package while focusing on just your source code, tests, and docs. The system handles the ancillary concerns — packaging, CI, linting, type checking, releases — centrally, so there's almost no per-project configuration to write or maintain.

This guide covers how the system works, how to work with an existing Coherent project, and how to create a new one. For the rationale behind these conventions, see the [Design guide](design.md); for the bigger picture, see the [README](README.md).

## How it works

A Coherent project is, at minimum, a Git repo whose name is the package name and whose root holds the package source (starting with `__init__.py`). The system derives everything else — version, dependencies, description, author, supported Pythons, license text, typing marker — from that source, the repo, and its history rather than from static config. Metadata and config that must live in the tree are collected under a separate `(meta)` path, keeping the root for source. See the [Design guide](design.md) for the full set of what's derived and why.

You drive the system with the `coh` command from [coherent.cli](https://pypi.org/project/coherent.cli). Install it, or run it ad hoc with `pipx run coherent.cli`. The examples below show both.

## Working with an existing Coherent project

### Build

Produce a source distribution and wheel with:

```shell
 @ coh build
 ...
```

or without installing anything:

```shell
 @ pipx run coherent.cli build
 ...
```

The system uses metadata from the commit history to determine the author name and email, the name of the project from the repo root path, the version from the commit history (no tags means an 0.1 prerelease), the description from the GitHub repo, and supported Pythons from the non-EOL Python versions. See [the source](https://github.com/coherent-oss/coherent.build/blob/main/discovery.py) for more detail on what metadata is discovered and how.

### Test

Create doctests, or a `tests` directory with modules containing pytest tests, then run the tests with `coh test` or `pipx run coherent.test`:

```shell
 @ coh test
 ...
```

### Declaring dependencies

By default, the system infers dependencies from the imports in the source code, so in many cases dependencies are inferred automatically.

In some cases, such as for 'binary' dependencies (i.e. `subprocess.run('tool')`) or for dynamic imports (`import_module('somepkg')`), the system does not yet infer dependencies. Additionally, if there are specific version constraints, those will not be detected. In this case, declare the requirements in `__init__.py` like so:

```python
__requires__ = [
    "requests >= 5.0",
    "pip-run",
]
```

> [!Tip]
> To extract the `__requires__` list to pass it to pip or other tools,
> check [_Extracting Requirements_ in pip-run](https://github.com/jaraco/pip-run#extracting-requirements).

### Declaring a license

To indicate a license, declare a [SPDX license expression](https://spdx.org/licenses/) in the `__license__` attribute of `__init__.py`:

```python
__license__ = "Apache-2.0"
```

The system then publishes that expression in the package metadata and renders the corresponding license text into the distribution, as resolved by [coherent.licensed](https://pypi.org/project/coherent.licensed). Omitting a license indicator will leave the distribution without any indicated license.

By building a project under the Coherent System, both the project's maintainers and its downstream consumers are on notice that the project's code is offered under the terms of the identified license. This declaration — the SPDX expression recorded in the package metadata, together with the full license text bundled into each distribution — constitutes an explicit and authoritative grant of permission under that license, equivalent to a license notice carried within the repository itself. The absence of a duplicated license file in the source repository does not diminish or qualify that grant.

### Typing

Packages are presumed to be typed. Since coherent packages are written with inline annotations, the system emits a [PEP 561](https://peps.python.org/pep-0561/) `py.typed` marker into the built package, so a consumer's type checker honors those annotations without any boilerplate. A package that is knowingly untyped opts out in `__init__.py`:

```python
__typed__ = False
```

> [!Note]
> The marker only takes effect for regular installs. Type checkers resolve it statically against the install location, while an editable install redirects imports at runtime, so annotations from an editable install remain invisible to them ([coherent.build#74](https://github.com/coherent-oss/coherent.build/issues/74)).

### Cutting a release

To cut a release, just tag the commit to be released with the v-prefixed version for the release. The `coh` command provides a handy way to auto-detect the appropriate version based on [semver](https://semver.org/) (`major`, `minor`, `patch`). Then push that commit, where it will be built by CI and released to PyPI. e.g.:

```shell
 @ coh tag major
Created tag v1.0.0
 @ git push
```

### Ignoring artifacts

Coherent projects don't carry a `.gitignore` of language, tool, editor, or OS artifacts — that's a per-environment concern, not a per-project one. Configure those ignores once in your development environment, via a global gitignore referenced from git's `core.excludesfile`, so they apply across every project you work on rather than being committed (or synced into each repo's local excludes) per project. The skeleton's [Ignoring Artifacts](https://blog.jaraco.com/skeleton/skeleton-contrib#ignoring-artifacts) guidance explains the approach and the rationale.

## Creating a new Coherent project

To get started creating a Coherent System project, simply create a Git repository representing the name of the package as it should appear in the Python ecosystem and then commit the source code directly in the root of that project (probably starting with `__init__.py`). If using GitHub and the [`gh` tool](https://cli.github.com/):

```shell
 @ gh repo create --public --description "Library that achieves world peace" --clone worldpeacelib
 @ cd worldpeacelib
 @ echo "print('magic happens here')" > __init__.py
 @ git add __init__.py
 @ git commit -m "Add initial library behavior."
```

As the system employs graceful degradation, that one commit is all that's necessary to bootstrap a viable project; `coh build` will produce an installable distribution from it. A few additions as the project grows:

- **Package data** — have package data? Just commit it to the repo alongside the code, and it's included automatically.
- **Description** — add a `README.md` to the repo to give users an overview, to be included in the project's description.
- **Essential layout** — keep the sole package at the root of the repo so the source reflects the shape of the code as it will appear when installed. See the [essential layout](essential-layout.md).
- **Namespaces** — the Coherent System loves namespace packages. To create a package in a namespace, simply name the repo with the dot-separated path to the module within the namespace. See [coherent.build](https://github.com/coherent-oss/coherent.build) as an example. From [the Zen of Python](https://peps.python.org/pep-0020/):
  > Namespaces are one honking great idea -- let's do more of those!

## Migrating an existing project

Moving a project from the [skeleton](https://github.com/jaraco/skeleton) (or another layout) onto the Coherent System? See [skeleton-migrate.md](skeleton-migrate.md) for a step-by-step migration.
