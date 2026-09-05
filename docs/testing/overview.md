# Testing

WinDev uses small known-good projects as smoke tests for toolchains,
libraries, CMake configuration, and runtime behavior.

``` text
Toolchain
   ↓
Library environment
   ↓
CMake configure
   ↓
Build
   ↓
Run
   ↓
Deployment / dependency check
```

The sample projects are stored under `sample_test_codes` in
`WinDevLibs.iso` so the same validation workflow can be repeated after
mounting the library payload.
