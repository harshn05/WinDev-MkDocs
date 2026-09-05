# Requirements

## Core environment

WinDev is distributed as a portable core environment plus an optional,
independently mounted `WinDevLibs.iso`.

## WinDevLibs

The library ISO should use the filesystem label:

``` text
WinDevLibs
```

`CmdInit.cmd` discovers the mounted volume by this label and exposes its
root through `WINDEV_LIBS`.

## Read-only libraries

A mounted ISO is read-only. Library builds therefore use a writable
staging/install directory before the finished tree is copied into the
library ISO.
