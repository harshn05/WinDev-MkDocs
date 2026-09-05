# Toolchains

WinDev keeps toolchain initialization separate from general-purpose
programs.

``` text
WinDev::Python
WinDev::MinGW
WinDev::MSVC
WinDev::ClangMSVC
WinDev::Python+MinGW
WinDev::Python+MSVC
WinDev::Python+ClangMSVC
```

`CmdInit.cmd` establishes the common environment. Toolchain loaders then
activate the compiler/language-specific environment and, where
applicable, the matching `WinDevLibs` namespace.
