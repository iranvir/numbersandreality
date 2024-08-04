# Installing Rust Lang on Alpine Linux
Setting up Rust Lang on Alpine is relatively straight-forward. You get the
dependencies that includes clang and lld from the apk repositories and curl to
download and install the [Rust
Toolchain](https://www.rust-lang.org/tools/install).

> UPDATE: Since the introduction of rust-lld within the rust toolchain, we have
the option to use `rust-lld` as the default linker and not install system's
default C compiler and Linker. This method is documented after section 4.

## 1. Getting the dependencies
We will need a C compiler and a Linker. Actually only the linker will be used to
link your binaries against the standard C library unless your Rust project is
explicitly called C programs. I will be using the LLVM Project's `lld` linker
and `clang` compiler.
```
apk add clang lld
```
I would also need curl to actually download the Rust comiler toolchain in the
second step.
## 2. Getting Rust
I will just use the standard Rust installation method as shown in the official
website.
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
I will go ahead and select all the defaults, which in case of Alpine will look
something like below:
```
Current installation options:

default host triple: x86_64-unknown-linux-musl
    default toolchain: stable (default)
            profile: default
modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
```

You can change the defaults if you like. Make sure to add ~/.cargo/bin to your
`PATH` variable.

```
source ~/.cargo/env
```

## 3. Compiling a Rust Program
To get started with Rust, let's try running a simple "Hello, World!" program:
```
$ cargo new hello
$ cd ./hello
$ RUSTFLAGS="-C linker=clang -C link-arg=-fuse-ld=lld" cargo build
```

Notice, we have to pass `RUSTFLAGS` variable so that the `rustc` compiler would
know which C compiler and linker to invoke.

## 4. Static binaries
That's it. Now you have a barely working Rust Lang environment. Have fun fellow
Rustaceans. If you want to build static binaries for some targets with glibc
defaults, you can use the flag `--target x86_64-unknown-linux-musl`.


## Almost System independent install
Given the introduction of rust-lld into the compiler chain. A simpler approach
would be to just install libgcc system wide (this is cargo's runtime dependency)
and then us `rust-lld` as our default linker. Without using any of the
`RUSTFLAGS` options above.

```
apk add libgcc curl
```

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Curl is no longer needed, so you can get rid of it.
```
apk del -r curl
```

Then create a file `~/.cargo/config.toml` and put the following contents in it:

```
[target.x86_64-unknown-linux-musl]
linker = 'rust-lld'
```
This will put in place a global setting to use `rust-lld`.

If you would rather not mess with the global settings, just use this within
your cargo project folder by create a `.config/config.toml` file within it.

This approach is a lot cleaner as it doesn't depend on a ton of system
dependencies. It just uses `libgcc` (which is used by `rustc` during
compilation) and you default libc which in my case is
`musl` which is a runtime dependency as well.
