# Chapter 1 — Why WinDev?

## 1.1 The Problem

Modern scientific and engineering software rarely depends on a single compiler, language, or library. A typical computational development environment can involve C/C++, Python, multiple compilers (MSVC, MinGW-w64, Clang), CMake, Qt, VTK, ITK, OpenCV, BLAS/LAPACK implementations, numerical libraries, Python bindings, build tools, platform SDKs, shell utilities, environment variables, and a web of DLLs and runtime dependencies. Individually, none of these is hard to install or configure. The real difficulty, the part that consumes days rather than minutes, lies in how they interact once assembled together.

**Compiler and ABI mismatches.** A library compiled with MSVC and a library compiled with MinGW-w64 do not necessarily agree on calling conventions, name mangling, exception handling, or the C++ runtime they link against. Code can compile cleanly, link without errors, and still crash or corrupt memory at runtime because two halves of the program disagree about how a `std::string` or `std::vector` is laid out in memory. These bugs rarely announce themselves at build time. They surface later, intermittently, and are disproportionately expensive to trace back to their root cause.

**Hidden and transitive dependencies.** A library like VTK or ITK doesn't just depend on a compiler. It depends on a specific version of a specific compiler, built against a specific runtime, often with its own vendored or expected versions of zlib, libpng, HDF5, or similar. Qt adds its own dependency tree and its own deployment tooling (`windeployqt`) that has to correctly resolve everything a plugin or module pulls in. One library's build settings can quietly constrain what every library above it in the stack is allowed to do.

**Stale build state.** CMake caches absolute paths, compiler identities, and found-package locations inside `CMakeCache.txt`. Move a source tree, rename a drive letter, swap a compiler, or relocate a prebuilt-library directory, and CMake will often keep using the old, now-invalid paths rather than failing loudly, leading to confusing errors that have nothing to do with the actual change that was made.

**Invisible runtime dependencies.** A DLL being present on disk doesn't mean the OS loader can find it. Windows resolves DLLs through a specific search order (the application's own directory, system directories, then `PATH`), and a DLL sitting in the wrong folder, or a `PATH` that wasn't updated after installing a new toolchain, produces the classic "it works on my machine" (or worse, "it worked yesterday") failure. This is precisely the class of problem tools like `ldd`-based dependency walkers exist to diagnose.

**Fragility under change.** Because every one of these components has implicit assumptions about the others (compiler ABI, runtime version, file paths, environment variables), a change to any single piece can invalidate an environment that had been stable for months. Upgrading one library, updating a compiler, or even just moving a directory can ripple outward in ways that are hard to predict in advance and just as hard to diagnose after the fact.

Taken together, these interactions are why assembling a scientific computing environment is not really an installation task. It's systems engineering. It demands version pinning, reproducible builds, isolation between toolchains, and tooling to inspect and verify dependencies rather than assume they're correct. 
**WinDev exists to address this problem.**

## 1.2 The PhD Experience

The motivation behind WinDev is not purely theoretical.

During a PhD, building and maintaining a computational environment can consume a surprising amount of time and effort. The difficult part is often not writing the scientific code itself, but getting all the surrounding software infrastructure to work together correctly.

Libraries have to be built. Compilers have to be configured. Dependencies have to be located. Build systems have to be understood. Environment variables have to be configured. Different versions have to be tested.

And when something breaks, the developer has to determine whether the problem comes from the compiler, the library, CMake, the linker, the runtime environment, or an incorrect configuration.

The deeper problem is that **the knowledge required to reproduce the environment accumulates gradually**.

After spending days solving a particular problem, the solution eventually becomes obvious to the person who solved it. Months or years later, however, the exact reason for a particular compiler flag, directory structure, patch, or environment variable may no longer be obvious.

This creates a dangerous situation:

> A working environment can become dependent on the memory of the person who built it.

WinDev is an attempt to eliminate that dependency.

## 1.3 Reproducibility

Reproducibility is one of the central ideas behind WinDev.

A development environment should not depend on:

- a particular Windows installation,
- a particular drive letter,
- undocumented configuration,
- manually remembered commands,
- globally installed libraries,
- accidental entries in the system `PATH`,
- or knowledge that exists only in the developer's memory.

Instead, the environment should have a defined structure and a documented procedure for reconstructing it.

The objective is not necessarily to reproduce every byte of a previous installation.

The objective is to reproduce its **development behavior**.

For example, WinDev provides controlled toolchain environments:

```text
WinDev
   │
   ├── Select MinGW
   │      └── GCC environment
   │
   ├── Select MSVC
   │      └── MSVC + Windows SDK
   │
   ├── Select ClangMSVC
   │      └── Clang-CL + MSVC environment
   │
   └── Select Python
          └── Portable Python environment
```

