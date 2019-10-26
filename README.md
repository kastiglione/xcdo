# `xcdo`

`xcrun` provides some basic defaults, and `xcdo` adds more defaults. Intended for manual (exploratory) use, not from a build system.

## Usage

Use `xcdo` instead of `xcrun` when running `swiftc` or `clang`. It's possible to alias `xcrun` to `xcdo`.

Use the `-###` flag to show, but not execute, the underlying `xcrun` command. For example:

```sh
xcdo -### -sdk iphoneos clang++ -c source.cpp
```

Which prints:

```sh
/usr/bin/xcrun -sdk iphoneos clang++ -target arm64-apple-ios13.0 -std=c++17 -Wall -c source.cpp
```

## `xcrun` Defaults

`xcrun` sets up the SDK for the compiler, via the `SDKROOT` environment variable. When using `xcrun -sdk <SDK>`, both `clang` or and `swiftc` will use the `SDKROOT` environment variable to set the `clang -isysroot` flag, or the `swiftc -sdk` flag. Here's a table showing the results:

| SDK | `swift -sdk` / `clang -isysroot` |
| --- | --- |
| `iphoneos` | `iPhoneOS.platform/Developer/SDKs/iPhoneOS13.0.sdk` |
| `iphonesimulator` | `iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator13.0.sdk` |
| `macosx` | `MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk` |

## `xcdo` Defaults

`xcdo` is a wrapper for `xcrun` which adds more defaults.

### Correct `-target`

Unfortunately, `xcrun` assumes the target arch is `x86_64`, or whatever arch your machine uses. `xcdo` picks a compiler `-target` that matches the given `-sdk`.

`xcdo` determines which [target `-triple`](https://clang.llvm.org/docs/CrossCompilation.html#target-triple) to use based on the `xcrun -sdk` flag. The mapping is:

| SDK | `xcrun` Target | `xcdo` Target |
| --- | --- | --- |
| `iphoneos` | `x86_64-apple-ios13.0.0` | **`arm64-apple-ios13.0`**  |
| `iphonesimulator` | `x86_64-apple-ios13.0.0-simulator` | `x86_64-apple-ios13.0-simulator` |
| `macosx` | `x86_64-apple-macosx10.14.0` | `x86_64-apple-macosx10.14` |

### Defaults for `swiftc`

#### `swiftc` Compilation Mode

Unless `-whole-module-optimization` is used, `xcdo` adds `-enable-batch-mode`.

#### Incremental Compilation

Swift incremental compilation requires a JSON output-file-map. Running `xcdo swiftc -incremental` will supply an `-output-file-map` if none is given. This output-file-map is automatically generated. Also, `-driver-show-incremental` is added.

See Swift's documentation [Driver: Incremental Builds](https://github.com/apple/swift/blob/master/docs/Driver.md#incremental-builds) for details on swift incremental builds.

### Defaults for `clang` 

#### Language Standard

`clang` defaults to `-std=c++98`, but for interactive use the default should be a modern standard. `xcdo` defaults to `-std=c++17` when no `-std` flag is used.

#### Warnings

`xcdo` adds `-Wall` when no other `-W` flag is used.

#### Objective-C

`clang` does not enable ARC by default. `xcdo` enables ARC when compiling objc/objc++ sources by adding `-fobjc-arc`.

#### Assembly

Generated assembly can be noisey. Running `clang -S` with `xcdo` will add `-fomit-frame-pointer` and `-fno-unwind-tables`. Thanks to Greg Parker for sharing this tip https://twitter.com/gparker/status/1147723601656729600.
