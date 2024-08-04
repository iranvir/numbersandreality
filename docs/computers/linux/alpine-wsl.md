# Automated Alpine Linux Setup in WSL2
Recently, I got quite frustrated with the bloated WSL Distros that are shipped by Canonical and Debian Team. 
It comes preinstalled with Systemd, a horribly old version of Python and for some reason insists that vim depends on 
sound drivers and sound themes from freedesktop.org

Having no intention of trusting Microsoft store again, I create my own WSL distro in 3 simple setups:

1.  Get the latest minirootfs for your architecture from 
    [Alpine Downloads Page](https://alpinelinux.org/downloads/).
    Verify the sha256sum and gpg signatures.
2. Run the WSL import command:
    ```
    $ wsl --import MyCustomDistro C:\Directory\For\Custom\WSL .\alpine.tar.gz
    $ wsl -d MyCustomDistro
    ```
3.  Create symlinks to useful Windows binaries like `C:\Windows\explorer.exe` or
    VS Code to somewhere in your WSL $PATH variable, if you want to invoke them
    from within the WSL environment. You can also customize further, as I did on
    [my previous post](blog/wsl-setup/). Or copy the PATH environment from an
    existing "official" distro and add it to your `/etc/profile`.

    ```
    $ ln -s /usr/bin/code /mnt/c/Users/r/AppData/Local/Programs/Microsoft VS Code/bin/code
    $ ln -s /usr/bin/explorer.exe /mnt/c/Windows/explorer.exe
    ```
Here's a reference video if you prefer videos over articles:
{{< youtube U4I1VDsKD9g >}}

The rootfs is less than 3MiB and the installation takes only 77MiB of disk space.
Meaning that there's not much unwanted/outdated library, driver, userland
utilities to deal with.
