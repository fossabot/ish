# [iSH](https://ish.app)

[![Build Status](https://travis-ci.org/tbodt/ish.svg?branch=master)](https://travis-ci.org/tbodt/ish)
[![goto counter](https://img.shields.io/github/search/tbodt/ish/goto.svg)](https://github.com/tbodt/ish/search?q=goto)
[![fuck counter](https://img.shields.io/github/search/tbodt/ish/fuck.svg)](https://github.com/tbodt/ish/search?q=fuck)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Filyastray777%2Fish.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Filyastray777%2Fish?ref=badge_shield)

<p align="center">
<a href="https://ish.app">
<img src="https://ish.app/assets/github-readme.png">
</a>
</p>

A project to get a Linux shell running on iOS, using usermode x86 emulation and syscall translation.

For the current status of the project, check the issues tab, and the commit logs.

You can [join the Testflight beta](https://testflight.apple.com/join/97i7KM8O) now. There's also a [Discord server](https://discord.gg/SndDh5y).

# Hacking

You'll need these things to build the project:

 - Python 3
 - Ninja
 - Node and NPM (only when building for iOS)
 - Meson (`pip install meson`)
 - Clang and LLD (on mac, `brew install llvm`, on linux, `sudo apt install clang lld` or `sudo pacman -S clang lld` or whatever)
 - sqlite3 (this is so common it may already be installed on linux and is definitely already installed on mac. if not, do something like `sudo apt install libsqlite3-dev`)

## Build for iOS

Important: Meson 0.50 has a bug that prevents this from working. Install an older working version of meson with `pip install meson==0.49.2`.

Open the project in Xcode and click Run. If you're not me, first open the project build settings and change `ROOT_BUNDLE_IDENTIFIER` to something unique. There are scripts that should do everything else automatically. If you run into any problems, open an issue and I'll try to help.

## Build command line tool for testing

To set up your environment, cd to the project and run `meson build` to create a build directory in `build`. Then cd to the build directory and run `ninja`.

To set up a self-contained Alpine linux filesystem, download the Alpine minirootfs tarball for i386 from the [Alpine website](https://alpinelinux.org/downloads/) and run the `tools/fakefsify.py` script. Specify the minirootfs tarball as the first argument and the name of the output directory as the second argument. Then you can run things inside the Alpine filesystem with `./ish -f alpine /bin/login -f root`, assuming the output directory is called `alpine`.

You can replace `ish` with `tools/ptraceomatic` to run the program in a real process and single step and compare the registers at each step. I use it for debugging. Requires 64-bit Linux 4.11 or later.

## Logging

iSH has several logging channels which can be enabled at build time. By default, all of them are disabled. To enable them:

- In Xcode: Sett the `ISH_LOG` project setting to a space-separated list of log channels.
- With Meson (command line tool for testing): Run `meson configure -Dlog="<space-separated list of log channels>`.

Available channels:

- `strace`: The most useful channel, logs the parameters and return value of almost every system call.
- `instr`: Logs every instruction executed by the emulator. This slows things down a lot.
- `verbose`: Debug logs that don't fit into another category.
- Grep for `DEFAULT_CHANNEL` to see if more log channels have been added since this list was updated.

# A note on the JIT

Possibly the most interesting thing I wrote as part of iSH is the JIT. It's not actually a JIT since it doesn't target machine code. Instead it generates an array of pointers to functions called gadgets, and each gadget ends with a tailcall to the next function; like the threaded code technique used by some Forth interpreters. The result is a speedup of roughly 3-5x compared to pure emulation.

Unfortunately, I made the decision to write nearly all of the gadgets in assembly language. This was probably a good decision with regards to performance (though I'll never know for sure), but a horrible decision with regards to readability, maintainability, and my sanity. The amount of bullshit I've had to put up with from the compiler/assembler/linker is insane. It's like there's a demon in there that makes sure my code is sufficiently deformed, and if not, makes up stupid reasons why it shouldn't compile. In order to stay sane while writing this code, I've had to ignore best practices in code structure and naming. You'll find macros and variables with such descriptive names as `ss` and `s` and `a`. Assembler macros nested beyond belief. And to top it off, there are almost no comments.

So a warning: Long-term exposure to this code may cause loss of sanity, nightmares about GAS macros and linker errors, or any number of other debilitating side effects. This code is known to the State of California to cause cancer, birth defects, and reproductive harm.


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Filyastray777%2Fish.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Filyastray777%2Fish?ref=badge_large)