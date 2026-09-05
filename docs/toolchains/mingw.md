# MinGW-w64

# MinGW-w64

WinDev uses a portable WinLibs MinGW-w64 toolchain.

Current configuration:

``` text
GCC        15.3.0
MinGW-w64  14.0.0
Runtime    UCRT
Threading  POSIX
Exception  SEH
Architecture x86_64
```

The distribution used is:

``` text
winlibs-x86_64-posix-seh-gcc-15.3.0-mingw-w64ucrt-14.0.0-r1
```

It is extracted directly under `%WINDEV%`.

Therefore:

``` text
%WINDEV%\winlibs-x86_64-posix-seh-gcc-15.3.0-mingw-w64ucrt-14.0.0-r1
```

The WinLibs distribution provides `mingwvars.bat`, which is used to
initialize the environment.

## 13.1 MinGW Toolchain Loader

The loader is:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_MinGW.cmd
```

The current working loader is:

``` bat
@echo off

call "%WINDEV%\winlibs-x86_64-posix-seh-gcc-15.3.0-mingw-w64ucrt-14.0.0-r1\mingw64\mingwvars.bat"

set "CC=gcc"
set "CXX=g++"

echo.
echo === MinGW Toolchain ===
echo CC=%CC%
echo CXX=%CXX%
echo.

%CC% --version
%CXX% --version

doskey gccbuild=cmake -G "MinGW Makefiles" -S . -B gccbuild -DCMAKE_BUILD_TYPE=Release $*

rem ============================================================
rem Load MinGW libraries
rem ============================================================

if defined WINDEV_LIBS (
    echo [WinDev] WinDevLibs detected: %WINDEV_LIBS%

    if exist "%WINDEV_LIBS%\mingw" (
        call "%ConEmuBaseDir%\_Toolchains\loadenv_MinGW_Libs.cmd"
    ) else (
        echo [WinDev] MinGW libraries not found in WinDevLibs.
    )
) else (
    echo [WinDev] WinDevLibs not mounted.
    echo [WinDev] Continuing with MinGW only.
)

echo.
echo [WinDev] MinGW environment active.
echo [WinDev] CC=%CC%
echo [WinDev] CXX=%CXX%

if defined WINDEV_LIBS (
    echo [WinDev] Libraries=%WINDEV_LIBS%\mingw
)

echo.
```

The important design point is that `loadenv_MinGW.cmd` remains the
user-facing MinGW entry point. If `WinDevLibs` is mounted, the MinGW
library environment is loaded automatically; otherwise MinGW works
normally as a standalone toolchain.

## 13.2 MinGW Library Environment

The library loader is:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_MinGW_Libs.cmd
```

Its purpose is to activate the libraries that belong to the MinGW
ecosystem without hard-coding the `WinDevLibs` drive letter.

Current working script:

``` bat
@echo off

rem ============================================================
rem WinDev - MinGW Libraries Environment
rem ============================================================

echo.
echo ------------------------------------------------------------
echo  WinDev :: MinGW Libraries
echo ------------------------------------------------------------

rem ============================================================
rem Validate WinDevLibs
rem ============================================================

if not defined WINDEV_LIBS (
    echo [WinDev] WINDEV_LIBS is not defined.
    echo [WinDev] No MinGW libraries loaded.
    exit /b 0
)

if not exist "%WINDEV_LIBS%\mingw" (
    echo [WinDev] MinGW library directory not found:
    echo          %WINDEV_LIBS%\mingw
    exit /b 0
)

set "WINDEV_MINGW_LIBS=%WINDEV_LIBS%\mingw"

echo [WinDev] MinGW libraries: %WINDEV_MINGW_LIBS%

rem ============================================================
rem Qt
rem ============================================================

set "QT_ROOT=%WINDEV_MINGW_LIBS%\qtbase\6.8.4"

if exist "%QT_ROOT%" (
    set "PATH=%QT_ROOT%\bin;%PATH%"
    set "CMAKE_PREFIX_PATH=%QT_ROOT%;%CMAKE_PREFIX_PATH%"

    echo [WinDev] Qt 6.8.4       : enabled
) else (
    echo [WinDev] Qt 6.8.4       : not found
)

rem ============================================================
rem VTK
rem ============================================================

set "VTK_ROOT=%WINDEV_MINGW_LIBS%\vtk\9.4.2"

if exist "%VTK_ROOT%" (
    set "PATH=%VTK_ROOT%\bin;%PATH%"
    set "CMAKE_PREFIX_PATH=%VTK_ROOT%;%CMAKE_PREFIX_PATH%"

    echo [WinDev] VTK 9.4.2       : enabled
) else (
    echo [WinDev] VTK 9.4.2       : not found
)

rem ============================================================
rem Future libraries
rem ============================================================
rem
rem ITK
rem ----
rem set "ITK_ROOT=%WINDEV_MINGW_LIBS%\itk\..."
rem
rem OpenCV
rem ------
rem set "OpenCV_ROOT=%WINDEV_MINGW_LIBS%\opencv\..."

rem ============================================================
rem Done
rem ============================================================

echo.
echo [WinDev] MinGW library environment initialized.
echo.
```

