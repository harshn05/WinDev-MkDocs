# CMake

# CMake

WinDev includes the portable CMake ZIP distribution.

The package used is:

``` text
cmake-4.4.3-windows-x86_64.zip
```

The extracted installation is:

``` text
%WINDEV%\Programs\cmake-4.4.3-windows-x86_64
```

CMake's executable directory:

``` text
%WINDEV%\Programs\cmake-4.4.3-windows-x86_64\bin
```

is added to `PATH` through `_loadPrograms.cmd`.

Test:

``` bat
cmake --version
```

CMake is kept separate from compiler toolchains. It provides the
build-system generation layer and can be combined with MinGW or other
compilers.

------------------------------------------------------------------------
