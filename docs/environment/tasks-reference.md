# ConEmu Task Reference

ConEmu tasks are configured through **ConEmu → Settings → Startup →
Tasks**.

  Task                         Shortcut         Environment
  ---------------------------- ---------------- ---------------------
  `WinDev::Python`             `Ctrl+Shift+P`   WinPython 3.12
  `WinDev::MinGW`              `Ctrl+Shift+T`   MinGW-w64 / GCC
  `WinDev::MSVC`               `Ctrl+Shift+V`   MSVC + Windows SDK
  `WinDev::ClangMSVC`          `Ctrl+Shift+C`   clang-cl + MSVC
  `WinDev::Python+MinGW`       Task menu        Python + MinGW
  `WinDev::Python+MSVC`        Task menu        Python + MSVC
  `WinDev::Python+ClangMSVC`   Task menu        Python + Clang-MSVC

The default startup task is `WinDev::MinGW`.

The generated ConEmu XML is not manually maintained; the documentation
records the GUI configuration and command lines.
