# once_vec

`once_vec` provides a safe grow-only vector backed by `OnceCell`.

It is designed for cases where you want to append values without taking `&mut self`,
but do not need in-place mutation or removal after insertion. The crate is `no_std`
and only depends on `alloc`.

## Features

- Append-only storage
- Push without a mutable reference
- Immutable once inserted
- Chunked allocation in powers of two
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
