# Go Tutorial Stage 2

## Add a library

Move onto the `stage2` directory, where we'll build a new program that
prints your fortune. This program uses a separate Go package as a library that
selects a fortune from a predefined list of messages.

```none
go-tutorial/stage2
├── BUILD
├── MODULE.bazel
├── MODULE.bazel.lock
├── fortune
│   ├── BUILD
│   └── fortune.go
└── print_fortune.go
```

`fortune.go` is the source file for the library. The `fortune` library is a
separate Go package, so its source files are in a separate directory. Bazel
doesn't require you to keep Go packages in separate directories, but it's a
strong convention in the Go ecosystem, and following it will help you stay
compatible with other Go tools.

The `fortune` directory has its own `BUILD` file that tells Bazel how to build
this package. We use `go_library` here instead of `go_binary`.

We also need to set the `importpath` attribute to a string with which the
library can be imported into other Go source files. This name should be the
repository path (or module path) concatenated with the directory within the
repository.

Finally, we need to set the `visibility` attribute to `["//visibility:public"]`.
[`visibility`](https://bazel.build/concepts/visibility) may be set on any
target. It determines which Bazel packages may depend on this target. In our
case, we want any target to be able to depend on this library, so we use the
special value `//visibility:public`.

```bazel
load("@rules_go//go:def.bzl", "go_library")

go_library(
    name = "fortune",
    srcs = ["fortune.go"],
    importpath = "github.com/bazelbuild/examples/go-tutorial/stage2/fortune",
    visibility = ["//visibility:public"],
)
```

You can build this library with:

```posix-shell
bazel build //fortune
```

`print_fortune.go` imports the package using the same string declared in the
`importpath` attribute of the `fortune` library.

We also need to declare this dependency to Bazel. Here's the `BUILD` file in the
`stage2` directory.

```bazel
load("@rules_go//go:def.bzl", "go_binary")

go_binary(
    name = "print_fortune",
    srcs = ["print_fortune.go"],
    deps = ["//fortune"],
)
```

You can run this with the command below.

```posix-shell
bazel run //:print_fortune
```

The `print_fortune` target has a `deps` attribute, a list of other targets that
it depends on. It contains `"//fortune"`, a label string referring to the target
in the `fortune` directory named `fortune`.

Bazel requires that all targets declare their dependencies explicitly with
attributes like `deps`. This may seem cumbersome since dependencies are *also*
specified in source files, but Bazel's explictness gives it an advantage. Bazel
builds an [action graph](https://bazel.build/reference/glossary#action-graph)
containing all commands, inputs, and outputs before running any commands,
without reading any source files. Bazel can then cache action results or send
actions for [remote execution](https://bazel.build/remote/rbe) without built-in
language-specific logic.

### Understanding labels

A [label](https://bazel.build/reference/glossary#label) is a string Bazel uses
to identify a target or a file. Labels are used in command line arguments and in
`BUILD` file attributes like `deps`. We've seen a few already, like `//fortune`,
`//:print-fortune`, and `@rules_go//go:def.bzl`.

A label has three parts: a repository name, a package name, and a target (or
file) name.

The repository name is written between `@` and `//` and is used to refer to a
target from a different Bazel module (for historical reasons, *module* and
*repository* are sometimes used synonymously). In the label,
`@rules_go//go:def.bzl`, the repository name is `rules_go`. The repository name
can be omitted when referring to targets in the same repository.

The package name is written between `//` and `:` and is used to refer to a
target in from a different Bazel package. In the label `@rules_go//go:def.bzl`,
the package name is `go`. A Bazel
[package](https://bazel.build/reference/glossary#package) is a set of files and
targets defined by a `BUILD` or `BUILD.bazel` file in its top-level directory.
Its package name is a slash-separated path from the module root directory
(containing `MODULE.bazel`) to the directory containing the `BUILD` file. A
package may include subdirectories, but only if they don't also contain `BUILD`
files defining their own packages.

Most Go projects have one `BUILD` file per directory and one Go package per
`BUILD` file. The package name in a label may be omitted when referring to
targets in the same directory.

The target name is written after `:` and refers to a target within a package.
The target name may be omitted if it's the same as the last component of the
package name (so `//a/b/c:c` is the same as `//a/b/c`; `//fortune:fortune` is
the same as `//fortune`).

On the command-line, you can use `...` as a wildcard to refer to all the targets
within a package. This is useful for building or testing all the targets in a
repository.

```posix-shell
# Build everything
$ bazel build //...
```
