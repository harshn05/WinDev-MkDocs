# Updating WinDev

WinDev is maintained as a versioned portable environment.

When updating a component:

1.  Keep it inside the intended WinDev layer.
2.  Avoid hard-coded user-specific paths.
3.  Validate its loader.
4.  Run a representative smoke test.
5.  Record the exact version and configuration.
6.  For library payloads, install into a writable staging directory
    before copying into the read-only `WinDevLibs.iso`.

Generated vendor setup scripts such as the MSVC `setup_x64.bat` are
retained rather than unnecessarily recreated.
