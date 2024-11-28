# Converting an existing project into Bazel

We need to use another tool available in the official Bazel project called Gazelle (<https://github.com/bazelbuild/bazel-gazelle>).

First, we need to create a `WORKSPACE` file and copy-paste the code from the setup section of the Gazelle repository.

Example:

```bazel
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "io_bazel_rules_go",
    integrity = "sha256-M6zErg9wUC20uJPJ/B3Xqb+ZjCPn/yxFF3QdQEmpdvg=",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/rules_go/releases/download/v0.48.0/rules_go-v0.48.0.zip",
        "https://github.com/bazelbuild/rules_go/releases/download/v0.48.0/rules_go-v0.48.0.zip",
    ],
)

http_archive(
    name = "bazel_gazelle",
    integrity = "sha256-12v3pg/YsFBEQJDfooN6Tq+YKeEWVhjuNdzspcvfWNU=",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/bazel-gazelle/releases/download/v0.37.0/bazel-gazelle-v0.37.0.tar.gz",
        "https://github.com/bazelbuild/bazel-gazelle/releases/download/v0.37.0/bazel-gazelle-v0.37.0.tar.gz",
    ],
)


load("@io_bazel_rules_go//go:deps.bzl", "go_register_toolchains", "go_rules_dependencies")
load("@bazel_gazelle//:deps.bzl", "gazelle_dependencies", "go_repository")

############################################################
# Define your own dependencies here using go_repository.
# Else, dependencies declared by rules_go/gazelle will be used.
# The first declaration of an external repository "wins".
############################################################

go_rules_dependencies()

go_register_toolchains(version = "1.20.5")

gazelle_dependencies()
```

Note: we may need to set `GOPROXY` to default empty and not change `gazelle_dependencies()`.

Now, to actually run Gazelle, we need to add it to our main `BUILD` file.

Example:

```bazel
load("@bazel_gazelle//:def.bzl", "gazelle")

# gazelle:prefix github.com/example/project
gazelle(name = "gazelle")
```

Note: replace `github.com/example/project` with the prefix specification refers to the Go import path used across the project.

At this point, we can actually run Gazelle and let it generate BUILD files for our project.

```sh
bazel run //:gazelle
```

## Hermetic tests

One last problem that you will likely run into is hermetic tests. If you see your tests fail with access denied, file not found or operation not permitted failures, it is because Bazel enforces hermetic tests. This means that each test must be fully self-contained and independent of any other test.

For unit tests, any files need to be provided as dependencies to the test and accessed through the runfiles mechanism (<https://github.com/bazelbuild/rules_go/blob/master/go/tools/bazel/runfiles.go>).

A temporary directory for each test is provided in the TEST_TMPDIR environmental variable instead of the typical os.TempDir().

Hermetic integration and system tests require careful design from the start, so converting existing tests of this type might be tricky. Sadly I do not have a one-size-fits-all recommendation here.

While converting your tests to be hermetic can be annoying, it is a worthwhile effort that will bring your better test repeatability and lower flakiness.
