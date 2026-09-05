# MSVC

# Portable MSVC

WinDev includes a standalone, portable MSVC toolchain together with the
Windows SDK.

The MSVC package is stored directly at the WinDev root, alongside
WinPython and MinGW:

``` text
%WINDEV%\MSVC
```

The package has the following high-level structure:

``` text
MSVC
├── VC
│   └── Tools
│       └── MSVC
│           └── 14.51.36231
│
├── Windows Kits
│   └── 10
│       └── ...
│
└── setup_x64.bat
```

The current configuration is:

``` text
MSVC Tools : 14.51.36231
Windows SDK: 10.0.28000.0
Host       : x64
Target     : x64
```

## 14.1 Creating the Portable MSVC Package

The portable MSVC package was created using the Python script from:

``` text
https://gist.github.com/mmozeiko/7f3162ec2988e81e56d5c4e22cde9977
```

The script was used to download and unpack the required standalone
Microsoft Visual C++ components and Windows SDK components without
installing the full Visual Studio IDE.

The selected versions were:

``` text
MSVC v14.51
Windows SDK v28000
```

During setup, the script requested acceptance of the Microsoft Visual
Studio license before downloading the components.

The download log reported:

``` text
Total downloaded: 479 MB
Done!
```

The resulting package contains the compiler and linker toolset, MSVC
runtime headers/libraries, Windows SDK headers/libraries, and supporting
components required for native x64 C/C++ development.

The generated `setup_x64.bat` is retained as part of the portable
package and is used by WinDev rather than being replaced by a custom
recreation of Microsoft's environment setup.

## 14.2 MSVC Environment Setup

The generated setup script is:

``` text
%WINDEV%\MSVC\setup_x64.bat
```

It configures the native x64 MSVC environment:

``` bat
set VSCMD_ARG_HOST_ARCH=x64
set VSCMD_ARG_TGT_ARCH=x64

set VCToolsVersion=14.51.36231
set WindowsSDKVersion=10.0.28000.0\
```

It then adds the compiler, linker, Windows SDK, UCRT, include, and
library directories to:

``` text
PATH
INCLUDE
LIB
```

The setup script uses `%~dp0` for its paths. This is important for
portability: the MSVC package can be moved with the WinDev directory
without rewriting the internal setup paths.

The generated setup script is intentionally kept unchanged.

## 14.3 WinDev MSVC Loader

WinDev provides a small wrapper to activate the generated MSVC
environment:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_MSVC.cmd
```

The wrapper calls:

``` bat
call "%WINDEV%\MSVC\setup_x64.bat"
```

and then reports the active configuration.

The loader is:

``` bat
@echo off

@call "%WINDEV%\MSVC\setup_x64.bat"

@echo.
@echo ------------------------------------------------------------
@echo   WinDev :: MSVC
@echo ------------------------------------------------------------
@echo   MSVC Tools : %VCToolsVersion%
@echo   Windows SDK: %WindowsSDKVersion%
@echo   Host       : %VSCMD_ARG_HOST_ARCH%
@echo   Target     : %VSCMD_ARG_TGT_ARCH%
@echo ------------------------------------------------------------
@echo.

rem ============================================================
rem Load MSVC libraries
rem ============================================================

if defined WINDEV_LIBS (
    echo [WinDev] WinDevLibs detected: %WINDEV_LIBS%

    if exist "%WINDEV_LIBS%\msvc" (
        call "%ConEmuBaseDir%\_Toolchains\loadenv_MSVC_Libs.cmd"
    ) else (
        echo [WinDev] MSVC libraries not found in WinDevLibs.
    )
) else (
    echo [WinDev] WinDevLibs not mounted.
    echo [WinDev] Continuing with MSVC only.
)

rem ============================================================
rem Status
rem ============================================================

echo.
echo [WinDev] MSVC environment active.
echo [WinDev] CC=%CC%
echo [WinDev] CXX=%CXX%

if defined WINDEV_LIBS (
    echo [WinDev] Libraries=%WINDEV_LIBS%\msvc
)

