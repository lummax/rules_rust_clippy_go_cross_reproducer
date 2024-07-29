# rules_rust_clippy_go_cross_reproducer
Reproducer for an issue with `rust_clippy_aspect` in combination with `go_cross_binary()`.

## To reproduce (on Linux/Mac):

- This builds fine:

```
bazel build //...
```

```
bazel build //rust/... --aspects=@rules_rust//rust:defs.bzl%rust_clippy_aspect --output_groups=+clippy_checks
```

- But this yields an error:

```
bazel build //... --aspects=@rules_rust//rust:defs.bzl%rust_clippy_aspect --output_groups=+clippy_checks
Use --verbose_failures to see the command lines of failed build steps.
ERROR: Analysis of aspects '[@@rules_rust~//rust:defs.bzl%rust_clippy_aspect] with parameters {} on //go:windows_x64' failed; build aborted: No matching toolchains found for types @@rules_rust~//rust:toolchain_type, @@bazel_tools//tools/cpp:toolchain_type.
To debug, rerun with --toolchain_resolution_debug='@@rules_rust~//rust:toolchain_type|@@bazel_tools//tools/cpp:toolchain_type'
If platforms or toolchains are a new concept for you, we'd encourage reading https://bazel.build/concepts/platforms-intro.
INFO: Elapsed time: 0.480s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
ERROR: Build did NOT complete successfully
```

## Analysis

This is due to the `rust_clippy_aspect` aspect requiring a C++/Rust toolchain for the aspect
execution. `rules_go`'s `go_cross_binary()` transitions the target platform which results in
a requirement for a C++/Rust toolchain that is compatible.

With `bazel`'s optional toolchains we can solve this issue.
See:
- https://github.com/bazelbuild/proposals/blob/main/designs/2022-01-21-optional-toolchains.md
- https://github.com/bazelbuild/bazel/issues/16966
    - 7.0.0: https://github.com/bazelbuild/bazel/commit/6b126d60c319c574ed598eb38c0bdcc6b10fde03
    - 6.1.0: https://github.com/bazelbuild/bazel/commit/ac47e1085c59654f29b82b0ca3be224447895774