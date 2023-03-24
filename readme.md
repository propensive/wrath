## Wrath

_Wrath_ is a simplistic source-based build script for Scala, written in Bash,
with only limited functionality intended for bootstrapping projects which don't
want to rely upon a more complex tool. It was written primarily to build
[Fury](https://github.com/propensive/fury/), and understands a minimal subset
of Fury builds.

### Running a build

Running a build is typically as simple as running the `wrath` command in a
directory containing a `fury` file,
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
of [CoDL](https://github.com/propensive/codl/). A typical build file contains
a `project declaration`,
```
project myproject
```
and one or more module declarations, for example:
```
project myproject
  module core
  module test
```

A top-level `target` declaration should point to the default target to compile/run
when calling `wrath` without arguments.

A number of top-level `repo` declarations should 

Within a `module`, the following may be defined:
- `main` — the name of the main method to invoke when running (with `wrath -x`)
- `sources` — a space-separate list of source directories
- `lib` — a library dependency; two parameters, an id and a URL
- `include` — a space-separated list of dependencies in the form, `<project>/<module>`
- `plugin` — should be set to `yes` if the module defines a compiler plugin

A more complete module definition might look like this:
```codl

repo acme/mylibrary
target myproject/test

project myproject
  module core
    sources  src/core
    include  mylibrary/core mylibrary/extras
    lib      servlet-api https://repo1.maven.org/maven2/javax/servlet/javax.servlet-api/3.0.1/javax.servlet-api-3.0.1.jar

  module test
    sources  src/test
    include  myproject/core
    main     myproject.Tests
```

### Files

Wrath stores all its working files in a directory called `.wrath` in the
project root directory. This directory can be safely deleted (though
artifacts may need to be downloaded again). All produced artifacts are
stored in a directory called `dist` in the project root directory.

