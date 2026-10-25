# Coherent Software Development System

The Coherent Software Development System (system) is the implementation of [Jason's vision](https://blog.jaraco.com/vision-for-scalable-OSS-development/) for scalable and sustainable open source development in the Python ecosystem. It builds on decades of experience in the ecosystem and working on enterprise software at a global scale.

This README gives an overview of the system. For a hands-on walkthrough — how it works and how to build, test, release, create, and migrate projects — see the [User Guide](user-guide.md). For the rationale behind its conventions and the invariants to preserve, see the [Design guide](design.md).

## Overview

The system first and foremost aims to institutionalize and automate the [ancillary concerns](https://docs.google.com/presentation/d/111mO95s35P3AZzvhMD7uRo0ARgb3GRkuzjqqi8GS48U/edit?slide=id.g3561a517004_0_53#slide=id.g3561a517004_0_53) of software development (packaging, CI, linting, type checking, etc), allowing the developer to focus primarily on their code, tests, and documentation. The system achieves this goal by aggressively adopting and evolving best practices and reflecting configuration from existing state, such as the name of the repo. The system aims to "refactor" the development process such that common concerns are managed through common tools and frameworks rather than through shared config.

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

To create or work on a Coherent project, see the [User Guide](user-guide.md). In short: create a Git repository named for your package, commit your source at the root (starting with `__init__.py`), and build it with `coh build` (from [coherent.cli](https://pypi.org/project/coherent.cli)) — thanks to graceful degradation, that single commit is enough to bootstrap a viable project.

## Agent guidance

Coherent projects carry no `AGENTS.md`. Per-project agent instructions are themselves boilerplate, and they don't scale — every project would need its own copy. Instead, the guidance for working on a Coherent project is maintained once, centrally, as a set of prompts under [`(meta)/agents/`](<(meta)/agents/>) in this repository. Each prompt is brief and points to the generalized [User Guide](user-guide.md) and [Design guide](design.md).

Agents are expected to follow this convention: when working on a project built with the Coherent System, read the prompts under this repository's `(meta)/agents/` directory before making changes, rather than looking for an `AGENTS.md` in the project itself.

## Limitations

Currently, the system supports only pure-Python packages with exactly one root package.

Follow the [issues](https://github.com/coherent-oss/system/issues) (also [coherent.build issues](https://github.com/coherent-oss/coherent.build/issues) and [coherent.test issues](https://github.com/coherent-oss/coherent.test/issues)) for details and please file requests for gaps prevent adoption in your favorite packages.
