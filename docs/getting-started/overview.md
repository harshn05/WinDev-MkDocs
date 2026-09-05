# Overview

# Directory Structure

The WinDev root is represented by:

``` text
%WINDEV%\
```

The current structure is approximately:

``` text
WinDevCore
│
├── Launch_WinDev.cmd
│
├── Programs
│   ├── CoreUtils
│   ├── PortableGit
│   ├── cmake-4.4.3-windows-x86_64
│   └── _loadPrograms.cmd
│
├── WPy64-312101
│
├── winlibs-x86_64-posix-seh-gcc-15.3.0-mingw-w64ucrt-14.0.0-r1
│
└── ConEmuPack.230724
    └── ConEmu
        ├── clink
        │   └── oh-my-posh.lua
        │
        ├── _GNU_Utils
        │   ├── _GOW
        │   └── _MSYS2
        │
        ├── _Tools
        │   └── cpldd.exe
        │
        ├── _OhMyPosh
        │   ├── _themes
        │   │   └── peru.omp.json
        │   └── _Fonts
        │
        ├── _Toolchains
        │   ├── _icons
        │   │   ├── python.ico
        │   │   ├── mingw.ico
        │   │   ├── msvc.ico
        │   │   └── python_mingw_128.ico
        │   │
        │   ├── loadenv_WinPython.cmd
        │   ├── loadenv_MinGW.cmd
        │   ├── loadenv_MinGW_Libs.cmd
        │   ├── loadenv_MSVC.cmd
        │   └── loadenv_MSVC_Libs.cmd
        │
        └── CmdInit.cmd

WinDevLibs.iso
└── mounted independently
    ├── mingw
    │   └── qtbase
    │       └── 6.8.4
    └── msvc
        └── qtbase
            └── 6.8.4
```

`WinDevCore` contains the shell, utilities, languages, compilers, and
toolchain infrastructure. `WinDevLibs.iso` is a separate library payload
containing prebuilt development SDKs/libraries. The two are
intentionally decoupled.

The mounted library ISO does not have a fixed drive letter.
`CmdInit.cmd` discovers the volume by its filesystem label
(`WinDevLibs`) and exports `WINDEV_LIBS` accordingly.

For example, the ISO may currently be mounted as `F:`:

``` text
WINDEV_LIBS=F:\
```

# `%WINDEV%`

The entire development environment is rooted at `%WINDEV%`.

For example:

``` bat
set WINDEV=C:\Users\harshn\Desktop\WinDev
```

The actual path is not hard-coded into the internal scripts wherever
possible.

Scripts derive the root from their own location.

This allows the complete environment to be moved to another directory or
drive.

------------------------------------------------------------------------
