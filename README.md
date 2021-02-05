<h1>Shoebill <img align="right" src="stork_tiny_head3.jpg" /></h1>

A Macintosh II emulator that runs A/UX (and A/UX only).

Platform | SDL GUI | OSX GUI
---------|:--------|:-------
Linux    | [![Linux Build Status](http://badges.herokuapp.com/travis/emaculation/shoebill?env=BADGE=linux&label=build&branch=master)](https://travis-ci.org/emaculation/shoebill)
OSX      |         | [![OSX Build Status](http://badges.herokuapp.com/travis/emaculation/shoebill?env=BADGE=osx&label=build&branch=master)](https://travis-ci.org/emaculation/shoebill)
Windows  | [![Windows Build status](https://img.shields.io/appveyor/ci/ianfixes/shoebill.svg)](https://ci.appveyor.com/project/ianfixes/shoebill)<br />⚠️[Known issue](https://github.com/emaculation/shoebill/issues/1) 


Shoebill is an all-new, BSD-licensed Macintosh II emulator designed from the ground up with the singular goal of running A/UX.

Shoebill requires a Macintosh II, IIx or IIcx ROM, and a disk image with A/UX installed.

This is a rough attempt to unifty the fixes from the [https://github.com/f4ftx/shoebill](f4tx), [https://github.com/hidikal/shoebill](hidikal), and [https://github.com/JMarlin/shoebill](JMarlin) branches up until now (2020 or so). The long-term effects of doing so has not been studied.

### Supports
* A/UX 1.1.1 through 3.1 (and 3.1.1 a little)

### Currently Implements
* 68020 CPU (mostly)
* 68881 FPU (mostly)
* 68851 PMMU (just enough to boot A/UX)
* SCSI
* ADB
* PRAM
* Ethernet (via emulated Apple EtherTalk/DP8390 card)
* A NuBus video card with 24-bit depth.

### Does not implement (yet)
* Sound
* Floppy
* Serial ports

## How To Build Shoebill

Directories in the git repo:

* `/core` - the platform-independent guts of shoebill
* `/debugger` - a CLI debugger whose GUI is implemented with GLUT (not SDL)
* `/gui` - a Cocoa-based macOS GUI
* `/sdl-gui` - a bad, slapped-together SDL-based GUI for linux/windows/macOS


### Building the core:

The core is plain C with no external dependancies, and it should build cleanly
with clang/gcc without any weird compiler flags.

For the macOS Cocoa GUI, the core is compiled and linked as a static library,
and the GUI is built as a separate Xcode project. For the SDL GUI, there isn't
even a makefile, just a script that compiles and links the whole binary with a
single clang/gcc invocation. (Feel free to improve on this :)

Compiling the core requires a few extra steps, aside from actually compiling and
linking it: several `.c` files need to be preprocessed with a special perl script
(`core/macro.pl`), and the instruction decoder needs to be generated (via
`core/decoder_gen.c`).

Here's how you invoke the preprocessor for a single .c file:

```console
$ perl core/macro.pl <input file> <output file>
```

E.g.

```console
$ perl core/macro.pl core/adb.c tmp/adb.preprocessed.c
```

And here are the `.c` files in `core/` that require preprocessing:

* `adb.c`
* `fpu.c`
* `mc68851.c`
* `mem.c`
* `via.c`
* `floppy.c`
* `core_api.c`
* `cpu.c`
* `dis.c`

The instruction decoder generator is implemented in C, so it itself needs to be
compiled:

```console
$ gcc -O1 core/decoder_gen.c -o tmp/decoder_gen
```

And then it should be called twice to generate two separate files (for the cpu
and disassembler):

```console
$ tmp/decoder_gen inst tmp/
$ tmp/decoder_gen dis tmp/
```
(This will create two files: `tmp/inst_decoder_guts.c` and `tmp/dis_decoder_guts.c`)

Now you have a bunch of .c files which you can compile and link together to
build the core, namely:

1. All the files generated by the preprocessor
2. The two files generated by the instruction decoder
3. These remaining files in `core/`
    * `alloc_pool.c`
    * `atrap_tab.c`
    * `coff.c`
    * `ethernet.c`
    * `exception.c`
    * `filesystem.c`
    * `macii_symbols.c`
    * `redblack.c`
    * `scsi.c`
    * `sound.c`
    * `toby_frame_buffer.c`
    * `video.c`
    * `SoftFloat/softfloat.c`

The makefile in `core/` does all this, but the linking only works on macOS. The
existing makefile tries to create universal (fat 32-/64-bit) static library, and
uses weird macOS-only tools to do so.


### Building the macOS GUI

The GUI is implemented in Cocoa in an Xcode project in `/gui`. You can build
the GUI with:
```console
$ cd gui; xcodebuild -target Shoebill
```

That'll dump the built app in `gui/build/Release/Shoebill.app`


### Building the SDL GUI (for macOS/Linux/Windows)

In `/sdl-gui`, there are
scripts for macOS, Linux, and Windows, which go through all the usual steps to
prepare the core, but then just compile and link the binary together with one
giant gcc call.
