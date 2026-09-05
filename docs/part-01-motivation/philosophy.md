# Chapter 2 — WinDev Philosophy

WinDev is not defined by a particular compiler, library, shell, or directory structure. Tools and versions will change over time. What should remain stable are the principles by which the development environment is assembled, isolated, maintained, and reproduced.

The philosophy of WinDev is therefore less about choosing specific software and more about controlling the relationships between the pieces that make up a development environment.

---

## 2.1 Environment as Infrastructure

A development environment is often treated as something that exists around a project: install a compiler, install some libraries, configure CMake, and then begin writing code.

WinDev takes a different view.

**The development environment is infrastructure.**

If an application depends on a particular compiler, a particular runtime, specific library versions, build tools, environment variables, and a known method of locating those dependencies, then those components are part of the environment required to develop and maintain that application.

The source code is only one part of the system.

```text
Application
    │
    ├── Source Code
    ├── Build System
    ├── Compiler
    ├── Libraries
    ├── Runtime
    └── Development Tools
```

A change to any of these components can change the behavior of the project.

This means that the development environment should be treated with the same discipline as the source code itself. It should have a defined structure, known versions, documented configuration, and a reproducible method of reconstruction.

WinDev is built around this idea.

---

## 2.2 Portability Over Installation

Traditional software installation assumes that the host operating system is the primary environment.

A compiler is installed into the system. Libraries are installed somewhere on the machine. Environment variables are modified globally. Additional tools are installed as they become necessary. Over time, the machine accumulates configuration.

This approach can work well for ordinary desktop software, but it becomes increasingly fragile for complex development environments.

WinDev instead treats portability as a design goal.

The objective is not simply:

> **Install everything on Windows.**

The objective is:

> **Carry a known development environment with you.**

The environment should be capable of being moved, mounted, reconstructed, or replaced without requiring the entire development stack to be manually rebuilt from memory.

This does not mean that every byte of the environment must be identical on every machine. It means that the important characteristics of the environment should not depend unnecessarily on the physical machine on which it happens to be running.

Portability therefore reduces the dependency on the host system and makes the development environment itself a controlled object.

---

## 2.3 Isolation Over Global Configuration

One of the easiest ways to create an unreliable development environment on Windows is to allow every tool to modify the global environment.

A typical machine can gradually accumulate something like:

```text
System PATH
    │
    ├── Python
    ├── MinGW
    ├── MSVC tools
    ├── CMake
    ├── Qt
    ├── other DLL directories
    ├── another Python
    └── another compiler
```

Eventually, it becomes difficult to determine which executable, header, library, or DLL a build is actually using.

WinDev prefers controlled environments.

```text
WinDev
   │
   ├── MinGW environment
   │
   ├── MSVC environment
   │
   ├── ClangMSVC environment
   │
   ├── MSYS2 environment
   │
   └── Python environment
```

Selecting an environment should determine which tools and dependencies become visible to the current development session.

Isolation does not mean that WinDev attempts to sandbox Windows or create completely independent operating-system instances. It means that the composition of the development environment is deliberate and controlled rather than being an accidental consequence of everything previously installed on the machine.

For example, selecting MinGW should not unexpectedly expose MSVC headers or libraries. Similarly, selecting MSVC should not cause a stray GCC installation somewhere on the system to become the compiler used by a build.

The goal is simple:

> **A selected environment should behave predictably.**

---

## 2.4 Reproducibility Over Memory

A reproducible environment cannot depend on information that exists only in someone's memory.

If a particular compiler flag is required, that flag should be recorded.

If a particular library version is known to work, that version should be recorded.

If a source patch is required, the patch should be recorded.

If a particular installation layout is important, the layout should be documented.

If a particular sequence of commands is required, that sequence should be reproducible.

The principle is:

> **If something is required for the environment to work, it should exist somewhere other than the developer's memory.**

This changes the role of documentation.

Documentation is no longer merely a description of how to use the software. It becomes part of the environment itself.

The knowledge accumulated while building the environment becomes a reproducible artifact.

```text
Experience
    ↓
Build decisions
    ↓
Commands
    ↓
Configuration
    ↓
Documentation
    ↓
Reproducible environment
```

The objective is not to eliminate the need for engineering judgment. Some problems will always require investigation.

The objective is to ensure that once a problem has been solved, the knowledge gained from solving it does not disappear.

---

## 2.5 Modularity Over Monolithic Installation

A large development environment should not have to be treated as one indivisible installation.

Different components have different lifecycles.

A compiler may need to be upgraded without rebuilding every library immediately. A library may need to be replaced without changing the shell environment. Development tools may evolve independently from prebuilt scientific libraries.

WinDev therefore separates the environment into logical components.

```text
WinDevCore
WinDevLibs
Projects
```

`WinDevCore` contains the tools and infrastructure required to operate the development environment.

`WinDevLibs` contains the development libraries and SDKs used by projects.

`Projects` contains the user's writable project data.

This separation allows components to evolve independently and makes it possible to replace or update one part of the environment without unnecessarily disturbing the others.

Modularity also applies within the library environment itself. Libraries can be organized by compiler family, version, and dependency scope rather than being scattered arbitrarily across the host system.

The principle is:

> **A component should be replaceable without requiring the entire environment to be rebuilt.**

---

## 2.6 Explicit Toolchains Over Implicit Defaults

A development environment should not depend on whichever compiler happens to be found first.

Modern Windows development may involve several valid compiler ecosystems:

```text
WinDev::MinGW
        ↓
       GCC

WinDev::MSVC
        ↓
       MSVC

WinDev::ClangMSVC
        ↓
      clang-cl
```

These toolchains may have different ABIs, runtimes, headers, libraries, linkers, and compatibility requirements.

