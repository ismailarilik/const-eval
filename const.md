# Constants

"Constants" in this document refers to `const` bodies, array sizes, and non-`Copy` array initializers.
On top of what applies to [statics](static.md), they are subject to an additional constraint: In code like
```rust
const CONST: T = EXPR;
```
is supposed to behave as-if `EXPR` was written at every use site of `CONST`.
However, in current versions of rustc, `EXPR` is just computed once and the result is bit-wise copied each time `CONST` is used.
To make this work, we need to ensure [const safety](const_safety.md).

Based on this requirement, we allow other constants and [promoteds](promotion.md) to read from constants.
This is why the value of a `const` is subject to validity checks.

Constants can also be [used in patterns](patterns.md), which brings its own soundness concerns.

## References

One issue is constants of reference type:
```rust
const REF: &u32 = &EXPR;
```
Instead of creating a new allocation for storing the result of `EXPR` on every
use site, we just have a single global "static" allocation and every use of
`REF` uses its address.  It's as if we had written:
```rust
const REF: &u32 = { const _VAL = EXPR; static _STATIC = EXPR; &_STATIC };
```
(`EXPR` is assigned to a `const` first to make it subject to the restrictions
discussed in this document.)

There are various reasons why this could be an issue.

### 1. Pointer equality

We effectively "deduplicate" all the allocations that would otherwise locally be
created at each use site of `REF`.  This is observable when the program compares
these pointers for equality.  We consider this okay, i.e., programs may not rely
on such constants all getting distinct addresses.  They may not rely on them all
getting the same address either.

### 2. Interior mutability

If the reference has type `&Cell<i32>` it is quite clear that the program can
easily observe whether two references point to the same memory even without
comparing their address: Changes through one reference will affect reads through
the other.  So, we cannot allow constant references to types that have interior
mutability (types that are not `Freeze`):

```rust
const BAD: &Cell<i32> = &Cell::new(42);
// Inlining `BAD` everywhere clearly is not the same as them all pointing to the same thing.
```

However, we can do better than that: Even if a *type* is not `Freeze`, it can
have *values* that do not exhibit any interior mutability.  For example, `&None`
at type `&Option<Cell<i32>>` would be rejected by the naive analysis above, but
is actually accepted by the compiler because we know that there is no
`UnsafeCell` here that would permit interior mutability.

*Dynamic check.* The Miri engine enforces this dynamically by ensuring that the
new data that is interned for a constant is all marked as immutable.
(Except for
[data inside promoteds](https://github.com/rust-lang/rust/blob/d538b80ad77949e46989cd355cdec193b574f052/src/librustc_mir/interpret/intern.rs#L363-L367)
which cannot easily be checked, and is thus just *marked* as immutable because
it descends from a shared reference subject to the strict syntactic checks of
lifetime extension.)

Note that a constant *referring to* some already existing mutable memory is
fine: inlining that reference everywhere has the same behavior as computing a
new reference each time.  In both cases, there exists exactly one instance of
the mutable memory that everything references.

### 3. `Send`

Finally, the same constant value is actually shared across threads.  This is
very similar to sending the same value across threads, so it seems like we
should reject non-`Send` types.  For shared references, this means the pointee
type ought to be `Sync`.  That is already required for `static` items, so this
is consistent with the desugaring described above.

However, this does not currently happen, and there are several crates across the
ecosystem that would break if we just started enforcing this now. See
[this issue](https://github.com/rust-lang/rust/issues/49206) and the
[PR attempting to fix this](https://github.com/rust-lang/rust/pull/54424/).

One could make the argument that the value does not have to be `Send` because it
is not actually sent to other threads; instead, conceptually, each thread
re-does the same computation.  But we know they will all come to the same
result.  This works, except when we consider address identity: with references
in the `const`, all threads will get the same address, unlike in case of a
per-thread recomputation which would lead to different addresses. As a
consequence, non-`Send` `const` without references are fine, but once references
and thus address identity comes into play, we have a problem.

*Dynamic check.* It is unclear how the Miri engine could dynamically check this.

### 4. Drop

`Drop` is actually not an issue, at least not more so than for statics:

```rust
struct Foo;

impl Drop for Foo {
    fn drop(&mut self) {
        println!("foo dropped");
    }
}

const FOO: Foo = Foo; // Ok, drop is run at each use site in runtime code

// Not ok, cannot run `Foo::drop` because it's not a const fn
const BAR: i32 = (Foo, 42).1;
```

## Reading statics

Beyond values of reference type, we have to be careful that *computing* a
`const` cannot read from a static that could get mutated (because it is `static
mut`, or because it has interior mutability).  That would lead to the constant
expression (being computed at compile-time) not having the same value any more
when evaluated at run-time.

This is distinct to the concern about interior mutability above: That concern
was about first computing a `&Cell<i32>` and then using it at run-time (and
observing the fact that it has been "deduplicated"), this here is about using
such a value at compile-time even though it might be changed at run-time.

*Dynamic check.* The Miri engine checks this dynamically by refusing to access
global mutable memory, and refusing to dereference any pointer to a static, when
computing a const.
