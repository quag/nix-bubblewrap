# nix-bubblewrap

**[Nix](https://nixos.org) - [bubblewrap](https://github.com/containers/bubblewrap) integration**

## Why

In a typical Linux system, bubblewrap is run like this:

    bwrap --ro-bind /usr /usr --proc /proc --dev /dev --unshare-pid bash

With Nix, one would have to replace `/usr` with `/nix/store`...
but
[all kind](https://search.nixos.org/options?channel=unstable&sort=relevance&type=packages&query=password)
[of stuff](https://search.nixos.org/options?channel=unstable&sort=relevance&type=packages&query=secret)
you may not want an attacker to see can end up in the store.
Binding individual store paths can also be a pain since the whole closure is
needed.
This script automates that process.
Additional flags to add permissions in a nixos-specific way
(eg. keeping `/run/opengl-driver` and `/etc/ssl` into account)
are provided.

## Usage

    ./nix-bwrap.tcl [OPTIONS] COMMAND ...

Examples:

    $ nix-shell -p hello --run "./nix-bwrap.tcl hello"
    Hello, world!
    $ nix-shell -p tree --run "./nix-bwrap.tcl tree -L 3 /"
    /
    `-- nix
        `-- store
            |-- 0ldsqvqp3y1bn6852ymksfa2kfkr3dkb-tree-1.8.0
            |-- 563528481rvhc5kxwipjmg6rqrl95mdx-glibc-2.33-56
            |-- qbdsd82q5fyr0v31cvfxda0n0h7jh03g-libunistring-0.9.10
            `-- scz4zbxirykss3hh5iahgl39wk9wpaps-libidn2-2.3.2

    6 directories, 0 files

## Troubeshooting

You may want to add a `realpath` call to remove layers of indirection that
won't be found in the sandbox, such as `/run/current-system/sw/bin/` for
packages installed through `/etc/nixos/configuration.nix`.

    $ ./nix-bwrap.tcl -x11 -gpu -net firefox
    bwrap: execvp /run/current-system/sw/bin/firefox: No such file or directory
    $ ./nix-bwrap.tcl -x11 -gpu -net $(realpath $(which firefox)) https://example.org
    [firefox starts...]

This is not done automatically because it breaks executables that rely on
`argv[0]`, such as coreutils and busybox.

## Contributing

TODO