WinDev therefore treats toolchain selection as an explicit operation.

The developer selects the environment first. The environment then establishes the appropriate compiler and supporting tools.

This is preferable to allowing the build system to discover an arbitrary compiler from the host machine.

The principle is:

> **The compiler used by a project should be a deliberate choice, not an accidental discovery.**

This also makes cross-toolchain testing practical. The same project can potentially be evaluated under MinGW, MSVC, and ClangMSVC without turning the host system into one large mixture of incompatible development components.

---

## 2.7 Controlled Dependencies

A dependency being present on a machine does not necessarily mean that it is the correct dependency.

A project may find:

- the wrong version,
- the wrong architecture,
- a library built with another compiler,
- an incompatible runtime,
- or an unrelated installation that happens to appear earlier in the search path.

WinDev therefore treats dependency location as part of environment design.

Instead of scattering libraries across the host operating system, WinDev provides a controlled library hierarchy from which dependencies can be exposed to the selected toolchain.

A simplified dependency relationship might look like:

```text
Application
    │
    └── Armadillo
            │
            └── OpenBLAS
```

The important question is not merely whether OpenBLAS exists.

The important questions are:

- Which OpenBLAS?
- Which version?
- Where is it installed?
- Which compiler is using it?
- Which headers are being included?
- Which library is being linked?
- Which runtime DLL is being loaded?

Controlled dependencies turn these questions from implicit assumptions into properties of the environment.

The principle is:

> **Dependencies should be known, intentional, and discoverable through predictable mechanisms.**

---

## 2.8 Documentation as Infrastructure

Documentation is often treated as something written after the real work is finished.

WinDev treats documentation differently.

**Documentation is part of the infrastructure.**

The purpose is not merely to record successful commands. A useful record should preserve the reasoning and context surrounding those commands.

WinDev documentation should record not only:

> **what works**

but also:

> **why it works, how it was built, what failed, and how the failure was solved.**

A successful build command is useful.

A successful build command accompanied by:

- the source version,
- compiler version,
- dependency versions,
- configuration options,
- installation prefix,
- patches,
- known failures,
- workarounds,
- and verification steps

is much more valuable.

This is especially important for problems whose solutions are not obvious from the final configuration.

A source patch that took several hours to discover may eventually be only three lines long. Those three lines are easy to preserve. The reasoning that explains why they were necessary is much easier to lose.

WinDev attempts to preserve both.

The principle is:

> **A workaround that is not documented is a future rediscovery waiting to happen.**

---

## 2.9 Build, Test, Verify

A successful build is not necessarily a successful environment.

A compiler can produce an executable that does not run.

A library can install successfully but fail when linked into another project.

A CMake configuration can complete while selecting an unintended dependency.

A program can launch while loading the wrong DLL.

WinDev therefore distinguishes between building something and verifying that it actually works.

A simplified workflow is:

```text
Configure
    ↓
Build
    ↓
Install
    ↓
Link
    ↓
Run
    ↓
Verify
```

Verification should be proportional to the importance of the component.

For a library, verification may include a small test program.

For a toolchain, verification may include compiler identification and a simple compilation.

For a complex dependency stack, verification may require testing the actual combination of components that the environment is intended to support.

The principle is:

> **"It built" is not the same as "it works."**

WinDev therefore treats testing and verification as part of environment construction rather than as an optional step performed afterwards.

---

## 2.10 Pragmatism Over Perfection

Real development environments are rarely perfect.

Upstream projects can contain platform-specific assumptions. Build systems can make incorrect assumptions about a compiler. Dependencies can change behavior between releases. A compiler update can expose previously hidden problems.

Sometimes the correct solution is to use an upstream-supported configuration.

Sometimes the correct solution is to change a build option.

Sometimes a clean patch is required.

And sometimes a small, carefully documented workaround is the most practical solution available.

WinDev does not attempt to pretend that these situations do not exist.

The preferred approach is always to use clean and maintainable solutions where possible. However, when a workaround is necessary, it should be:

1. understood,
2. reproducible,
3. documented,
4. isolated where practical,
5. and verified.

The important distinction is between an undocumented hack and a documented engineering decision.

The principle is:

> **Prefer clean solutions, but document pragmatic solutions when reality requires them.**

A workaround that is understood and recorded becomes part of the environment's knowledge rather than becoming unexplained technical debt.

---

## 2.11 The WinDev Design Principles

The philosophy of WinDev can therefore be summarized in a small set of principles.

| Principle | Meaning |
|---|---|
| **Portability** | The environment should not depend unnecessarily on one physical machine. |
| **Isolation** | Toolchains should coexist without uncontrolled interference. |
| **Reproducibility** | The environment should be reconstructible from recorded knowledge. |
| **Modularity** | Components should be replaceable and maintainable independently. |
| **Explicitness** | Toolchain and dependency selection should be deliberate. |
| **Control** | Dependencies should come from known and predictable locations. |
| **Verification** | A successful build should not be assumed to mean a working environment. |
| **Documentation** | Knowledge should survive beyond the person who discovered it. |
| **Pragmatism** | Real-world workarounds are acceptable when properly understood and documented. |

These principles are more important than any particular software version.

GCC versions will change. Visual Studio versions will change. Qt, VTK, ITK, OpenCV, CMake, Python, and other dependencies will change. Directory layouts may evolve as the environment grows.

The principles should remain.

WinDev is therefore not defined by the particular tools it contains.

It is defined by **how those tools are assembled, isolated, maintained, tested, documented, and reproduced.**

The implementation will evolve.

The philosophy should not.

---

> **WinDev is not a collection of software. It is a method for building and maintaining a development environment.**
