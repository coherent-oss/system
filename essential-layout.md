## Overview

The essential layout aims to move the project's core functionality (code) into the root of the repository, giving it primacy over other ancillary concerns (linting, configuration, packaging, licensing) while reducing the redundant layers of directories (repo, "src", namespace, package) and enabling composition of projects using version control.

The essential layout is a key innovation of the [Coherent Software Development System](README.md).


## Motivation

Consider the world's simplest "hello world" package. The classic "hello world" program is simply a file:

```python
print("hello world")
```

But in a world with version control and packaged distributions, the "hello world" library/program suddenly requires a redundant directory and boilerplate:

```shell
$ git init hello-world
$ cd hello-world
$ mkdir hello_world
$ cat > hello_world/__init__.py
print("hello world")
```

The name "hello world" now appears twice (and in two forms). Shortly thereafter, the user adds more ancillary details (a pyproject.toml, a ruff config, testing and coverage config) until an [empty project looks anything but empty](https://github.com/jaraco/skeleton/) and a one-line project like "hello world" is dominated by the ancillary concerns (boilerplate the ocean).

Moreover, since project maintainers want to keep the essential source code separate from from these ancillary concerns, they're motivated to further bury the core functionality in yet another directory:

```shell
$ mkdir src
$ mv hello_world src
```

Note that while the functionality is still no more sophisticated than the canonical "hello world" script, the user is already dealing with a fair bit of complication:

```
hello-world
└── src
    └── hello_world
        └── __init__.py
```

Now someone wanting to bring these projects together into a mono repo or just a tree of code for a larger superproject has a problem - these extra directories and layers of abstraction are an impediment to composing projects because of the repetitive and redundant nature. Consider a superproject containing three dependencies, cssutils, path, and towncrier:

```
superproject
├── cssutils
│   └── cssutils
│       └── __init__.py
├── path
│   └── path
│       └── __init__.py
└── towncrier
    └── src
        └── towncrier
            └── __init__.py
```

The superproject cannot easily represent the code in a shape that reflects the installed layout, meaning that a tooling step is necessary to transform the code from the shape in the repository into the shape in the running environment.

The problem gets even worse when composing projects that share a namespace:

```
superproject
├── cssutils
│   └── cssutils
│       └── __init__.py
├── jaraco.context
│   └── jaraco
│       └── context
│           └── __init__.py
├── jaraco.functools
│   └── jaraco
│       └── functools
│           └── __init__.py
├── path
│   └── path
│       └── __init__.py
└── towncrier
    └── src
        └── towncrier
            └── __init__.py
```

Not only are the `functools` and `context` modules not found in the same `jaraco` directory, but their names are repeated in the repo name and in the namespace.


## Modeling the Essence

The essential layout aims to move the essence of the library (its code) into the root of the repository, such that the repository root _is_ the Python package--no `src` directory and no separate directory for the package:

```
hello_world
├── .git
└── __init__.py
```

Notice now that the name (concern) "hello world" appears exactly once. Renaming the project happens in only one place (instead of multiple places).

## Clone Directly Into a Search Path

Because the repository root _is_ the package directory, a pure package (one that requires no build steps) can be cloned directly into `site-packages`—or any directory on `sys.path` or `PYTHONPATH`—and imported immediately:

```shell
$ cd $(python -c 'import site; print(site.getsitepackages()[0])')
$ git clone https://example.com/hello_world
$ python -c 'import hello_world; print(hello_world.__file__)'
```

No build step, no install step, no transformation from a "repo shape" into an "installed shape." The checkout works as-is because its layout already matches what the import system expects. Editing the code, switching branches, or checking out a different tag takes effect on the next import—the working copy and the importable package are one and the same.


## Essentially Composable

Since the essential layout moves the code into the root of the repository, it allows the root of the repo to represent the directory of the package, so projects compose into a single tree in which the essential functionality is presented at the top and ancillary concerns are tucked away. This enables lots of useful benefits:

- A pure package (something that requires no build steps) can be cloned directly into the site-packages and imported.
- A build system could clone a series of packages into a directory tree, build the respective packages, and run them in place.
- A project can compose a series of dependencies (perhaps declared as VCS references) as an application, perhaps using submodules to track the version relationships.
- Enterprise companies with a monorepo culture can more readily integrate these projects into the mono repo (possibly with direct "submodule"-like references).

For example, the above superproject would look like the following:

```
superproject
├── cssutils
│   └── __init__.py
├── jaraco
│   └── context
│       └── __init__.py
│   └── functools
│       └── __init__.py
├── path
│   └── __init__.py
└── towncrier
    └── __init__.py
```

Note how the structure naturally reflects the shape of the libraries as found when installed and one could readily put `superproject/` on the PYTHONPATH and import any of the projects.


## Tracking Ancillary Details

Although the Coherent Software Development System advises to move most if not all ancillary details into tooling and infrastructure, there are bound to be legitimate details that are most appropriately tracked statically in the repo as part of the code. In particular, concerns that are specific to the project, such as overrides, tooling directives, and customization.

To satisfy this need without the same proliferation of metadata in the root of the project, the essential layout proposes a special directory in the root of the repo named `(meta)`. This one directory is intended to be used for all of the project-specific configs. This directory is named with the unusual characters to make it clearly distinguished from code (it's not importable in Python) and can be readily omitted from installs.

```
superproject
├── cssutils
│   ├── (meta)
│   └── __init__.py
├── jaraco
│   ├── context
│   │   ├── (meta)
│   │   └── __init__.py
│   └── functools
│       ├── (meta)
│       └── __init__.py
├── path
│   ├── (meta)
│   └── __init__.py
└── towncrier
    ├── (meta)
    └── __init__.py
```


## Beyond Python

Although the essential layout is described here in terms of Python packages, the underlying idea is language-agnostic: let the repository root _be_ the unit of code as the ecosystem consumes it, and tuck ancillary concerns out of the way. Any language whose module or namespace resolution is driven by directory structure can benefit.

Consider a .NET project. Today a solution typically carries a nest of `src` directories, intermediate build instructions, and generated artifacts that stand between the source tree and the compiled output. Applying the essential layout, a tree of code could be composed—each library's root holding its source directly—and compiled into a single binary without the indirection of per-project `src` directories and hand-maintained build scaffolding. The shape of the tree would reflect the shape of the resulting assembly, just as the Python tree reflects the installed package layout.


## All Together

Put all together, the essential layout presents a means to collect ancillary details separate from the essential functionality of the project and readily compose superprojects from individual projects.
