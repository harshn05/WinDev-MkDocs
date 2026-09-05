# Build Aliases

## Generic Ninja

Defined by `CmdInit.cmd`:

``` bat
doskey ninjabuild=cmake -GNinja -S . -B ninjabuild -DCMAKE_BUILD_TYPE=Release $*
```

## MinGW

Defined by `loadenv_MinGW.cmd`:

``` bat
doskey gccbuild=cmake -G "MinGW Makefiles" -S . -B gccbuild -DCMAKE_BUILD_TYPE=Release $*
```

## Clang-MSVC

Defined by `loadenv_ClangMSVC.cmd`:

``` bat
doskey clangbuild=cmake -GNinja -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl -DOpenMP_libomp_LIBRARY="%WINDEV%\clang+llvm-23.1.0-x86_64-pc-windows-msvc\lib\libomp.lib" -S . -B clangbuild -DCMAKE_BUILD_TYPE=Release $*
```

`ninjabuild` uses the currently active compiler environment. The
compiler-specific aliases select additional toolchain-specific behavior.
