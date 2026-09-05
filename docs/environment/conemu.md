# ConEmu

# ConEmu

The portable ConEmu package is located at:

``` text
%WINDEV%\ConEmuPack.230724
```

The ConEmu directory is:

``` text
%WINDEV%\ConEmuPack.230724\ConEmu
```

ConEmu provides the terminal environment in which the rest of WinDev is
initialized.

The ConEmu base directory is available through:

``` text
%ConEmuBaseDir%
```

This variable is used extensively by the WinDev configuration.

# WinDev Launcher

The main WinDev launcher is kept directly in the WinDev root:

``` text
%WINDEV%\Launch_WinDev.cmd
```

Current launcher:

``` bat
@echo off

for %%I in ("%~dp0") do set "WINDEV=%%~fI"
set "CONEMU=%WINDEV%ConEmuPack.230724\"
set "FONTDIR=%CONEMU%\ConEmu\_OhMyPosh\_Fonts"

start "" "%CONEMU%\ConEmu64.exe" -FontDir "%FONTDIR%"
```

The launcher determines `%WINDEV%` from its own location.

Therefore the complete WinDev directory can be moved without changing
the launcher.

# ConEmu Appearance

The current ConEmu color scheme is:

``` text
Cobalt2
```

The terminal prompt is customized separately using:

``` text
Oh My Posh
    +
PERU theme
    +
portable Nerd Font
```

# ConEmu Tasks

ConEmu Tasks provide predefined WinDev development environments.

Tasks are configured through:

**ConEmu → Settings → Startup → Tasks**

The generated ConEmu XML configuration is automatically produced by
ConEmu and is not manually maintained.

Documentation therefore describes the GUI configuration rather than the
generated XML.

# WinDev::Python Task

Create a task under:

**ConEmu → Settings → Startup → Tasks**

Configuration:

``` text
Group name:
WinDev::Python
```

Hotkey:

``` text
Ctrl+Shift+P
```

Task parameters:

``` text
/icon "%ConEmuBaseDir%\_Toolchains\_icons\python.ico"
```

Commands:

``` bat
cmd.exe /k RenameTab "Python"&"%ConEmuBaseDir%\CmdInit.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_WinPython.cmd"
```

The task:

1.  Starts a CMD session.
2.  Renames the tab to `Python`.
3.  Initializes the common WinDev environment.
4.  Loads WinPython.

# WinDev::MinGW Task

Create a task under:

**ConEmu → Settings → Startup → Tasks**

Configuration:

``` text
Group name:
WinDev::MinGW
```

Hotkey:

``` text
Ctrl+Shift+T
```

Task parameters:

``` text
/icon "%ConEmuBaseDir%\_Toolchains\_icons\mingw.ico"
```

Commands:

``` bat
cmd.exe /k RenameTab "MinGW"&"%ConEmuBaseDir%\CmdInit.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_MinGW.cmd"
```

The task:

1.  Starts a CMD session.
2.  Renames the tab to `MinGW`.
3.  Initializes the common WinDev environment.
4.  Loads the MinGW-w64 toolchain.

# WinDev::Python + MinGW Task

A combined development environment is also provided.

Task name:

``` text
WinDev::Python+MinGW
```

Commands:

``` bat
cmd.exe /k RenameTab "Python+MinGW"&"%ConEmuBaseDir%\CmdInit.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_WinPython.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_MinGW.cmd"
```

The task loads both toolchains into the same CMD session.

The resulting environment provides:

``` text
Python 3.12
+
GCC
+
G++
+
CMake
+
Ninja
+
Git
```

This is useful for projects involving Python/C++ interoperability,
native Python extensions, Cython, pybind11, scientific Python packages,
and C/C++ development.

# WinDev::MSVC Task

Task name:

``` text
WinDev::MSVC
```

Hotkey:

``` text
Ctrl+Shift+V
```

Task icon:

``` text
/icon "%ConEmuBaseDir%\_Toolchains\_icons\msvc.ico"
```

Commands:

``` bat
cmd.exe /k RenameTab "Visual C++"&"%ConEmuBaseDir%\CmdInit.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_MSVC.cmd"
```

The task initializes the common WinDev environment and then activates
the portable MSVC toolchain and Windows SDK.

# Default ConEmu Startup Task

ConEmu is configured to start directly with:

``` text
WinDev::MinGW
```

This is configured through:

**ConEmu → Settings → Startup → Task**

Set the startup task to:

``` text
WinDev::MinGW
```

Launching ConEmu therefore immediately produces a ready-to-use MinGW
development terminal.

# WinDev::Python + MSVC Task

Task name:

``` text
WinDev::Python+MSVC
```

Task icon:

``` text
/icon "%ConEmuBaseDir%\_Toolchains\_icons\python-msvc.ico"
```

Commands:

``` bat
cmd.exe /k RenameTab "Python+MSVC"&"%ConEmuBaseDir%\CmdInit.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_WinPython.cmd"&"%ConEmuBaseDir%\_Toolchains\loadenv_MSVC.cmd"
```

The task loads WinPython first and then activates the MSVC toolchain in
the same CMD session.

# C/C++ Build Stack

The current WinDev C/C++ development stack consists of:

``` text
Git
 │
 └── Source control

CMake
 │
 └── Build-system generation

Ninja
 │
 └── Build execution

MinGW-w64
 │
 ├── GCC
 ├── G++
 ├── GNU linker
 └── supporting development tools

MSVC
 │
 ├── Microsoft C/C++ compiler
 ├── linker
 ├── MSVC runtime / headers
 └── Windows SDK
```

A typical CMake + Ninja + MinGW workflow is:

``` bat
cmake -S . -B build -G Ninja
cmake --build build
```

Git is available independently for source control.

------------------------------------------------------------------------
