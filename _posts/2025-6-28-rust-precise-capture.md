---
layout: article
title: Factory with referenced parameters in Rust
key: rust-precise-capture
tags: Rust
---

Precise capture comes to the rescue!

<!-- more -->

Occasionally in Rust, we'd like to write something this:

```rust
trait T {}

impl T for () {}

fn factory(x: &i32) -> impl T {
    ()
}

fn foo(x: &i32) -> Arc<dyn T> {
    Arc::new(factory(x))
}
```

We'd like to use a factory function to factory trait objects whose concrete type can only be determined at runtime. Additionally, we use references to pass parameters for construction, where `x` references some variable with arbitrary lifetime, e.g. some complex configuration.

However, the code above does not compile. Rust will complain in the calling function `foo` that `x` does not have static lifetime. How come?

The situation here is that Rust cannot determine the lifetime of the returned `Arc<dyn T>` at compile time, which is why we use `Arc` in the first place. Rust then, in the name of memory safety, assumes that `x` may be part of the returned variable as is, and therefore it must outlive `Arc`, which essentially requires `x` to be static.

Sometimes Rust will prompt you to mark the lifetime of the returned value as `dyn T + '_`, so it shares the same lifetime as `x` by default. However, this approach is not practical when the call stack is deep, or when we have many parameters by reference, or in async where Rust basically gave up reasoning about lifetimes (since no one knows when an async block will end, if it ends at all), or maybe we just want to use the constructed object anywhere I wish it to be.

A more straightforward way to solve this problem, also conforming to our factory semantics, is to explicitly tell Rust that `x` will not end up in the returned value and will only be used inside the factory function. We use the precise capture syntax to achieve this:

```rust
fn factory(x: &i32) -> impl T + use<> {
```

The empty `use<>` states that all implementations of `T` returned by `factory` will have nothing to do with any lifetime implied by the reference(s) passed as parameters, i.e. it precisely captures none of the lifetimes implied by `factory`.
