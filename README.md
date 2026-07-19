# Emela Standard Library

The Emela standard library's **pure layer**, distributed as a **Pome** (spec
0032).

- **Source path:** `github.com/emela-lang/stdlib`
- **Import root:** `std` (declared by `[pome].module` in `Pome.toml`; without it
  the root would default to the source-path leaf `stdlib`)
- **Requires:** emela ≥ 0.6 (module-unit imports and first-class effects, spec
  0037; the embedded core landed in 0.5, spec 0038)

A Pome is Emela's unit of distribution: one or more modules supplied as a Git
repository, identified by its source path and versioned by `v`-prefixed semver
git tags. There is no central registry — a Pome is fetched straight from the
repository its source path names.

## The core/std boundary (spec 0038)

This Pome contains **pure Emela only**. Every std module that declares
`intrinsic fn` (spec 0021) or platform `extern fn` (spec 0013) — `core`, `io`,
`clock`, `string`, `float` — is **embedded in the compiler** and resolves with
no dependency at all: those declarations are version-locked to the backends
that supply them, so they ship together. `import std.io` works out of the box;
this Pome provides everything else, and may not (and does not) declare
intrinsics or externs of its own. The embedded module names are reserved: a
package addressed as `std` providing one of them is a compile error.

This layer is where contributions land — new combinators and modules here are
plain Emela code, needing no compiler changes.

## Layout

Modules live under `src/`, one file per module. `Pome.toml` at the root declares
the Pome's identity.

```text
stdlib/
  Pome.toml
  src/
    int.emel
    list.emel
    option.emel
    ord.emel
    result.emel
```

## Use as a dependency

From inside your own Pome:

```sh
emela pome add github:emela-lang/stdlib   # fetch, pin in Pome.lock, audit capabilities
emela build src/main.emel                 # dependencies are on the import path automatically
```

Once it is a dependency, the modules are addressed under the import root `std`,
alongside the embedded ones. Imports name whole modules, and a module's
functions are called in qualified form (spec 0037):

```emela
import std.io         -- embedded in the compiler (spec 0038)
import std.list       -- this Pome; its functions are called as list.<name>

fn main() -> Unit uses { Io } {
    let xs = list.map(list.from_array([1, 2, 3]), fn (x: Int) -> Int { x + 1 })
    Io.print(to_string(list.length(xs)) ++ "\n")
}
```

`emela pome add` records the dependency in your `Pome.toml` under its canonical
source path and pins the resolved tag + commit + content hash in `Pome.lock`.
Because a Pome's required capabilities are computable from source (spec 0025),
it also prints the capability set this library requires before committing —
for this Pome that set is empty: the pure layer performs no effects.

## Modules

- `std.int` — integer helpers (`abs`, `signum`, `is_even`, `is_odd`, `pow`,
  `gcd`).
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

## Publishing

Publishing is just tagging — there is no central `publish` step:

```sh
git tag v0.2.0 && git push origin v0.2.0
```

Consumers can then depend on `github.com/emela-lang/stdlib` with a version
requirement such as `^0.2`.

## Local development

To resolve a dependency against a local checkout or mirror (offline development,
CI), set `EMELA_POME_REPLACE="github.com/emela-lang/stdlib=/path/to/stdlib"`;
`EMELA_POME_CACHE` redirects the fetch cache.
