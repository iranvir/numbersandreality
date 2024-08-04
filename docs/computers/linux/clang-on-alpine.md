# Installing LLVM/Clang on Alpine Linux
Setting up Clang was probably the most involved process in this series.
Ironically, creating a C programming environment without any GNU utilities is a
lot harder than setting up a plain [Go Lang
environment](series/alpine-series/go-lang-on-alpine/) setup or even Rust.

But that's my goal here. To create an environment for C programming that doesn't
involve me having to rely on GNU utilities. I am using Alpine Linux which
immediately eliminates the glibc dependency, the rest of the article will cover
how to minimize other GNU dependencies.

## 1. The compiler
We start with installing clang from apk repository, it will depend on libgcc and
libstdc++ both of which are a part of GNU project, from what I can tell.
Although there is no way to install Clang via apk repos without also installing
libgcc and libstdc++, I can later show you how to avoid using libgcc. (Libstc++
can be avoided by simply not using C++)
```
$ apk add clang
(1/8) Installing libffi (3.3-r2)
(2/8) Installing libgcc (9.3.0-r2)
(3/8) Installing libstdc++ (9.3.0-r2)
(4/8) Installing xz-libs (5.2.5-r0)
(5/8) Installing libxml2 (2.9.10-r5)
(6/8) Installing llvm10-libs (10.0.0-r2)
(7/8) Installing clang-libs (10.0.0-r2)
(8/8) Installing clang (10.0.0-r2)
Executing busybox-1.31.1-r19.trigger
OK: 156 MiB in 22 packages
```

## 2. The linker
We will then install the LLVM linker, called lld.
```
$ apk add lld
(1/7) Installing libffi (3.3-r2)
(2/7) Installing libgcc (9.3.0-r2)
(3/7) Installing libstdc++ (9.3.0-r2)
(4/7) Installing xz-libs (5.2.5-r0)
(5/7) Installing libxml2 (2.9.10-r5)
(6/7) Installing llvm10-libs (10.0.0-r2)
(7/7) Installing lld (10.0.0-r0)
Executing busybox-1.31.1-r19.trigger
OK: 72 MiB in 21 packages
```
Again, you can see a lot of familiar packages include the libgcc and libstdc++.

## 3. The standard library
The standard library is not going to be from the LLVM project but rather from
musl libc project. Alpine already uses musl as an alternative to glibc and the
header files that we are going to fetch now will be from [musl
project](https://musl.libc.org/) as well.
```
$ apk add musl-dev
```

## 4. The Compiler Runtime
The compiler-rt from LLVM is the next piece of the puzzle and let's install
that.
```
$ apk add compiler-rt compiler-rt-static
```

## 5. Compiling a C program
Now you are ready to compile you C programming, with as little GNU dependencies
as possible. Just remember to use the flags as shown below:
```
$ clang -fuse-ld=lld --rtlib=compiler-rt hello.c
```
The `-fuse-ld=lld` flag sets the linker to be LLVM's lld linker rather than
searching for a GNU Gold ld, and similarly, `--rtlib=compiler-rt` makes use of
the compiler-rt runtimes rather than searching for and failing to fetch their
gcc counterpart.

Use `-v` flag to print verbose output and verify that indeed the correct
toolchain is being used. Since you are working with Alpine, you might be
interested in developing static binaries, use `-static` flag to make sure that
the binaries are static and would run on any Linux system, regardless of its
usage of glibc or musl...As long as the kernel supports it ;)

Here's a reference video if you prefer video format over the written word:
{{< youtube gQHGxCB_lLQ >}}
