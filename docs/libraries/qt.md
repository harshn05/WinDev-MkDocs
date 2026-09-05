# Qt

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
