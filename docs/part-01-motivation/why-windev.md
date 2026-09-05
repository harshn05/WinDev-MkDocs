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

The motivation behind WinDev is not purely theoretical. It comes directly out of my own PhD experience.

During my PhD, I found that building and maintaining a computational environment could consume a surprising amount of time and effort, often more than the scientific work the environment was supposed to support. The difficult part was rarely writing the code itself. The code, once the environment was in place, was often the easy part. What actually ate the hours, days, and sometimes weeks was getting the surrounding software infrastructure to work together correctly, and then keeping it working as things changed around it.

Libraries had to be built, often from source, often against a specific compiler that the library's own documentation only partially specified. Compilers had to be configured, and configured consistently across every tool that touched them, from the IDE to the build system to the linker. Dependencies had to be located, and half the time a dependency wasn't missing so much as present in the wrong version, in the wrong place, or built with the wrong flags. Build systems like CMake had to be understood well enough to read through their error messages, which often described the symptom rather than the cause. Environment variables had to be set correctly, and forgotten just as easily, especially after a system update or a new machine. Different versions of the same library or compiler had to be tested against each other, because the "latest" version was not always the one that actually worked with everything else already in place.

And when something inevitably broke, I had to work out, usually under time pressure, whether the problem was coming from the compiler, the library, CMake, the linker, the runtime environment, or simply a configuration I had gotten wrong myself. Each of these failure modes looks similar from the outside: a build that won't complete, or a program that crashes for no obvious reason. Distinguishing between them required experience I didn't have yet the first time I encountered them, and had to build up the hard way, one broken environment at a time.

The deeper problem, though, wasn't any single bug. It was that the knowledge required to reproduce the environment accumulated gradually, and mostly in my head. After spending days chasing down a particular problem, the eventual solution would feel obvious, almost embarrassingly so, once I finally understood it. But that sense of obviousness was deceptive. It existed only because I had just spent days building the context needed to see it. Months or years later, when I looked back at a particular compiler flag, a specific directory structure, a small patch to a library's source, or an environment variable I had set, the original reasoning behind it was often gone. I knew it had mattered. I no longer knew why.

This creates a genuinely dangerous situation, one I ran into more than once: a working environment becomes dependent on the memory of the person who built it. As long as I remembered why things were arranged the way they were, everything worked. The moment that memory faded, or I moved to a new machine, or someone else needed to reproduce what I had built, the environment became fragile in a way that had nothing to do with the code itself and everything to do with lost context.

WinDev is my attempt to eliminate that dependency, to move the knowledge out of my head and into something durable, versioned, and reproducible, so that a working environment no longer depends on anyone's memory, including my own.

## 1.3 Reproducibility

Reproducibility is one of the central ideas behind WinDev, and it is worth being precise about what that word actually means in this context.

A development environment should not depend on any of the following:

- a particular Windows installation, with its own accumulated registry state and installed software history,
- a particular drive letter, which can shift the moment a USB drive, network share, or external disk is plugged in differently,
- undocumented configuration, the kind that exists only as a setting someone changed once and never wrote down,
- manually remembered commands, run in a particular order that isn't recorded anywhere,
- globally installed libraries, which silently couple every project on the machine to whatever version happens to be installed system-wide,
- accidental entries in the system `PATH`, left behind by some other installer and now quietly resolving a name to the wrong binary,
- or knowledge that exists only in the developer's memory, which is exactly the failure mode described earlier: an environment that works only as long as one person remembers why.

Each of these is a hidden dependency. None of them shows up in a project's source code or its documentation, yet all of them can determine whether a build succeeds or fails. A reproducible environment has to be defined independently of all of them. Instead of relying on the accumulated, undocumented state of a particular machine, the environment should have a defined structure and a documented procedure for reconstructing it, one that produces the same result on a different machine, a different drive letter, or the same machine after a clean reinstall.

It's worth being clear about what reproducibility does not require. The objective is not to reproduce every byte of a previous installation, down to the exact registry entries or the exact filesystem layout. That standard is both impractical and unnecessary. The objective is to reproduce its **development behavior**: given the same source code and the same selected toolchain, the build should compile, link, and run the same way, regardless of which physical machine or drive it happens to be running from.

WinDev approaches this by providing controlled, self-contained toolchain environments that the developer selects explicitly, rather than inheriting whatever happens to be configured globally:

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

Each branch of this structure is isolated from the others. Selecting MinGW should not leak MSVC's headers or libraries into the build, and selecting MSVC should not accidentally pick up a stray GCC binary that happens to be earlier in the `PATH`. Once an environment is selected, the required tools, compilers, and libraries should become available through a single predictable mechanism, consistently, every time, regardless of what else is installed on the machine.

Crucially, the developer should not have to remember how that environment was assembled. The knowledge that once lived only in memory, which compiler flag mattered and why, which library version was compatible with which toolchain, which environment variable had to be set before which tool would run correctly, is meant to live in WinDev's structure instead. Selecting an environment should be enough. Remembering how it was built should not be necessary.

## 1.4 The Cost of Rebuilding an Environment

The cost of a development environment is not simply the disk space occupied by its software. Disk space is cheap and easy to measure. There is another, much larger cost, one that rarely gets budgeted for and almost never gets documented: **the time required to reconstruct it.**

Consider a library that took several hours, or in some cases days, to build correctly. Once it finally works, the final successful command might be reduced to something deceptively short:

```text
cmake ...
```

Looking at that single line in isolation, it would be reasonable to assume the whole process took a few minutes. It doesn't show the hours of work that preceded it, and it gives no hint of how fragile the path to that command actually was. Reaching that one working invocation may have required a long, often nonlinear sequence of steps, something closer to this:

1. identifying the correct version of the library, since not every version is compatible with the rest of the environment, and the most recent release is not always the right choice,
2. choosing the compiler, and confirming that the chosen compiler actually produces a binary compatible with everything else the library needs to link against,
3. identifying compatible dependencies, often by trial and error, since compatibility is rarely stated explicitly and version constraints are often implicit,
4. discovering an undocumented build option, buried in a forum post, a mailing list archive, or a comment inside the library's own build scripts, without which the build fails or silently misconfigures itself,
5. modifying a source file directly, because a bug, a missing platform check, or an assumption baked into the code simply doesn't hold for this particular combination of compiler and OS,
6. cleaning a stale CMake cache, after wasting time debugging an error that had nothing to do with the actual code and everything to do with a cached path from a previous, unrelated build,
7. correcting an installation prefix, so that the library installs where the rest of the environment expects to find it, rather than where its defaults happen to place it,
8. resolving a linker problem, tracing an unresolved symbol or a duplicate definition back to a mismatch between library versions or calling conventions,
9. fixing a runtime DLL problem, discovering that the build succeeded but the resulting binary can't actually run because a dependency isn't where the loader expects it,
10. testing the final installation, to confirm that what was built doesn't just compile, but actually behaves correctly when used from the rest of the environment.

Each of these steps can consume anywhere from minutes to hours on its own, and any one of them, if skipped or forgotten the next time, is enough to make the build fail again from scratch. The final command does not contain any of that history. It is the last line of a much longer story, and it gives no indication that the story exists at all. Without documentation, that knowledge disappears, not gradually, but almost immediately, since the details of steps 4 through 9 are exactly the kind of thing that feels obvious right after solving them and becomes completely opaque a few months later.

WinDev therefore treats **the process of building the environment as knowledge that must itself be preserved.** The one-line command is not the artifact worth keeping. The path that led to it is.

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
