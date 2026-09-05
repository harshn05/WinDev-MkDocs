# Environment Variables

## `%WINDEV%`

The WinDev root is derived from the initialization script:

``` bat
for %%I in ("%~dp0..\..") do set "WINDEV=%%~fI"
```

## `%ConEmuBaseDir%`

ConEmu exposes its installation base directory through this variable.

## `WINDEV_LIBS`

The mounted library ISO is discovered by its filesystem label and
exposed through:

``` text
WINDEV_LIBS=F:\
```

The drive letter above is only an example.

## Library roots

The library loaders derive:

``` text
WINDEV_MINGW_LIBS=%WINDEV_LIBS%\mingw
WINDEV_MSVC_LIBS=%WINDEV_LIBS%\msvc
```

and then set library-specific roots and `CMAKE_PREFIX_PATH` entries.
