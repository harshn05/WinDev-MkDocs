# Clink

# Clink

Clink is used to provide enhanced command-line editing, history,
completion, and suggestion features in the ConEmu environment.

The current Clink configuration uses Oh My Posh for the prompt and
enables the suggestion list by default:

``` bat
clink config prompt use oh-my-posh
clink set suggestionlist.default true
```

The second setting makes the Clink suggestion list available by default,
without requiring `F2` to activate it for every session.

Clink is used to enhance the CMD shell and provide Lua-based shell
customization.

Clink is installed inside:

``` text
%ConEmuBaseDir%\clink
```

The Clink directory is kept as part of the portable ConEmu environment.

## 5.1 Portable Clink and Oh My Posh

Clink is kept inside the portable ConEmu installation:

``` text
%ConEmuBaseDir%\clink
```

`CmdInit.cmd` adds this portable Clink directory to `PATH` and
initializes the Oh My Posh custom prompt. The Clink block is
intentionally placed **after WinDevLibs volume discovery** in
`CmdInit.cmd`:

``` bat
rem ============================================================
rem Clink
rem ============================================================

set "PATH=%ConEmuBaseDir%\clink;%PATH%"
clink config prompt use oh-my-posh
clink set clink.customprompt
```

The ordering is intentional. During testing, placing the Clink
initialization before the `WinDevLibs` discovery block caused the
PowerShell-based volume discovery to fail, even though `Get-Volume`
itself correctly reported the `WinDevLibs` volume. Moving the Clink
block to the end of `CmdInit.cmd` resolved the issue. The current
ordering is therefore treated as part of the working WinDev startup
configuration.

The prompt configuration is refreshed during every WinDev terminal
initialization. This is important for portability because Clink can
otherwise retain an absolute path to the `.clinkprompt` file in its
persistent user configuration.

For example, after renaming or moving the WinDev directory, an old
configuration may still point to:

``` text
C:\Users\<user>\Desktop\WinDev\ConEmuPack.230724\ConEmu\clink\themes\oh-my-posh.clinkprompt
```

The initialization above refreshes the custom prompt configuration for
the current portable WinDev installation.

This was verified by renaming the WinDev directory and launching ConEmu
again. The Oh My Posh prompt continued to work without manually editing
the Clink configuration.