Once an environment is selected, the required tools and libraries should become available through a predictable mechanism.

The developer should not have to remember how that environment was assembled.

## 1.4 The Cost of Rebuilding an Environment

The cost of a development environment is not simply the disk space occupied by its software.

There is another, much larger cost:

**the time required to reconstruct it.**

Consider a library that took several hours to build correctly.

The final successful command might be only one line:

```text
cmake ...
```

But reaching that command may have required:

1. identifying the correct version,
2. choosing the compiler,
3. identifying compatible dependencies,
4. discovering an undocumented build option,
5. modifying a source file,
6. cleaning a stale CMake cache,
7. correcting an installation prefix,
8. resolving a linker problem,
9. fixing a runtime DLL problem,
10. testing the final installation.

The final command does not contain that history. Without documentation, that knowledge disappears.

WinDev therefore treats **the process of building the environment as knowledge that must itself be preserved.**

## 1.5 Complexity of Modern Scientific Software

Scientific computing increasingly combines several software ecosystems.

For example, a single application may eventually look like:

```text
                     Scientific Application
                              │
             ┌────────────────┼────────────────┐
             │                │                │
           Python             C++             GUI
             │                │                │
          NumPy          Armadillo            Qt
             │                │                │
             └────────── OpenBLAS ─────────────┘
                              │
                         Visualization
                              │
                             VTK
```

Another project may introduce OpenCV, ITK, VTK, Qt, Python, CMake, OpenMP, and BLAS/LAPACK together.

The challenge is therefore no longer simply:

> “Can I install this library?”

The real question becomes:

> **“Can I maintain a coherent development ecosystem in which these components can coexist, be rebuilt, tested, and reproduced?”**

WinDev is designed around that larger problem.

## 1.6 Goals of WinDev

### 1.6.1 Portability

The development environment should be portable rather than tightly coupled to a particular Windows installation.

The environment should be capable of being moved, mounted, or reconstructed without requiring a complete reinstallation of the development stack.

### 1.6.2 Isolation

Different toolchains should be able to coexist without unnecessarily interfering with one another.

For example, MinGW, MSVC, ClangMSVC, MSYS2, and Python should not become one uncontrolled global environment. WinDev instead provides controlled environments through its initialization and toolchain-loading mechanisms.

### 1.6.3 Reproducibility

Versions, build procedures, installation locations, patches, and configuration decisions should be documented.

A future rebuild should be based on documentation rather than memory.

### 1.6.4 Modularity

The environment should be composed of independent components.

The WinDev architecture therefore separates:

```text
WinDevCore
WinDevLibs
Projects
```

This allows development tools and development libraries to evolve independently.

### 1.6.5 Multiple Toolchains

Scientific software sometimes needs to be tested with different compiler ecosystems. WinDev therefore supports multiple toolchains rather than treating one compiler as the universal solution.

The same project can potentially be evaluated under MinGW, MSVC, or ClangMSVC depending on its requirements.

### 1.6.6 Controlled Dependencies

Libraries should not be scattered arbitrarily across the host operating system.

WinDev provides a controlled library hierarchy so that dependencies can be located through the WinDev environment rather than through a collection of manually configured system paths.

### 1.6.7 Documented Knowledge

Perhaps the most important goal is the preservation of knowledge.

WinDev documentation should record not only:

> **what works**

but also:

> **why it works, how it was built, what failed, and how the failure was solved.**

A successful build command is useful. A successful build command accompanied by its reasoning, dependencies, patches, known failures, and verification procedure is far more valuable.

## 1.7 What WinDev Is Not

WinDev is not intended to be merely:

- a collection of compilers,
- a folder containing portable applications,
- a replacement for an IDE,
- or a script that happens to configure `PATH`.

It is a **structured development environment**.

Its purpose is to provide a reproducible relationship between:

```text
User
  ↓
ConEmu
  ↓
WinDev bootstrap
  ↓
Toolchain
  ↓
Libraries
  ↓
Build system
  ↓
Project
```

The mechanisms implementing this architecture are described in subsequent chapters.

## 1.8 The Central Idea

The entire WinDev philosophy can be summarized in one principle:

> **The development environment is part of the project.**

Source code alone is not enough.

If a scientific application depends on a particular compiler, library version, build configuration, runtime environment, and collection of tools, then those dependencies form part of the application's effective development environment.

WinDev attempts to make that environment **portable, isolated, reproducible, modular, testable, and documented.**

And ultimately, the purpose is simple:

> **Future-you should not have to rediscover what present-you already learned the hard way.**
