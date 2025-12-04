+++
title = 'Introduction to Nix: Getting Started with Nix Package Manager'
date = '2023-07-03T15:30:00-05:00'
description = "A practical introduction to Nix package manager covering basics, nix repl, derivations, channels, and profiles with hands-on examples."
excerpt = "Learn the fundamentals of Nix package manager through practical examples. Explore nix repl, derivations, channels, and profiles."
tags = ['nix', 'nixpkgs', 'package-manager', 'devops', 'tutorial', 'linux', 'macos']
categories = ['Development', 'Tools']
keywords = ['nix', 'nixos', 'nix repl', 'derivation', 'nix channels', 'package management', 'functional package manager']
draft = false
+++

## Introduction to Nix

Nix is great but the learning curve is a bit steep and it takes time to start building production stuff with it. Nevertheless, I'd say the best way to learn is to start your own configuration and build on top of that.

In this article, I will be dissecting the basics of Nix and will be going through number of basic and simple recipes that can be done with `nix`. I am planning to add more in the future.

Let's start by exploring different scenario in `nix repl`:

```nix
❯ nix repl
Welcome to Nix 2.13.3. Type :? for help.

nix-repl>
```

Loading all nix expressions and adding them to current scope:

```nix
nix-repl> :l <nixpkgs>
Added 19653 variables.
```

To list and count the loaded packages:
```nix
nix-repl> builtins.length(builtins.attrNames pkgs)
19654

nix-repl> builtins.attrNames pkgs
[ "AAAAAASomeThingsFailToEvaluate" "AMB-plugins" "ArchiSteamFarm"  .. ]
```

If you're wondering about the `builtins` functions, you can simply ready the documentation about that within the repl:
```nix
nix-repl> :doc builtins.attrNames
Synopsis: builtins.attrNames set

    Return the names of the attributes in the set set in an alphabetically sorted list. For instance, builtins.attrNames { y = 1; x = "foo"; } evaluates to [ "x"
    "y" ].
```

To verify if `pkgs` is a set:
```nix
nix-repl> :doc builtins.isAttrs
Synopsis: builtins.isAttrs e

    Return true if e evaluates to a set, and false otherwise.

nix-repl> builtins.isAttrs pkgs
true
```

**Note:** _If you are like me wondering how `pkgs` was implicitly defined, I believe it's because it's defined in the `nixpkgs` upstream so you can pull the packages from `pkgs` folder/variable._

Another question is why `<nixpkgs>` used instead of `nixpkgs` directly, why `<>` is needed? 🤔

It looks like 
```nix
❯ nix-instantiate --eval --expr '&lt;nixpkgs>'
/Users/s0x/.nix-defexpr/channels/nixpkgs

# OR:
# nix-repl> <nixpkgs>
# /Users/s0x/.nix-defexpr/channels/nixpkgs
```

So based on the above, it tells us that `<nixpkgs>` is an expression used to search for `nixpkgs` in the Nix paths that are defined in the environment variables (`$NIX_PATH or -I`). Another proof:


```nix
❯ nix-instantiate --eval --expr '&lt;mew>'
error: file 'mew' was not found in the Nix search path (add it using $NIX_PATH or -I)

       at «string»:1:1:

            1| <mew>
             | ^
```

Let's try and build python derivation:
```nix
nix-repl> :b python310
This derivation produced the following outputs:
  out -> /nix/store/m4kmky8ay96qwyakxjmj1idcvrdmvb9k-python3-3.10.12
```

We can verify the built derivation by accessing the python shell after *exiting the `nix repl`:
```bash
❯ /nix/store/m4kmky8ay96qwyakxjmj1idcvrdmvb9k-python3-3.10.12/bin/python
Python 3.10.12 (main, Aug 31 2023, 13:04:33) [Clang 11.1.0 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
```nix
nix-repl> :b python311.withPackages(p: [p.requests])

This derivation produced the following outputs:
  out -> /nix/store/bl65mjkcchgamgqw1zw9ns46p4gpi1wh-python3-3.11.4-env

# exit nix repl
❯ /nix/store/bl65mjkcchgamgqw1zw9ns46p4gpi1wh-python3-3.11.4-env/bin/python
Python 3.11.4 (main, Aug 31 2023, 13:04:34) [Clang 11.1.0 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>>
```

Nix Closure is the Nix's package dependency tree. It includes all the packages required to build and run the package.

Channels
A channel is a git repository containing a list of packages. 
```nix
❯ nix-channel --list
darwin https://github.com/LnL7/nix-darwin/archive/master.tar.gz
nixpkgs https://github.com/NixOS/nixpkgs/archive/staging-next.tar.gz
```

Profiles
Profiles are groups of generations so that different users don’t interfere with each other if they don’t want to. Profiles and User Environments are Nix’s mechanisms to allow different users to use different configurations.

Derivation
A Derivation is a Nix expression describing everything that goes into a package build action (build tools, dependencies, sources, build scripts, environment variables). It is anything needed to build up a package