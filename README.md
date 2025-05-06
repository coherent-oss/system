# Coherent Software Development System

The Coherent Software Development System (system) is the implementation of [Jason's vision](https://blog.jaraco.com/vision-for-scalable-OSS-development/) for scalable and sustainable open source development in the Python ecosystem. It builds on decades of experience in the ecosystem and working on enterprise software at a global scale.

This user guide provides an overview of the system and directions on getting started with it.

## Overview

The system first and foremost aims to institutionalize and automate the ancillary concerns of software development (packaging, CI, linting, type checking, etc), allowing the developer to focus primarily on their code, tests, and documentation. The system achieves this goal by aggressively adopting and evolving best practices and reflecting configuration from existing state, such as the name of the repo. The system aims to "refactor" the development process such that common concerns are managed through common tools and frameworks rather than through shared config.

The system additionally aims to reduce duplication and unlock composition by adopting a new [*essential layout*](essential-layout.md), allowing the source code as found in the repo to reflect the shape of the code as it will appear when installed.

## Key Features

This system is characterized by a few key features.

### Source is primary

For any given (Git) repo, the developer's primary concern should be the source code, tests, and (maybe) documentation. The developer is empowered to focus on the design, implementation, and that lifecycle and freed from systematic concerns that apply to many projects. Metadata and other config concerns are collected in a separate `(meta)` path.

### Systematic concerns are shared

The scope of the "system" is flexible, but typically represents the shared concerns around a team or organization or individual. The scope of a system is limited by the willingness of members of that group to align with the vision of the system. For example, some organizations have a highly converged system for managing all software across the organization (monorepo, build infra, deployment infra). Others are more fragmented.

The Open Source community is something of an organization with some shared conventions (Git for source control, GitHub popular for a platform), but also follows a convention of "each repo is their own island." Even systematic solutions like ReadTheDocs require boilerplate copied into each repository. In the Coherent System, most concerns apart from source code development are managed centrally in tooling and infrastructure.

### Metadata is dynamic

Although much metadata is often static, it also does tend to change and evolve over time. The Coherent System aims to avoid storing this metadata in a static form, but instead adapt as assumptions change. For example, the default behavior for "supported Python versions" is that all non-sunset Python versions are supported. When a older version of Python is sunset, the metadata adapts automatically.

### Derive config from reusable sources

As a maintainer of open source projects, developers are asked to replicate config and metadata to multiple locations (source code, GitHub project config, GitHub release notes, published documentation, enterprise support systems). The Coherent philosophy is to ask the developer to define that config and metadata in one place and the any "copies" should reference from there. For example, instead of configuring the project description both in source code metadata and in GitHub's project description, the system resolves the project description from the GitHub project description, allowing it to be consistently defined exactly once. Dependencies are derived from the imports that demand them.

### Projects are composable

By adopting the essential layout, the source code is shaped in a format consistent with its runtime manifestation, allowing it to be readily composed across projects. This approach removes redundant and repetitive naming within and across projects and enables more elegant source code management.

## Getting Started

To get started creating a Coherent System project, simply create a Git repository representing the name of the package as it should appear in the Python ecosystem and then commit the source code directly in the root of that project (probably starting with `__init__.py`). If using the GitHub and the [`gh` tool](https://cli.github.com/):

```shell
 @ gh repo create --public --description "Library that achieves world peace" --clone worldpeacelib
 @ cd worldpeacelib
 @ echo "print('magic happens here')" > __init__.py
 @ git add __init__.py
 @ git commit -m "Add initial library behavior."
```

As the system employs graceful degradation, that one commit is all that's necessary to bootstrap a viable project. To build it, install [coherent.cli](https://pypi.org/project/coherent.cli) or use `pipx run`:

```shell
 @ coh build
 ...
```

or

```shell
 @ pipx run coherent.cli build
 ...
```

The system will use metadata from the commit history to determine the author name and email, the name of the project from the repo root path, the version from the commit history (no tags means an 0.1 prerelease), the description from the GitHub repo, supported Pythons from the non-EOL Python versions, and produce a source distribution and wheel suitable for installation. See [the source](https://github.com/coherent-oss/coherent.build/blob/main/discovery.py) for more detail on what metadata is discovered and how.

Have package data? Just commit it to the repo alongside the code.

Does the package have dependencies? By default, the system will infer dependencies from the imports in the source code, so in many cases, dependencies are inferred automatically.

In some cases, such as for 'binary' dependencies (i.e. `subprocess.run('tool')`) or for dynamic imports (`import_module('somepkg')`), the system does not yet infer dependencies. Additionally, if there are specific version constraints, those will not be detected. In this case, declare the requirements in `__init__.py` as so:

```python
__requires__ = [
    "requests >= 5.0",
    "pip-run",
]
```

> [!Tip]
> To extract the `__requires__` list to pass it to pip or other tools,
> check [_Extracting Requirements_ in pip-run](https://github.com/jaraco/pip-run#extracting-requirements).

Add a README.md to the repo to give users an overview to be included in the project's description.

Ready to create some tests? Create doctests or a `tests` dir with modules containing pytest tests. Then run the tests using `coh test` or `pipx run coherent.cli test`:

```shell
 @ coh test
 ...
```

The Coherent System loves namespace packages. From [the Zen of Python](https://peps.python.org/pep-0020/):

> Namespaces are one honking great idea -- let's do more of those!

To create a package in a namespace, simply name the repo with the dot-separate path to the module within the namespace. See [coherent.build](https://github.com/coherent-oss/coherent.build) as an example.

To cut a release, just tag the commit to be released with the v-prefixed version for the release. The `coh` command provides a handy way to auto-detect the appropriate version based on [semver](https://semver.org/) (`major`, `minor`, `patch`). Then, push that commit, where it will be built by CI and released to PyPI. e.g.:

```shell
 @ coh tag major
Created tag v1.0.0
 @ git push
```

## Limitations

Currently, the system supports only pure-Python packages with exactly one root package.

Follow the [issues](https://github.com/coherent-oss/system/issues) (also [coherent.build issues](https://github.com/coherent-oss/coherent.build/issues) and [coherent.test issues](https://github.com/coherent-oss/coherent.test/issues)) for details and please file requests for gaps prevent adoption in your favorite packages.

## Compromises

In some cases, practicality beats purity, and the Coherent System has had to compromise on its principle of "no boilerplate".

### GitHub workflows

By necessity, GitHub requires a workflow definition for each repo. The Coherent System requires a boilerplate workflow that references a shared workflow across projects.

### pyproject.toml

While the Cohrent System does not require a `pyproject.toml`, many tools including the basic builders and installers require one to know to use the `coherent.build` backend. While Jason has [aspirations to obviate this need](https://hackmd.io/gsTGbh9TRbKFj5Y7nkuNzA), it's still convenient to add a minimal pyproject.toml to each repo.
