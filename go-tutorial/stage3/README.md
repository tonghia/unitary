# Go Tutorial Stage 3

## Test your project

Next, move to the `stage3` directory, where we'll add a test.

```none
go-tutorial/stage3
├── BUILD
├── MODULE.bazel
├── MODULE.bazel.lock
├── fortune
│   ├── BUILD
│   ├── fortune.go
│   └── fortune_test.go
└── print-fortune.go
```

`fortune/fortune_test.go` is our new test source file.

This file uses the unexported `fortunes` variable, so it needs to be compiled
into the same Go package as `fortune.go`. Look at the `BUILD` file to see
how that works:

```bazel
load("@rules_go//go:def.bzl", "go_library", "go_test")

go_library(
    name = "fortune",
    srcs = ["fortune.go"],
    importpath = "github.com/bazelbuild/examples/go-tutorial/stage3/fortune",
    visibility = ["//visibility:public"],
)

go_test(
    name = "fortune_test",
    srcs = ["fortune_test.go"],
    embed = [":fortune"],
)
```

We have a new `fortune_test` target that uses the `go_test` rule to compile and
link a test executable. `go_test` needs to compile `fortune.go` and
`fortune_test.go` together with the same command, so we use the `embed`
attribute here to incorporate the attributes of the `fortune` target into
`fortune_test`. `embed` is most commonly used with `go_test` and `go_binary`,
but it also works with `go_library`, which is sometimes useful for generated
code.

You may be wondering if the `embed` attribute is related to Go's
[`embed`](https://pkg.go.dev/embed) package, which is used to access data files
copied into an executable. This is an unfortunate name collision: rules_go's
`embed` attribute was introduced before Go's `embed` package. Instead, rules_go
uses the `embedsrcs` to list files that can be loaded with the `embed` package.

Try running our test with `bazel test`:

```posix-shell
$ bazel test //fortune:fortune_test
INFO: Analyzed target //fortune:fortune_test (0 packages loaded, 0 targets configured).
INFO: Found 1 test target...
Target //fortune:fortune_test up-to-date:
  bazel-bin/fortune/fortune_test_/fortune_test
INFO: Elapsed time: 0.168s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
//fortune:fortune_test                                          PASSED in 0.3s

Executed 0 out of 1 test: 1 test passes.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option to see which ones these are.
```

You can use the `...` wildcard to run all tests. Bazel will also build targets
that aren't tests, so this can catch compile errors even in packages that don't
have tests.

```posix-shell
bazel test //...
```
