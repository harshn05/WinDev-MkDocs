# Python

# WinPython

WinDev uses **WinPython 3.12** as its portable Python distribution.

The current installation is:

``` text
%WINDEV%\WPy64-312101
```

The Python executable is:

``` text
%WINDEV%\WPy64-312101\python\python.exe
```

Python headers and libraries are also available from the portable Python
installation.

The environment is loaded through a dedicated toolchain script.

## 12.1 Python Toolchain Loader

The WinPython loader is:

``` text
%ConEmuBaseDir%\_Toolchains\loadenv_WinPython.cmd
```

Its current contents are:

``` bat
@set PYTHONHOME=%WINDEV%\WPy64-312101\python
@set PYTHON_INCLUDE_DIR=%PYTHONHOME%\include
@set PYTHON_LIBRARY=%PYTHONHOME%\libs\python312.lib
@set PYTHON_PATH=%PYTHONHOME%
@set PYTHON_LIB=%PYTHONHOME%\libs
@set INCLUDE=%PYTHONHOME%\include;%INCLUDE%
@set LIB=%PYTHONHOME%\libs;%LIB%
@set PYTHON_VERSION=312

@call "%WINDEV%\WPy64-312101\scripts\env.bat"

@echo.
@echo ------------------------------------------------------------
@echo   WinDev :: Python
@echo ------------------------------------------------------------
@python --version
@echo   Home : %PYTHONHOME%
@echo ------------------------------------------------------------
@echo.
```

This configures:

-   `PYTHONHOME`
-   Python include directory
-   Python library
-   Python library directory
-   `INCLUDE`
-   `LIB`
-   Python version information

and finally invokes WinPython's own `env.bat`.

## 12.2 Python Packages

The following packages have been installed using `pip`:

``` text
mayavi
art
pyvista
mkdocs-material
```

Installation command:

``` bat
pip install mayavi art pyvista mkdocs-material
```

The Python environment is intended to remain self-contained within
WinDev.

------------------------------------------------------------------------
