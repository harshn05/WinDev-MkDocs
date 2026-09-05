# Libraries

Third-party development libraries are maintained separately from
WinDevCore in `WinDevLibs.iso`.

``` text
WinDevLibs.iso
├── mingw
│   ├── qtbase\6.8.4
│   └── vtk\9.4.2
├── msvc
│   └── qtbase\6.8.4
└── sample_test_codes
```

Each toolchain has its own library namespace. `CmdInit.cmd` discovers
the ISO root, and the active toolchain loader activates compatible
libraries.