echo.
```

The MSVC loader therefore remains the user-facing entry point for the
MSVC environment. If `WinDevLibs` is mounted and contains the MSVC
library tree, compatible libraries are activated automatically.

## 14.4 MSVC Library Environment

The MSVC library loader is:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_MSVC_Libs.cmd
```

Its purpose is to activate libraries built for the MSVC environment
without hard-coding the `WinDevLibs` drive letter.

The current library layout is:

``` text
WinDevLibs.iso
└── msvc
    └── qtbase
        └── 6.8.4
```

Current working loader:

``` bat
@echo off

rem ============================================================
rem WinDev - MSVC Libraries Environment
rem ============================================================

echo.
echo ------------------------------------------------------------
echo  WinDev :: MSVC Libraries
echo ------------------------------------------------------------

if not defined WINDEV_LIBS (
    echo [WinDev] WINDEV_LIBS is not defined.
    echo [WinDev] No MSVC libraries loaded.
    exit /b 0
)

if not exist "%WINDEV_LIBS%\msvc" (
    echo [WinDev] MSVC library directory not found:
    echo          %WINDEV_LIBS%\msvc
    exit /b 0
)

set "WINDEV_MSVC_LIBS=%WINDEV_LIBS%\msvc"

echo [WinDev] MSVC libraries: %WINDEV_MSVC_LIBS%

rem ============================================================
rem Qt
rem ============================================================

set "QT_ROOT=%WINDEV_MSVC_LIBS%\qtbase\6.8.4"

if exist "%QT_ROOT%" (
    set "PATH=%QT_ROOT%\bin;%PATH%"
    set "CMAKE_PREFIX_PATH=%QT_ROOT%;%CMAKE_PREFIX_PATH%"

    echo [WinDev] Qt 6.8.4       : enabled
) else (
    echo [WinDev] Qt 6.8.4       : not found
)

rem ============================================================
rem Future libraries
rem ============================================================
rem
rem VTK
rem ----
rem set "VTK_ROOT=%WINDEV_MSVC_LIBS%\vtk\..."
rem
rem ITK
rem ----
rem set "ITK_ROOT=%WINDEV_MSVC_LIBS%\itk\..."
rem
rem OpenCV
rem ------
rem set "OpenCV_ROOT=%WINDEV_MSVC_LIBS%\opencv\..."

echo.
echo [WinDev] MSVC library environment initialized.
echo.
```

As with MinGW, the normal workflow does not require manual `PATH` or
`CMAKE_PREFIX_PATH` edits. The MSVC toolchain loader automatically
invokes the MSVC library loader when the corresponding library tree is
available.

This keeps the WinDev-specific presentation separate from the generated
Microsoft setup script.

## 14.4 MSVC Components

The downloaded MSVC package included components such as:

``` text
Microsoft.VC.14.51.Tools.HostX64.TargetX64.base.vsix
Microsoft.VC.14.51.Premium.Tools.HostX64.TargetX64.base.vsix
Microsoft.VC.14.51.CRT.Headers.base.vsix
Microsoft.VC.14.51.CRT.Redist.X64.base.vsix
Microsoft.VC.14.51.CRT.x64.Desktop.base.vsix
Microsoft.VC.14.51.PGO.Headers.base.vsix
Microsoft.VC.14.51.PGO.X64.base.vsix
Microsoft.VisualCpp.DIA.SDK.vsix
```

The Windows SDK download included desktop headers and libraries and
Universal CRT headers/libraries, along with other SDK components used by
the toolchain.

This provides a complete native x64 compiler environment rather than
only a standalone `cl.exe`.

## 14.5 Testing the MSVC Environment

After launching `WinDev::MSVC`, the environment can be checked with:

``` bat
where cl
where link
cl
```

The active toolchain should provide:

``` text
cl.exe
link.exe
MSVC headers
MSVC libraries
Windows SDK headers
Windows SDK libraries
```

CMake and Ninja can be used with the active MSVC environment:

``` bat
cmake -S . -B build -G Ninja
cmake --build build
```

------------------------------------------------------------------------
