# Deployment / CPLDD

# CPLDD Deployment Utility

WinDev includes a standalone deployment utility named **CPLDD**.

CPLDD is designed to simplify deployment of Windows C/C++ applications
by inspecting DLL dependencies reported by `ldd` and copying the
required non-system runtime DLLs into a deployment directory.

For Qt applications, CPLDD can also invoke `windeployqt` when it is
available in the active environment.

The source project is maintained separately:

``` text
https://github.com/harshn05/cpldd
```

## 29.1 Standalone CPLDD

CPLDD is packaged as a standalone Windows executable using PyInstaller.

The WinDev copy is placed in:

``` text
%ConEmuBaseDir%\_Tools\cpldd.exe
```

`_Tools` is added to `PATH` by `CmdInit.cmd`, so CPLDD is available
directly from any initialized WinDev terminal:

``` bat
cpldd --help
```

No Python environment needs to be loaded to run CPLDD.

The standalone executable contains its Python runtime internally. It
does not bundle the active toolchain's `ldd`, `cygpath`, or
`windeployqt`. Those tools continue to come from the currently active
environment.

This is intentional because CPLDD should deploy an application according
to the compiler/runtime environment in which it is being used.

## 29.2 Deployment Workflow

The intended workflow is:

``` text
Build application
       │
       ▼
Load appropriate WinDev toolchain
       │
       ▼
Run CPLDD
       │
       ▼
ldd resolves DLL dependencies
       │
       ▼
CPLDD copies non-system DLLs
       │
       ▼
windeployqt
(if available and applicable)
       │
       ▼
Deployment directory
```

For example:

``` bat
cpldd myapp.exe
```

If no output directory is supplied, CPLDD uses the executable name as
the default deployment directory.

For:

``` text
myapp.exe
```

the default output is:

``` text
myapp\
    myapp.exe
    ...
```

A custom output directory can be supplied explicitly:

``` bat
cpldd myapp.exe release
```

## 29.3 System DLL Handling

Windows DLLs resolved from locations such as:

``` text
%WINDIR%\System32
%WINDIR%\SysWOW64
%WINDIR%\WinSxS
```

are ignored by default.

This prevents Windows components such as `COMCTL32.dll` from being
accidentally copied from the Windows `WinSxS` store into an application
deployment directory.

System DLL copying can be forced with:

``` bat
cpldd myapp.exe --force
```

This should only be used when there is a specific reason to deploy
Windows system components.

## 29.4 Qt Deployment

When `windeployqt` is available, CPLDD can invoke it for Qt
applications.

Qt deployment can be disabled with:

``` bat
cpldd myapp.exe --no-qt
```

CPLDD itself does not require Qt.

## 29.5 Toolchain Awareness

CPLDD uses the active environment rather than hard-coding one compiler.

Typical usage is therefore:

``` text
WinDev::MinGW
    │
    ├── MinGW runtime
    ├── ldd
    └── cpldd
```

or:

``` text
WinDev::Clang-MSVC
    │
    ├── LLVM/MSVC runtime
    ├── ldd
    ├── windeployqt
    └── cpldd
```

This is especially useful for the WinDev environment because multiple
C/C++ toolchains coexist without being mixed into one global
environment.

## 29.6 Building the Standalone Executable

The current Python implementation can be packaged with:

``` bat
pyinstaller --clean --noconfirm --onefile --console --name cpldd cpldd.py
```

The resulting executable is:

``` text
dist\
└── cpldd.exe
```

The executable is then copied into:

``` text
%ConEmuBaseDir%\_Tools
```

After restarting or reinitializing the WinDev shell:

``` bat
where cpldd
cpldd --help
```

should resolve the WinDev copy.

------------------------------------------------------------------------
