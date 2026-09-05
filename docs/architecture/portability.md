# Portability Goals

# Portability Goals

The WinDev environment is designed around the following principles:

-   Keep the core development tools inside the WinDev core payload.
-   Keep third-party development libraries in a separate
    `WinDevLibs.iso`.
-   Avoid unnecessary system-wide installations.
-   Keep Python portable.
-   Keep MinGW portable.
-   Keep MSVC and the Windows SDK portable.
-   Keep Git portable.
-   Keep CMake portable.
-   Keep Ninja portable.
-   Keep GNU utilities portable.
-   Keep Oh My Posh portable.
-   Keep terminal fonts portable.
-   Use relative or dynamically resolved paths wherever possible.
-   Discover the `WinDevLibs` mount point by volume label rather than
    drive letter.
-   Use ConEmu Tasks for predefined development environments.
-   Keep common environment initialization separate from individual
    toolchains.
-   Automatically expose compatible libraries when their library ISO is
    available.
-   Make the environment movable between directories and drives.
-   Allow the environment to be distributed as portable ISO payloads.

The intended workflow is:

``` text
WinDevCore
    │
    ▼
Launch_WinDev.cmd
    │
    ▼
ConEmu
    │
    ▼
CmdInit.cmd
    │
    ├── discover WinDevLibs
    │       │
    │       └── WINDEV_LIBS=F:\  (example only)
    │
    ▼
Toolchain Task
    │
    ├── MinGW
    │     └── if WinDevLibs is available
    │             └── activate MinGW libraries
    │
    └── MSVC
          └── if WinDevLibs is available
                  └── activate MSVC libraries
```

Additional environments can then be launched through ConEmu Tasks:

``` text
Ctrl+Shift+P  → Python
Ctrl+Shift+T  → MinGW
Ctrl+Shift+V  → MSVC
Ctrl+Shift+C  → ClangMSVC
```

The combined environments can be selected from the ConEmu Task list:

``` text
WinDev::Python+MinGW
WinDev::Python+MSVC
WinDev::Python+ClangMSVC
```

------------------------------------------------------------------------
