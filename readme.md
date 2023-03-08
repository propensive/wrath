## Wrath

_Wrath_ is a simplistic source-based build script for Scala, written in Bash,
with only limited functionality intended for bootstrapping projects which don't
want to rely upon a more complex tool. It was written primarily to build
[Fury](https://github.com/propensive/fury/).

### Running a build

Running a build is typically as simple as running the `wrath` command in a
directory containing a `build.wrath` file,
```sh
wrath
```
which will compile the default module (if there is one), or offer advice on any
additional parameters that may be required.

Several other parameters can also be specified:
- `--target <module>` (`-t`) — specify a different target module
- `--exec` (`-x`) — execute the main method for the target module
- `--repl` (`-r`) — launch the REPL with the target module on the classpath
- `--clean` (`-c`) — clean the target module before rebuilding
- `--deep-clean` (`-c`) — clean and rebuild all sources
- `--fetch` (`-f`) — fetch any source repositories required by the build
- `--fetch-all` (`-F`) — fetch the Scala compiler as well
- `--main <class>` (`-m`) — specify the main class to invoke (with `-x`)
- `--jdk <dir>` (`-j`) — specify a different `JAVA_HOME` directory
- `--help` (`-h`) — print a usage information

### Dependencies

Dependent repositories are referenced as subdirectories of the project root
directory. Running Wrath with the `-f` parameter will fetch any missing
dependencies, but symbolic links can be used to point Wrath to a directory
elsewhere on disk.

The Scala compiler is similarly a dependency, and will be kept in the `scala`
subdirectory. It's common to want to use a single copy of the Scala compiler
for all projects, and is most easily specified with a symlink. Scala will
only be fetched from GitHub if the `-F` option is specified. This may be
useful in CI environments.

### Configuration files

Configuration files are called `build.wrath` and are written in a simple form
of [TOML](https://toml.io/). They consist of _build sections_, such as,
```
[omega]
```
and _module sections_, such as,
```
[omega/app]
```
distinguished by the absence or presence of a `/`.

A build section defines parameters for the whole build, specifically,
- `repos` — a space-separated list of [GitHub](https://github.com/) repositories in the form `<group>/<repo>`,
- `target` — the default module to compile (and maybe run), in the form `<build>/<module>`.

For example:
```
[omega]
repos = propensive/rudiments omega/omegadb
target = omega/app
```

Under a module section, the following parameters may be used:
- `src` — a single source directory containing .scala files
- `main` — the default main class to execute for this module
- `lib` — a space-separated list of URLs to be downloaded and included on the classpath
- `refs` — a space-separated list of module dependencies, in the form `<build>/<module>`

For example:
```
[omega/app]
refs = rudiments/core omegadb/core
src = src/scala/core
main = omega.Main
lib = https://example.com/library-dependency.jar https://example.com/another-dependency.jar
```

### Files

Wrath stores all its working files in a directory called `.wrath` in the
project root directory. This directory can be safely deleted. All produced
artifacts are stored in a directory called `dist` in the project root
directory.