At the current stage, Qt 6.8.4 is integrated for both the MinGW and MSVC
library environments. VTK, ITK, OpenCV, and other libraries can be added
here as their WinDevLibs layouts become finalized.

This deliberately avoids requiring manual commands such as:

``` bat
set PATH=%PATH%;F:\mingw\qtbase\6.8.4\bin
```

The drive letter is discovered automatically and `CMAKE_PREFIX_PATH` is
configured along with `PATH`.

## 13.3 WinDevLibs ISO

The third-party development libraries are maintained separately from the
WinDevCore environment in `WinDevLibs.iso`. This separation provides a
clean distinction between:

``` text
WinDevCore.iso
    shell + utilities + Python + compilers + build tools

WinDevLibs.iso
    prebuilt development libraries / SDKs
```

The library ISO can therefore be mounted independently and replaced or
updated without rebuilding the core environment.

The current library layout is:

``` text
WinDevLibs.iso
├── mingw
│   ├── qtbase
│   │   └── 6.8.4
│   └── vtk
│       └── 9.4.2
├── msvc
│   └── qtbase
│       └── 6.8.4
└── sample_test_codes
    └── ...
```

Each toolchain has its own library namespace. This keeps
compiler/runtime specific library builds separate while allowing them to
share the same versioned library ISO.

The actual mounted drive letter is intentionally irrelevant to the
WinDev configuration. `CmdInit.cmd` discovers it through the
`WinDevLibs` volume label and stores the root in `WINDEV_LIBS`.

## 13.4 Building QtBase 6.8.4 for WinDevLibs

QtBase 6.8.4 is built as a shared Qt distribution for the portable MinGW
environment. The build is kept separate from the final read-only ISO
payload.

The current configuration uses:

``` text
QtBase 6.8.4
MinGW-w64 / GCC 15.3.0
Shared libraries
UCRT
POSIX threading
SEH
```

The build was configured with the WinDev Ninja shortcut and the
following feature options:

``` bat
ninjabuild -DBUILD_SHARED_LIBS=ON -DFEATURE_freetype=OFF -DFEATURE_opengl_desktop=ON
```

The build/install workflow is intentionally split into two locations.
The Qt build tree and install prefix must be writable, while the mounted
ISO is read-only. Therefore `ninja install` should not be directed
directly into the mounted `WinDevLibs.iso`.

A writable staging/install location was used instead:

``` bat
cmake --install . --prefix "C:/Users/harshn/Desktop/WinDevLibs/build/qtbase/6.8.4"
```

After installation, the resulting Qt tree can be copied into the
prepared `WinDevLibs.iso` layout under the matching toolchain namespace:

``` text
WinDevLibs.iso
├── mingw
│   └── qtbase
│       └── 6.8.4
└── msvc
    └── qtbase
        └── 6.8.4
```

The installed Qt distribution includes `windeployqt.exe`, so a separate
QtTools installation is not required merely to obtain `windeployqt` for
this QtBase build.

## 13.5 VTK 9.4.2 for MinGW

VTK 9.4.2 has been successfully built with the WinDev MinGW environment
using GCC 15.3.0 / MinGW-w64 14.0.0 / UCRT. The installation is stored
in the MinGW namespace of `WinDevLibs.iso`:

``` text
WinDevLibs.iso
└── mingw
    └── vtk
        └── 9.4.2
```

The installed VTK tree is activated automatically by
`loadenv_MinGW_Libs.cmd`. When present, its `bin` directory is added to
`PATH` and its installation prefix is added to `CMAKE_PREFIX_PATH`.

### 13.5.1 VTK Build Configuration

The successful build used:

