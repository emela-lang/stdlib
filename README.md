# Emela Standard Library

This directory contains the first source-layout sketch of the Emela standard
library.

The public API is written in Emela under `std/`. Backend-specific platform
extern declarations live in backend manifests.

Current compiler limitations:

- The resolver expands only the requested stdlib API and the dependencies it
  calls. Backends do not need to implement unused platform imports from the same
  stdlib module.
- There is no alias import yet. The stdlib uses private platform extern names
  such as `platform.io._write_stdout_utf8!` so the public wrapper can be named
  `write_stdout_utf8!`.
- Native bindings call C ABI runtime symbols. Runtime implementations live
  under `compiler/backends/`.

## Layout

```text
stdlib/
  std/
    io.emel
    clock.emel
```

## Public API

- `std.io.write_stdout_utf8!(value: String) -> Result<Unit, PlatformError>`
- `std.io.read_stdin_utf8!() -> Result<String, PlatformError>`
- `std.clock.now_i32!() -> I32`

## Native

The built-in native backend binds private platform externs to C ABI runtime
symbols. Those symbols are implemented by runtime C files under
`compiler/backends/`.

## JavaScript

The built-in JavaScript backend calls runtime symbols implemented by
`compiler/backends/js-*/runtime.js`.
