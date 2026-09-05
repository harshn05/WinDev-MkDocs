# ISO Architecture

# ISO Distribution

WinDev 2.0 separates the distribution into a portable core environment
and a separate library payload. The intended distribution model is:

``` text
WinDevCore.iso
    │
    ├── ConEmu / Clink / Oh My Posh
    ├── GNU utilities
    ├── Programs
    ├── WinPython
    ├── MinGW
    ├── MSVC / Windows SDK
    └── toolchain infrastructure

WinDevLibs.iso
    │
    └── prebuilt third-party development libraries
```

Both ISOs are portable. `WinDevCore.iso` contains the development
environment itself, while `WinDevLibs.iso` can be mounted independently
when the corresponding libraries are required.

A mounted ISO is read-only. Therefore the build/install process for a
library must use a writable staging directory before the resulting
library tree is copied into `WinDevLibs.iso`.

For example, QtBase 6.8.4 was installed into a writable directory first:

``` bat
cmake --install . --prefix "C:/Users/harshn/Desktop/WinDevLibs/build/qtbase/6.8.4"
```

The resulting tree was then copied into:

``` text
WinDevLibs.iso
└── mingw
    └── qtbase
        └── 6.8.4
```

The library ISO is discovered automatically through its filesystem
label, so its drive letter does not need to be documented or configured
manually. The active toolchain then selects the corresponding `mingw` or
`msvc` library namespace automatically.

------------------------------------------------------------------------
