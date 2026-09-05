# Installation

## Place the WinDev directory

Keep the complete WinDev core directory together. The launcher derives
`%WINDEV%` from its own location, so the environment can be moved
without rewriting the launcher.

## Mount WinDevLibs

Mount `WinDevLibs.iso` independently when development libraries are
required.

## Launch

Run:

``` text
%WINDEV%\Launch_WinDev.cmd
```

The launcher starts portable ConEmu with the bundled prompt font
directory.

## Verify the base tools

``` bat
git --version
cmake --version
ninja --version
```

Then launch the required toolchain task.
