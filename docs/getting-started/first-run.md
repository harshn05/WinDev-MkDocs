# First Run

## Default startup

The default ConEmu startup task is:

``` text
WinDev::MinGW
```

Useful shortcuts:

``` text
Ctrl+Shift+P  → WinDev::Python
Ctrl+Shift+T  → WinDev::MinGW
Ctrl+Shift+V  → WinDev::MSVC
Ctrl+Shift+C  → WinDev::ClangMSVC
```

Combined Python environments are available from the ConEmu Task list.

## Library detection

`CmdInit.cmd` reports the detected library ISO, for example:

``` text
[WinDev] Libraries: F:\
```

or:

``` text
[WinDev] Libraries: NOT FOUND
```

The drive letter is discovered dynamically and is not hard-coded.
