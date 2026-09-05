# VTK

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

------------------------------------------------------------------------
