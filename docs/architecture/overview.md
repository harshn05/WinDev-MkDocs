# Architecture Overview

# Toolchain Architecture

WinDev separates common environment initialization from
toolchain-specific initialization.

For MinGW and MSVC, the toolchain loaders conditionally load their
matching library environments when `WINDEV_LIBS` is available.

Common environment:

``` text
CmdInit.cmd
```

Toolchain-specific environments:

``` text
_Toolchains
├── loadenv_WinPython.cmd
├── loadenv_MinGW.cmd
├── loadenv_MinGW_Libs.cmd
├── loadenv_MSVC.cmd
├── loadenv_MSVC_Libs.cmd
└── loadenv_ClangMSVC.cmd
```

Toolchain-specific CMake shortcuts are defined by the corresponding
loader:

``` text
CmdInit.cmd
└── ninjabuild

loadenv_MinGW.cmd
└── gccbuild

loadenv_MSVC.cmd
├── msvcbuild
└── jombuild

loadenv_ClangMSVC.cmd
└── clangbuild
```

General programs:

``` text
Programs
├── PortableGit
├── CMake
├── Ninja
└── CoreUtils
```

The resulting architecture is:

``` text
                    ConEmu
                       │
                       ▼
                  CmdInit.cmd
                       │
        ┌──────────────┼──────────────┐
        │              │              │
       GOW           MSYS2         Programs
                                      │
                          ┌───────────┼───────────┐
                          │           │           │
                         Git        CMake       Ninja
                                     
                       │
                Toolchain Task
                       │
              ┌────────┴────────┐
              │                 │
           Python             MinGW
              │                 │
        WinPython 3.12      GCC / G++
```

This keeps the actual environment configuration in reusable scripts
instead of embedding it directly into ConEmu Tasks.

# Design Philosophy

WinDev is intended to function as a **portable development
workstation**, rather than simply a collection of portable applications.

The environment is organized into a core runtime/toolchain layer and an
independently mounted library layer:

``` text
┌──────────────────────────────────────────┐
│                 WinDevCore              │
├──────────────────────────────────────────┤
│                 ConEmu                  │
│        Terminal / User Interface        │
├──────────────────────────────────────────┤
│            Clink + Oh My Posh           │
│          Shell / Prompt / Fonts         │
├──────────────────────────────────────────┤
│          GNU Utilities / Programs       │
│   GOW / MSYS2 / CoreUtils / Git /       │
│            CMake / Ninja                │
├──────────────────────────────────────────┤
│               Toolchains                │
│      Python / MinGW / MSVC / Clang      │
└──────────────────────────────────────────┘
                     │
                     │ WINDEV_LIBS
                     ▼
┌──────────────────────────────────────────┐
│              WinDevLibs.iso             │
├──────────────────────────────────────────┤
│       Prebuilt development libraries    │
│   Qt / VTK / ITK / OpenCV / future...   │
└──────────────────────────────────────────┘
```

ConEmu provides the terminal.

Clink provides shell integration.

Oh My Posh provides the prompt.

Portable fonts provide the required prompt glyphs.

GNU utilities provide the Unix-style command-line experience.

Git, CMake, Ninja, and other general-purpose programs provide the
development infrastructure.

Toolchain scripts provide isolated development environments.

`CmdInit.cmd` discovers the optional `WinDevLibs` payload without
depending on a fixed drive letter. Toolchain loaders can then activate
libraries that match the active compiler/runtime environment.

For the MinGW and MSVC environments, the toolchain loaders automatically
invoke their corresponding library loaders when the matching library
tree is available. This allows the normal `WinDev::MinGW` and
`WinDev::MSVC` tasks to provide the compiler together with the currently
integrated libraries, without requiring repeated manual `PATH` or
`CMAKE_PREFIX_PATH` changes.

ConEmu Tasks provide a convenient user interface for switching between
toolchain environments.

The end result is a reproducible, movable Windows development
environment with a portable core and independently maintained, versioned
library payloads.

WinDev is intended to function as a **portable development
workstation**, rather than simply a collection of portable applications.

The environment is organized into four major layers:

``` text
┌──────────────────────────────────────┐
│              ConEmu                 │
│       Terminal / User Interface     │
├──────────────────────────────────────┤
│          Clink + Oh My Posh          │
│       Shell / Prompt / Fonts        │
├──────────────────────────────────────┤
│       GNU Utilities / Programs      │
│  GOW / MSYS2 / CoreUtils / Git /   │
│       CMake / Ninja                 │
├──────────────────────────────────────┤
│             Toolchains              │
│       Python / MinGW / Future       │
│              MSVC                   │
└──────────────────────────────────────┘
```

ConEmu provides the terminal.

Clink provides shell integration.

Oh My Posh provides the prompt.

Portable fonts provide the required prompt glyphs.

GNU utilities provide the Unix-style command-line experience.

Git, CMake, Ninja, and other general-purpose programs provide the
development infrastructure.

Toolchain scripts provide isolated development environments.

ConEmu Tasks provide a convenient user interface for switching between
those environments.

The end result is a reproducible, movable Windows development
environment that can be carried as a single WinDev directory.
