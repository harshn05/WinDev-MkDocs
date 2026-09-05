# Sample Projects

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

------------------------------------------------------------------------
