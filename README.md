# Coherent Software Development System

The Coherent Software Development System (system) is the implementation of [Jason's vision](https://blog.jaraco.com/vision-for-scalable-OSS-development/) for scalable and sustainable open source development. It builds on decades of experience in the Python ecosystem and working on enterprise software at a global scale.

This user guide provides an overview of the system and directions on getting started with it.

## Overview

The system first and foremost aims to institutionalize and automate the ancillary concerns of software development (packaging, CI, linting, type checking, etc), allowing the developer to focus primarily on their code, tests, and documentation. The system achieves this goal by aggressively adopting and evolving best practices and reflecting configuration from existing state, such as the name of the repo. The system aims to "refactor" the development process such that common concerns are managed through common tools and frameworks rather than through shared config.

The system additionally aims to reduce duplication and unlock composition by adopting a new [*essential layout*](essential-layout.md), allowing the source code as found in the repo to reflect the shape of the code as it will appear when installed.

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

Does the package have dependencies? If so, declare them in `__init__.py` (with [ambitions to auto-detect those from imports](https://github.com/coherent-oss/coherent.build/issues/3)):

```python
__requires__ = [
    "requests",
    "pip-run",
]
```

> [!Tip]
> If you need to extract the `__requires__` list to pass it to pip or other tools,
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

To cut a release, just tag the commit to be released with the v-prefixed version for the release. Push that commit (for good measure), build the release, and then upload the build using [twine](https://pypi.org/project/twine), e.g.:

```
 @ git tag -a v1.0.0 && git push --tags
 @ rm -r dist; coh build && twine upload dist/*
```

## Limitations

As this is a fledgling prototype of a system thusfar, the limitations are to great to enumerate, but follow the [issues](https://github.com/coherent-oss/system/issues) (also [coherent.build issues](https://github.com/coherent-oss/coherent.build/issues) and [coherent.test issues](https://github.com/coherent-oss/coherent.test/issues)) for more.
