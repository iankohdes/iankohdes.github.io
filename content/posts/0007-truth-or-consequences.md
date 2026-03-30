---
title: "Truth or Consequences"
date: 2999-12-31
draft: false
tags: ["rust", "command-line rust"]
series: ["Command-Line Rust"]
series_order: 1
---

{{< alert "circle-info" >}}
This entry, along with the rest of the series, summarises my learnings from Ken Youens-Clark’s _Command-Line Rust_ (2024 updated edition).
{{< /alert >}}

We start with the most basic of functions.

```rust
fn main() {
    println!("Hello, world!");
}
```

Unlike in the book, I will write tests directly in `main.rs`.

## Preliminaries

I have already created a new project by running `cargo new clr-01`. The directory structure by default should look like this:

```text
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
```

`target/` is a directory that contains the build artefacts. `target/debug/`, in particular, contains the `clr-01` binary executable that Cargo compiles and runs.

The default content of `Cargo.toml` looks like this:

```toml
[package]
name = "clr-01"
version = "0.1.0"
edition = "2024"

[dependencies]
```

Notice that the project name, `clr-01`, matches the argument I gave to the `cargo new` command.

## Integration tests

An integration test checks that software components work together. In this case it’s a little contrived, since we only have a single `main` function, but the principle stands.

Below is an example of a test. It always evaluates to `true` (which makes it a pointless test, but illustrates what a successful test looks like).

We add the `#[test]` attribute before _every_ test function. The `assert!` macro checks that an expression evaluates to `true`. The `assert_eq!` macro (not used here) checks that a return value matches an expected value.

A function may have multiple `assert!` or `assert_eq!` calls in a function. However, as soon as the first call fails, the entire test function fails.

```rust
fn main() {
    println!("Hello, world!");
}

#[test]
fn works() {
    assert!(true)
}
```

```bash
> cargo test -q

running 1 test
.
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

{{< alert "circle-info" >}}
We use the `-q` flag to suppress Cargo’s status messages.
{{< /alert >}}

If we replace `assert!(true)` by `assert!(false)`, we see what a failed test looks like.

```zsh
> cargo test -q

running 1 test
works --- FAILED

failures:

---- works stdout ----

thread 'works' (1208128) panicked at src/main.rs:7:5:
assertion failed: false
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    works

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--bin clr-01`
```

## The `PATH` environment variable

Anytime we execute a command, the operating system first searches in a predefined set of directories for a command with that name. The `PATH` environment variable allows us to see these directories. To view these directories separated by newlines, instead of the default colon, we run the following command:

```zsh
echo $PATH | tr : '\n'
```

The `tr` command stands for “translate” and replaces the colon with `"\n"`.

Let’s illustrate the above explanation with an example. This involves creating another test function called `runs()`, to be added after `works()` (defined above).

```rust
#[test]
fn runs() {
    let mut cmd = Command::new("ls");
    let res = cmd.output();
    assert!(res.is_ok());
}
```

{{< alert "comment" >}}
Notice what the documentation says about `Command::new()`: “if `program` [the argument] is not an absolute path [which ours isn’t], the `PATH` will be searched in an OS-defined way.”

The `.output()` method runs the command defined in `.new()`, returning a `Result<Output>`.
{{< /alert >}}

The `runs()` test passes because the `ls` command can be found in one of these predefined set of directories, indicating that `res` is an `Ok` value.

Now let’s make `runs()` fail by replacing `Command::new("ls")` by `Command::new("clr-01")`. We might think that, since `cargo run clr-01` works, `clr-01` on its own might work as well. It doesn’t, and running `clr-01` in the shell tells us why.

```zsh
> clr-01
zsh: command not found: clr-01
```

`clr-01` is not a recognised command because it cannot be found in the `PATH` environment variable (which, as a reminder, contains the predetermined directories that the operating system searches in).

Earlier, I mentioned that the `clr-01` binary executable is stored in `target/debug/`. However, changing to this directory and running `clr-01` will also not work. The reason is that the current working directory is **always excluded** from the `PATH` as a security measure. (This prevents malicious code from being executed in a particular directory.)

## Adding project dependencies

Libraries in Rust are called **crates**. We will add the following crates to our project:

- `assert_cmd` (allows for easier integration testing of command-line interfaces)
- `pretty_assertions` (colour codes diffs to make it easier to spot errors)

The dependencies section of `Cargo.toml` will now look like this:

```toml
[dependencies]
assert_cmd = "2.2.0"
pretty_assertions = "1.4.1"
```

This means we can now refactor `runs()` to use methods from these crates. As such, our source code will now look like this:

```rust
use std::process::Command;
use assert_cmd::assert::OutputAssertExt;
use assert_cmd::cargo::CommandCargoExt;

fn main() {
    println!("Hello, world!");
}

#[test]
fn works() {
    assert!(true)
}

#[test]
fn runs() {
    let mut cmd = Command::cargo_bin("clr-01").unwrap();
    cmd.assert().success();
}
```

`Command::cargo_bin()` creates a `Command` type to run a binary in the current crate, returning a `Result` in case this fails. `.unwrap()` causes a panic if the `Result` is an error value.

`.assert().success()` asserts that the command runs successfully; more generally, `.assert()` asserts the output of some finished process.

The code in this section is just a different way of doing things, compared to the code in the previous section (which relied only on the Rust standard library).

## Program exit values

