# once_vec

[![CI](https://github.com/vaivaswatha/once_vec/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/vaivaswatha/once_vec/actions/workflows/ci.yml)

`once_vec` provides a safe grow-only vector backed by `OnceCell`.

It is designed for cases where you want to append values without taking `&mut self`,
but do not need in-place mutation or removal after insertion. The crate is `no_std`
and only depends on `alloc`.

## Features

- Append-only storage
- Push without a mutable reference
- Immutable once inserted
- Chunked allocation in powers of two
- Efficient access and insertion: `get` is O(1); `push` is amortized O(1)
  (occasional chunk allocation/initialization cost).
  The accesses are not as fast as a `Vec<T>` because of the chunked layout
  (extra index computation), but are still O(1).
- Safe implementation
- `no_std` compatible

## Behavior

`OnceVec<T, N>` stores elements in chunks of size `1, 2, 4, ...`,
up to a maximum length of `2^N - 1`.

- `push` appends a value and returns its index (does not require a mutable reference).
- `get` returns a shared reference to a value by index.
- `iter` yields elements in insertion order.
- `clear` removes all elements and allows reuse of the same container.

Once a value has been pushed, it cannot be replaced or removed individually.
If you need interior mutation, store a type with its own interior mutability.

`OnceVec` does not implement `Sync`: it uses `core::cell` primitives internally,
so shared concurrent access from multiple threads is not supported.

## Example

```rust
use alloc::vec::Vec;
use once_vec::OnceVec;

let values = OnceVec::<&str>::default();

let a = values.push("alpha");
let b = values.push("beta");

assert_eq!(a, 0);
assert_eq!(b, 1);
assert_eq!(values.get(0), Some(&"alpha"));
assert_eq!(values.get(1), Some(&"beta"));

let collected: Vec<_> = values.iter().copied().collect();
assert_eq!(collected, vec!["alpha", "beta"]);
```

## Capacity

The default chunk count is `32`, which gives a theoretical maximum length of `2^32 - 1`
elements on 32-bit and larger targets, subject to available memory.

You can choose a smaller `N` when constructing the type if you want a lower maximum capacity.

## Comparison

### vs [`elsa::FrozenVec`](https://github.com/Manishearth/elsa)

- `FrozenVec` uses `stable_deref_trait::StableDeref` for the API that returns
  stable references to inserted items (`&T::Target`), which typically means
  storing values behind indirection (for example `Box<T>`, `Arc<T>`, etc.).
- Implementation includes `unsafe` internally (for example via `UnsafeCell`
  access patterns in `src/vec.rs`).
- `once_vec` is fully safe Rust and stores plain `T` directly.

### vs [`append-only-vec`](https://github.com/droundy/append-only-vec)

- Similar high-level approach (chunked growth with O(1) indexing and
  amortized O(1) append), including non-`&mut` append semantics.
- Implementation relies on `unsafe` (raw pointers, `UnsafeCell`, manual
  allocation/deallocation) in `src/lib.rs`.
- `once_vec` aims for the same usage style without `unsafe`.

### vs [`appendlist`](https://github.com/danieldulaney/appendlist/blob/master/src/appendlist.rs)

- Uses a `Vec<Vec<T>>` chunk layout and explicitly documents an unsafe-internal
  implementation strategy (for example with `UnsafeCell<Vec<Vec<T>>>`).
- Also provides O(1) indexing and append behavior with chunked growth.
- `once_vec` provides a similar append-only ergonomics with a fully safe,
  `OnceCell`-based implementation.
