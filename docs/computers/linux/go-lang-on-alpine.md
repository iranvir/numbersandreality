# Installing Go Lang on Alpine Linux
In my [previous post](/series/alpine-series/alpine-wsl/) I showed how you can build your own WSL
distro from Alpine root image. If you followed along, and then proceded to use
the resulting distro, you would have been very frusted....I apoligize.

The first attempt at installing any real language like Go or Rust or NodeJS
would have resulted in another explosion of packages.

I chose Alpine thinking that since this is used so heavily to package apps into
docker images, it really must have been silm and dependency free. I tried
installing Go Language from apk package manager and it started pulling libgcc,
gcc and a bunch of other libraries.

Seeing this heavy handed approach, I decided to start over. I downloaded the
prebuilt Go binaries straight from the source, but I ran into a few caveats so I
decided to document the entire process here:

1.  Download the [compressed archive](https://golang.org/dl/):
    ```
    $ cd /usr/bin $ wget https://golang.org/GetYourProperLinuxURL
    $ tar -xf golang-linux.tar.gz ## This is an example file name.
    $ rm golang-linux.tar.gz
    ```

2.  Add the Go Lang binaries path to your PATH environment variable:
    ```
    $ PATH="$PATH:/usr/bin/go/bin"
    ```
    You can also directly edit your /etc/profile file and add `/usr/bin/go/bin`
    at the end of your PATH variable's list of values.

3.  Try running Go
    ``` 
    $ go version -ash: go: not found
    ```
    However, gofmt would be working fine!
    ```
    $ gofmt
    ```

4. Try not to lose your mind after this incredulity. Let's debug this by
   inspecting the `go` binary in `/usr/bin/go/bin`.

5.  Install `file` and `patchelf` packages.
    ```
    $ apk add file patchelf
    ```
    Don't fret about these, we will get rid of them very soon.

6. Inspect the Package:
    ```
    /usr/bin/go/bin $ file go hugo
    .... dynamically linked, **interpreter /lib64/ld-linux-x86-64.so.2 ...
    ```
    So the interpreter is set to ld-linux-x86... we will simply swap it with
    Alpine's default /lib/libc.musl-x86_64.so.1

    To know your default interpreter in Alpine simply go to `/lib` and look for
    a name like ld-musl-...-so.1 and alongside will be a file libc.musl... which
    would be a symlink to the original ld-musl... interpreter. We will use this
    symlink, since package managers may update the original interpreter and
    redirect this symlink to the updated file.

7.  To update the interpreter we will use `patchelf`:
    ```
    $ patchelf --set-interpreter /lib/libc.musl-x86_64.so.1 go
    ```

8.  Test if everything is working.
    ```
    $ go version go version go1.15.3 linux/amd64
    ```
    I also tested this by [compiling hugo
    project](https://github.com/gohugoio/hugo#build-and-install-the-binaries-from-source-advanced-install)
    with CGO enabled it worked without any hiccups. With the only side-effect
    that the resulting binary was also linked to the original ld-linux file and
    needed to patch the resulting hugo binary as well.

9.  You no longer need `file` and `patchelf`, if you are building statically
    linked binaries. Feel free to remove them.
    ```
    $ apk del -r patchelf file
    ```

## Fin.

Hope you found this helpful. If I have done something incredibly stupid and
there's a simple configuration option to change the interpreter from ld-linux to
libc.musl please let me know. :)
