# Coherent Software Development System

The Coherent Software Development System (system) is the implementation of [Jason's vision](https://blog.jaraco.com/vision-for-scalable-OSS-development/) for scalable and sustainable open source development. It builds on decades of experience in the Python ecosystem and working on enterprise software at a global scale.

This user guide provides an overview of the system and directions on getting started with it.

## Overview

The system first and foremost aims to institutionalize and automate the ancillary concerns of software development (packaging, CI, linting, type checking, etc), allowing the developer to focus primarily on their code, tests, and documentation. The system achieves this goal by aggressively adopting and evolving best practices and reflecting configuration from existing state, such as the name of the repo. The system aims to "refactor" the development process such that common concerns are managed through common tools and frameworks rather than through shared config.

The system additionally aims to reduce duplication by adopting a new *essential layout*, allowing the source code as found in the repo to reflect the shape of the code as it will appear when installed.

## Getting Started

To get started creating a Coherent System project, simply create a Git repository representing the name of the package as it should appear in the Python ecosystem and then commit the source code directly in the root of that project (probably starting with `__init__.py`).

As the system aims to degrade gracefully, that one commit is all that's necessary to build a project. To build it, install [coherent.build](https://pypi.org/project/coherent.build) somewhere and run it or use pip-run to run it:

```shell
 @ pip-run coherent.build -- -m coherent.build
 ...
```

The system will use metadata from your commit history to determine the author name and email, the name of the project from the repo root path, the version from the commit history (no tags means an 0.0.1 prerelease), and produce a source distribution and wheel suitable for installation.

Have package data? Just commit it to the repo alongside the code.

Does the package have dependencies? If so, declare them in `__init__.py`:

```python
__requires__ = [
    "requests",
    "pip-run",
]
```

To add a description to the project, just create the repo in GitHub, set the description, and configure the remote. For example:

```shell
 @ gh repo create --public packagename --description "Awesome package with wings"
 @ git remote add origin https://github.com/$USER/packagename
```

That description will be used as the summary when building the package.

Add a README.md to the repo to give users an overview to be included in the project's description.

Ready to create some tests? Create a `tests` dir with modules containing pytest tests. Then run the tests using [coherent.test](https://pypi.org/project/coherent.test):

```shell
 @ pip-run coherent.test -- -m coherent.test
 ...
```

The Coherent System loves namespace packages. From the Zen of Python:

> Namespaces are one honking great idea -- let's do more of those!

To create a package in a namespace, simply name the repo with the dot-separate path to the module within the namespace. See [coherent.build](https://github.com/coherent-oss/coherent.build) as an example.

To cut a release, just tag the commit to be released with the v-prefixed version for the release. Push that commit (for good measure), build the release, and then upload the build using [twine](https://pypi.org/project/twine).

## Limitations

As this is a fledgling prototype of a system thusfar, the limitations are to great to enumerate, but follow the [issues](https://github.com/coherent-oss/system/issues) for more.
