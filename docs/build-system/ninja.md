# Ninja

# Ninja

WinDev includes Ninja as a standalone executable.

The package used is:

``` text
ninja-win.zip
```

The executable:

``` text
ninja.exe
```

is stored directly in:

``` text
%WINDEV%\Programs
```

Because `%WINDEV%\Programs` is already added to `PATH`, no additional
Ninja-specific configuration is required.

Test:

``` bat
ninja --version
```

CMake and Ninja can be used together with the MinGW toolchain:

``` bat
cmake -S . -B build -G Ninja
cmake --build build
```

------------------------------------------------------------------------
