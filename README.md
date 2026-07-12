# Emela Standard Library

The Emela standard library, distributed as a **Pome** (spec 0032).

- **Source path:** `github.com/emela-lang/stdlib`
- **Import root:** `std` (declared by `[pome].module` in `Pome.toml`; without it
  the root would default to the source-path leaf `stdlib`)

A Pome is Emela's unit of distribution: one or more modules supplied as a Git
repository, identified by its source path and versioned by `v`-prefixed semver
git tags. There is no central registry — a Pome is fetched straight from the
repository its source path names.

## Layout

Modules live under `src/`, one file per module. `Pome.toml` at the root declares
the Pome's identity.

```text
stdlib/
  Pome.toml
  src/
    clock.emel
    float.emel
    int.emel
    io.emel
    list.emel
    option.emel
    ord.emel
    result.emel
    string.emel
```

## Use as a dependency

From inside your own Pome:

```sh
emela pome add github:emela-lang/stdlib   # fetch, pin in Pome.lock, audit capabilities
emela build src/main.emel                 # dependencies are on the import path automatically
```

Once it is a dependency, the modules are addressed under the import root `std`.
Effect modules are imported whole and their operations are called qualified
(spec 0036), so `src/io.emel` (`effect io`) is used as:

```emela
import std.io   -- import the io effect; call its operations as io.print(...)

io.print("hi")
```

Pure modules keep per-function imports, so `src/list.emel` (`module list`) is
used as:

```emela
import std.list.map   -- callable as map, list.map, or std.list.map
```

`emela pome add` records the dependency in your `Pome.toml` under its canonical
source path and pins the resolved tag + commit + content hash in `Pome.lock`.
Because a Pome's required capabilities are computable from source (spec 0025),
it also prints the capability set this library requires before committing.

## Modules

- `std.clock` — the `clock` effect: monotonic time (`clock.now`). Requires the
  `clock` capability.
- `std.float` — float helpers (`abs`, `min`, `max`, `sqrt`).
- `std.int` — integer helpers (`abs`, `signum`, `is_even`, `is_odd`, `pow`,
  `gcd`).
- `std.io` — the `io` effect: standard output / error (`io.print`,
  `io.eprint`). Requires the `io` capability.
- `std.list` — the `List<T>` and `Pair<A, B>` types and operations (`length`,
  `is_empty`, `head`, `tail`, `prepend`, `reverse`, `append`, `map`, `filter`,
  `fold`, `contains`, `from_array`, `to_array`, `flat_map`, `zip`, `take`,
  `drop`, `range`).
- `std.option` — combinators over the prelude `Option<T>` type (`is_some`,
  `is_none`, `unwrap_or`, `unwrap_or_else`, `map`, `and_then`, `or`, `or_else`,
  `ok_or`, `filter`, `flatten`).
- `std.ord` — ordering helpers (`min`, `max`, `clamp`).
- `std.result` — the `Result<T, E>` type and combinators (`is_ok`, `is_err`,
  `map`, `map_err`, `and_then`, `unwrap_or`, `ok`, `err`).
- `std.string` — scalar string operations (`length`, `is_empty`, `char_at`,
  `slice`, `chars`).

Side effects enter only through **effects** (spec 0036): `io` and `clock` are
`effect` declarations whose operations wrap **platform functions** (`extern fn`)
resolved by the selected backend's runtime, so app code never names a backend
directly (see `src/io.emel`, `src/clock.emel`).

## Publishing

Publishing is just tagging — there is no central `publish` step:

```sh
git tag v0.1.0 && git push origin v0.1.0
```

Consumers can then depend on `github.com/emela-lang/stdlib` with a version
requirement such as `^0.1`.

## Local development

To resolve a dependency against a local checkout or mirror (offline development,
CI), set `EMELA_POME_REPLACE="github.com/emela-lang/stdlib=/path/to/stdlib"`;
`EMELA_POME_CACHE` redirects the fetch cache.
