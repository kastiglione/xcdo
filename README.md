# `xcdo`

`xcdo` automatically adds necessary or helpful compiler flags to `swiftc` and `clang`. Inspired by `xcrun`, which sets essential environment variables. `xcdo` is intended for exploratory use, not from a build system.

## Usage

Use `xcdo` instead of `xcrun` when running `swiftc` or `clang`. It's possible to alias `xcrun` to `xcdo`.

Use the `-###` flag to show, but not execute, the underlying `xcrun` command. For example:

```sh
xcdo -### -sdk iphoneos clang++ -c source.cpp
```

will print:

```sh
xcrun -sdk iphoneos clang++ -target arm64-apple-ios13.0 -std=c++17 -Wall -c source.cpp
```

## Features

### `swiftc` and `clang`

#### Better Target Triple

`xcrun` sets the correct sysroot/sdk, but not the correct target triple. Both `swiftc` and `clang` assume compilation for the current host, which for macOS is `x86_64`. When building for other SDKs, the correct target is needed as well.

`xcdo` determines which [target `-triple`](https://clang.llvm.org/docs/CrossCompilation.html#target-triple) to use based on the `-sdk` flag. The mapping is:

| SDK | `xcrun` | `xcdo` |
| --- | --- | --- |
| `iphoneos` | `x86_64-apple-ios13.0.0` | **`arm64-apple-ios13.0`**  |
| `iphonesimulator` | `x86_64-apple-ios13.0.0-simulator` | `x86_64-apple-ios13.0-simulator` |
| `macosx` | `x86_64-apple-macosx10.14.0` | `x86_64-apple-macosx10.14` |

#### Preserve Temporary Files

Using `clang -v` or `swiftc -v` prints the underlying commands being run. These command lines often contain temp files, but they are deleted by at the end of the command. `xcdo` adds `-save-temps` whenever `-v` is given, to allow the temp files to be inspected.

### `swiftc` Specific

#### Compilation Mode

Unless `-whole-module-optimization` is used, `xcdo` adds `-enable-batch-mode`.

#### Module Interfaces and Library Evolution

If `-emit-module-interface` is specified, then `-enable-library-evolution` is added if not present, since it is required to produce `.swiftinterface` files.

#### Incremental Compilation

Swift incremental compilation requires a JSON output-file-map. When using `-incremental`, `xcdo` will generate an output-file-map if none is given. Also, `-driver-show-incremental` is added.

See Swift's documentation [Driver: Incremental Builds](https://github.com/apple/swift/blob/master/docs/Driver.md#incremental-builds) for details on swift incremental builds.

### `clang` Specific

#### Language Standard

`clang` defaults to `-std=c++98`, which is quite old. `xcdo` defaults to `-std=c++17` when no `-std` flag is used.

#### Warnings

`xcdo` adds `-Wall` when no other `-W` flag is used.

#### Objective-C

`clang` does not enable ARC by default. `xcdo` enables ARC when compiling objc/objc++ sources by adding `-fobjc-arc`.

#### Assembly

Generated assembly can be noisy. Running `clang -S` with `xcdo` will add `-fomit-frame-pointer` and `-fno-unwind-tables`. Thanks to Greg Parker for sharing this tip https://twitter.com/gparker/status/1147723601656729600.