``` bat
gccbuild -DBUILD_SHARED_LIBS=True ^
-DVTK_GROUP_ENABLE_Qt=YES ^
-DCMAKE_CXX_FLAGS=-fcommon ^
-DCMAKE_C_FLAGS="-fcommon -D_GNU_SOURCE" ^
-DVTK_ENABLE_WRAPPING=False ^
-DVTK_MODULE_ENABLE_VTK_GUISupportQt=YES ^
-DVTK_MODULE_ENABLE_VTK_GUISupportQtQuick=NO
```

The resulting installation was staged to a writable directory before
being copied into the read-only ISO:

``` bat
cmake --install . --prefix "C:/Users/harshn/Desktop/WinDevLibs/mingw/vtk/9.4.2"
```

### 13.5.2 VTK 9.4.2 MinGW Compatibility Patches

The bundled third-party sources required a small number of source-level
compatibility changes for the WinDev MinGW environment.

**libproj**

File:

``` text
ThirdParty/libproj/vtklibproj/src/filemanager.cpp
```

Added:

``` cpp
#include <cstdint>
```

This provides the declaration of `uint32_t` used by the source.

**NetCDF**

File:

``` text
ThirdParty/netcdf/vtknetcdf/include/ncpathmgr.h
```

Added the platform-aware `STAT` definition:

``` c
#ifndef STAT
#ifdef _WIN32
#define STAT struct _stat64 *
#else
#define STAT struct _stat *
#endif
#endif
```

The `NCstat()` declaration was changed from:

``` c
EXTERNL int NCstat(const char* path, struct stat* buf);
```

to:

``` c
EXTERNL int NCstat(const char* path, STAT buf);
```

File:

``` text
ThirdParty/netcdf/vtknetcdf/libdispatch/dpathmgr.c
```

The Windows implementation was changed from:

``` c
NCstat(const char* path, struct stat* buf)
```

to:

``` c
NCstat(const char* path, struct _stat64* buf)
```

This makes the implementation compatible with MinGW's `_wstat64()`
interface.

**C compiler flags**

The successful configuration also uses:

``` text
-fcommon
-D_GNU_SOURCE
```

with `-fcommon` applied to both C and C++ compilation and `_GNU_SOURCE`
applied to C compilation. The latter provides the GNU declaration needed
by the bundled HDF5 code for `vasprintf()`.

## 13.6 Sample Test Codes

`WinDevLibs.iso` also contains a `sample_test_codes` directory at its
root. These are small known-good projects used to validate that the
installed libraries, toolchain, CMake configuration, and runtime
environment work together after mounting the ISO.

The intended structure is:

``` text
WinDevLibs.iso
└── sample_test_codes
    ├── Qt
    ├── VTK
    ├── Qt_VTK
    ├── ITK
    └── OpenCV
```

Each sample should preferably be a small self-contained CMake project:

``` text
hello_vtk
├── CMakeLists.txt
├── main.cpp
└── README.md
```

The samples are validation artifacts rather than part of the library
installation itself. A successful sample build provides a quick smoke
test for the corresponding WinDev environment.

## 13.7 Testing Qt from the Mounted WinDevLibs ISO

The Qt installation was tested after copying it into the separate
library ISO and mounting that ISO as a different drive from WinDevCore.

For example, with the library ISO mounted as `F:` the Qt binaries are
at:

``` text
F:\mingw\qtbase\6.8.4\bin
```

The same Qt Widgets + `.ui` + CMake + Ninja test application used during
development can locate Qt through:

``` bat
-DCMAKE_PREFIX_PATH="F:/mingw/qtbase/6.8.4"
```

The test confirmed that the Qt installation is not tied to a particular
absolute drive location. The location can be supplied through
`CMAKE_PREFIX_PATH`, while the WinDev library loader automatically
exposes the corresponding Qt `bin` directory through `PATH`.

The Qt test application was successfully built, executed, and deployed
using the WinDev deployment workflow.

## 13.6 Shared Qt and Deployment

WinDev uses the shared Qt build for the current MinGW library
environment. This is appropriate for the reusable library ISO model
because the Qt DLLs can be shared by multiple applications and
deployment can be handled by the existing WinDev deployment tooling.

For Qt applications, the active Qt installation provides:

``` text
Qt DLLs
windeployqt.exe
Qt CMake package configuration
```

CPLDD remains responsible for resolving the active compiler/runtime DLL
dependencies, while `windeployqt` handles Qt-specific deployment when
applicable.

------------------------------------------------------------------------
