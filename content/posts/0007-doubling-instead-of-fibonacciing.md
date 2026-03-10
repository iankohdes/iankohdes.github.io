---
title: "Doubling instead of fibonacci-ing"
date: 2026-03-12
draft: false
tags: ["mistakes and learnings", "rust"]
---

This is the first in a set of posts tagged under [“mistakes and learnings”](https://iankohdes.github.io/tags/mistakes-and-learnings/). Here I describe mistakes that I – or others – make, with learnings so I might avoid them in future.

## Introduction

I followed [section 16.5](https://doc.rust-lang.org/rust-by-example/trait/iter.html) of the online book _Rust by Example_, which introduces the [iterator trait](https://doc.rust-lang.org/core/iter/trait.Iterator.html). The section uses the example of the [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_sequence), which we’ve all probably learnt in school. It is an infinite sequence of integers where any given integer is the sum of the two integers immediately before it:

```text
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55
```

The Fibonacci sequence can be constructed with the set of natural numbers (excluding zero), but this example includes zero.

{{< alert "comment" >}}
Iterators are cool because they enable lazy evaluation and prevent the need to allocate memory for intermediate values. They are akin to a list of instructions that are accumulated and run only when required. Python has [iterators](https://docs.python.org/3/glossary.html#term-iterator) too.

To the reader: I treat iterators as assumed knowledge to keep this post concise.
{{< /alert >}}

## Typing along (and introducing a little error)

I find it helpful to type out example code and run it, as it helps me understand what’s going on and I can also modify the code to see what works and what doesn’t. Take a look at [section 16.5’s code](https://doc.rust-lang.org/rust-by-example/trait/iter.html) before looking at what I wrote. (In case the code’s been modified, [here](https://github.com/rust-lang/rust-by-example/commit/fc17a8a6a04076abf382cbfbe97079b51d288189#diff-c7e1dd1858e6f60a1fb483de2a539b96f4b83744bd64f04a1ad7695f79b3d96d) is the most recent commit at the time of writing; search for the file named `iter.md`.)

```rust
fn main() {
    let fib = Fibonacci::init();

    for x in fib.take(10) {
        println!("{}", x);
    }
}

struct Fibonacci {
    curr: u32,
    next: u32,
}

impl Iterator for Fibonacci {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        let current = self.curr;

        self.curr = self.next;
        self.next = self.curr + self.next;

        Some(current)
    }
}

impl Fibonacci {
    fn init() -> Self {
        Fibonacci { curr: 0, next: 1 }
    }
}
```

Now, there is a single error I made while typing out the code. See if you can spot it.

Let me print out the result of running the code as an added clue.

```text
> cargo run
0
1
2
4
8
16
32
64
128
256
```

I was confused over why my Fibonacci sequence turned out to be a geometric one. I wasn’t able to figure out the error until I copied the code line-by-line from section 16.5.

## My mistake

In the definition of the `.next()` method, I typed

```rust
self.next = self.curr + self.next;
            ^^^^^^^^^
```

when I should have typed

```rust
self.next = current + self.next;
            ^^^^^^^
```

After correcting the error, I got the result I expected:

```text
> cargo run
0
1
1
2
3
5
8
13
21
34
```

By adding `self.curr` to `self.next` _after_ `self.curr`’s original value was updated to `self.next`, I essentially added a number to itself (i.e. doubling it), thus the observed erroneous results.

## Learning

I pondered a little over why I made this silly mistake in the first place. After all, I was copying already-written code and yet was able to make a mistake that caused systematically different results.

I noticed eventually that `current` and `self.curr` look quite similar, and that’s probably what led to my error. For clarity, and to aid the reader’s comprehension, perhaps `current` could do with a different name. 

`current_to_return` seems apt. The word _current_ is still there, but there is now extra contextual information to help the reader make sense of the variable.

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

impl Iterator for Fibonacci {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        let current_to_return = self.curr;

        self.curr = self.next;
        self.next = current_to_return + self.next;

        Some(current_to_return)
    }
}
```

Sure, it lengthens the variable name, but we’re not doing maths or competitive programming and can live with the added verbosity.

Therefore, this incident is a reminder to myself to think about whether **visually similar variable names** might end up confusing readers, and to consider choosing different names or add extra contextual information (up to a reasonable point).

## Appendix: making sense of `Fibonacci.next()`

- When we initialise the `Fibonacci` type, our initial state contains the integers `0` and `1`. We’ve not called `.next()`, so there is no integer to return.
- When we call `.next()` for the _very first time_, we are asking for the first value of the sequence, which is `0`.
- This is why the current value of a `Fibonacci` type is returned. `.next()` refers to the next value in the _integer sequence_, not the `next` field.
- Perhaps for even greater clarity, the `curr` and `next` fields could be replaced by `n` and `n_plus_1` respectively.