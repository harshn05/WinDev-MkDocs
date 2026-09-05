# Clang + MSVC

# Clang + MSVC

WinDev also includes a portable LLVM/Clang toolchain configured to use
the portable MSVC installation and Windows SDK.

The LLVM distribution used is:

``` text
clang+llvm-23.1.0-x86_64-pc-windows-msvc
```

It is extracted directly under:

``` text
%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc
```

This is the Windows/MSVC-targeting LLVM build, not a MinGW toolchain.

The resulting architecture is:

``` text
Clang / clang-cl
       │
       ├── MSVC 14.51.36231
       │
       └── Windows SDK 10.0.28000.0
```

This provides a Clang frontend while retaining the MSVC ABI, libraries,
headers, linker environment, and Windows SDK.

## 15.1 Clang-MSVC Environment Loader

The WinDev-specific loader is:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_ClangMSVC.cmd
```

Current contents:

``` bat
@echo off

call "%WINDEV%\msvc\setup_x64.bat"

set "LLVM=%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc"
set "PATH=%LLVM%\bin;%PATH%"

@echo.
@echo ------------------------------------------------------------
@echo   WinDev :: Clang-MSVC
@echo ------------------------------------------------------------
@clang-cl --version
@echo   LLVM Home  : %LLVM%
@echo   MSVC Tools : %VCToolsVersion%
@echo   Windows SDK: %WindowsSDKVersion%
@echo   Host       : %VSCMD_ARG_HOST_ARCH%
@echo   Target     : %VSCMD_ARG_TGT_ARCH%
@echo ------------------------------------------------------------
@echo.
```

The loader first activates the portable MSVC environment and then
prepends the portable LLVM `bin` directory to `PATH`.

This allows `clang-cl` to use the MSVC headers, libraries, linker, and
Windows SDK while keeping LLVM itself portable.

No system-wide LLVM or Visual Studio installation is required by WinDev.

## 15.2 Why Clang-MSVC Is Separate From MinGW

WinDev intentionally keeps Clang-MSVC separate from the existing
MinGW/GCC environment.

The current compiler environments are:

``` text
WinDev::MinGW
    → GCC / G++ / MinGW-w64 / UCRT

WinDev::MSVC
    → Microsoft C/C++ / MSVC / Windows SDK

WinDev::Clang-MSVC
    → Clang / clang-cl + MSVC / Windows SDK
```

A Clang + MinGW combination was experimentally investigated, including
LLVM OpenMP runtime integration, but it is not part of the current
WinDev configuration.

For now, Clang-MSVC is the supported Clang environment.

## 15.3 Clang-MSVC and OpenMP

Clang-MSVC was tested with OpenMP.

A direct Clang/OpenMP build successfully resolved LLVM's OpenMP runtime:

``` text
libomp.dll
→ %WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\bin\libomp.dll
```

CMake also detects OpenMP:

``` text
-- Found OpenMP_CXX: /clang:-fopenmp (found version "5.1")
-- OpenMP found
-- OpenMP flags: /clang:-fopenmp
-- OpenMP libraries: libomp
```

CMake's automatic OpenMP library selection can, however, resolve
`libomp.lib` from the portable MSVC installation instead of LLVM's
`libomp.lib`.

For the current WinDev configuration, the Clang-MSVC CMake shortcut
therefore explicitly selects LLVM's OpenMP import library.

LLVM OpenMP import library:

``` text
%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\lib\libomp.lib
```

LLVM OpenMP runtime:

``` text
%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\bin\libomp.dll
```

This avoids accidentally selecting:

``` text
%WINDEV%\msvc\VC\Tools\MSVC\14.51.36231\lib\x64\libomp.lib
```

## 15.4 Clang-MSVC Build Alias

The Clang-MSVC environment defines the following compiler-specific CMake
shortcut:

``` bat
doskey clangbuild=cmake -GNinja -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl -DOpenMP_libomp_LIBRARY="%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\lib\libomp.lib" -S . -B clangbuild -DCMAKE_BUILD_TYPE=Release $*
```

It is defined in:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_ClangMSVC.cmd
```

The command explicitly selects:

``` text
Generator       → Ninja
C compiler      → clang-cl
C++ compiler    → clang-cl
OpenMP library  → LLVM libomp.lib
Build directory → clangbuild
Build type      → Release
```

Because the path uses `%WINDEV%`, it remains portable and does not
contain a user-specific absolute path.

A typical build is:

``` bat
clangbuild
```

Additional CMake arguments can be appended through `$*`, for example:

``` bat
clangbuild -DCMAKE_CXX_STANDARD=23
```

The generic Ninja shortcut remains in `CmdInit.cmd`:

``` bat
doskey ninjabuild=cmake -GNinja -S . -B ninjabuild -DCMAKE_BUILD_TYPE=Release $*
```

The distinction is intentional:

``` text
ninjabuild
    → generic Ninja build using the currently active compiler environment

clangbuild
    → explicitly selects clang-cl and LLVM OpenMP
```

## 15.5 Testing Clang-MSVC

After loading `WinDev::Clang-MSVC`:

``` bat
where clang-cl
clang-cl --version
```

Expected target:

``` text
Target: x86_64-pc-windows-msvc
```

A basic OpenMP test can be compiled with:

``` bat
clang-cl /clang:-fopenmp test_openmp.cpp
```

The resulting executable can be inspected with:

``` bat
ldd openmp_test.exe
```

For a direct Clang/OpenMP build, the expected LLVM runtime is:

``` text
libomp.dll
    → %WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\bin\libomp.dll
```

For a CMake build:

``` bat
clangbuild
```

and then:

``` bat
cmake --build clangbuild
```

This keeps Clang-MSVC CMake builds isolated in their own build directory
and explicitly selects LLVM's OpenMP import library.

## 15.4 Testing Clang-MSVC

After loading `WinDev::Clang-MSVC`:

``` bat
where clang-cl
clang-cl --version
```

Expected target:

``` text
Target: x86_64-pc-windows-msvc
```

A basic OpenMP test can be compiled with:

``` bat
clang-cl /clang:-fopenmp test_openmp.cpp
```

The resulting executable can be inspected with:

``` bat
ldd openmp_test.exe
```

For a direct Clang/OpenMP build, the expected LLVM runtime is:

``` text
libomp.dll
    → %WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\bin\libomp.dll
```

CMake + Ninja can also be used:

``` bat
cmake -S . -B build -G Ninja
cmake --build build
```
