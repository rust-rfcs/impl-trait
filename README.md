# impl Trait and explicit existential types

This repository tracks the design of the `impl Trait` shorthand along
with explicit syntax for existential types. The proposal has been
evolving for some time and there are a number of historical RFCs. This
repository is also tracking the current state of the feature and its
development.

## Sub-RFCS:

* [introduction of impl trait](impl-trait.md)
* [impl trait in argument position](expand-impl-trait.md)
* [introduction of nameabole impl trait](existential-types.md)
* [type grammar inconsistency fix](https://github.com/rust-lang/rfcs/blob/master/text/2250-finalize-impl-dyn-syntax.md)

# Short explanation of the feature

`impl Trait` is the same syntax for both

* anonymous generic parameters
* inferred return types

E.g. in the function `fn f<T: Trait1>(t: T, u: impl Trait2) -> impl Trait3` you have
one named generic parameter, one anonymous generic parameter and a return type
inferred from the function body.

## Return position `impl Trait`

A function `fn g() -> impl Trait3` means

> There is a concrete type `V` which implements `Trait3` that is the return type of `g`

or in math terms

```
∃ V: Trait3.
    fn f() -> V
```

Note that outside the function you will never know what type has been inferred
as the return type, all you know is that you can interact with the returned value
via the trait `Trait3`.

```rust
fn m() -> impl Trait3;
fn n() -> impl Trait3;

fn bar() {
    let mut y = m();
    y = n(); // ERROR, even if `m` and `n` return the same type, that information is hidden by `impl Trait`
}
```

## Named generic argument

A function `fn h<T: Trait1>(t: T)` means

> For any type `T` that implements `Trait1` there exists a function called `h` that takes an argument of type `T`

or

```
∀ T: Trait1.
    fn h(T)
```

## Anonymous generic parameter (argument position `impl Trait`)

A function `fn i(u: impl Trait2)` means

> For any type `U` that implements `Trait2` there exists a function called `i` that takes an argument of type `U`

or

```
∀ U: Trait2.
    fn i(U)
```

so `fn foo<U: Trait2>(u: U)` and fn `foo(u: impl Trait2)` are exactly the same function except in syntax.

### Argument position `impl Trait` is also existential types

You might have heard this argument and gotten confused.
Well, this argument is technically correct, which is the best kind of correct.
But also very unhelpful for human brains not juggling existential and
universal quantifiers in their head frequently.

The reason this argument exists (pun intended) is that you can move quantifiers into parentheses
similar to how `-(x + y)` can be changed to `-x - y`:

```
∀ U: Trait2.
    fn i(U)
```

is the same thing as


```
fn i(∃ U: Trait2. U)
```

Which in plain english means something along the lines of

> There is a function `i` for which there exists a type U that is its first argument.

I probably got that not quite mathematically right, but explaining that without
symbols breaks my brain. So that's pretty much why mostly you hear that argument
position impl Trait is a `universal quantifier` on the function.

## All of the above

A function `fn f<T: Trait1>(t: T, u: impl Trait2) -> impl Trait3` means

> For any type `T` that implements `Trait1`
> and for any type `U` that implements `Trait2`
> there is a concrete type `V` which implements `Trait3` that is the return type of `f`

or

```
∀ T: Trait1.
    ∀ U: Trait2.
        ∃ V: Trait3.
            fn f(T, U) -> V
```

## Named return position `impl Trait` or "named existential types"

Not yet in nightly, though there's an open PR: https://github.com/rust-lang/rust/pull/52024

```rust
existential type W: Trait4;
fn j() -> W;
```

is equivalent to `fn j() -> impl Trait4`. The only difference is that you can have multiple functions return the *same* `W`:

```rust
existential type W: Trait4;
fn j() -> W;
fn k() -> W;

fn foo() {
    let mut x = j();
    x = k(); // no error, because `j` and `k` return the same type
}
```

meaning

> There is a concrete type `W` which implements `Trait4` that is the return type of `j` and `k`

or

```
∃ W: Trait4.
    fn j() -> W,
    fn k() -> W
```
