# Oh My Posh

# Oh My Posh

Oh My Posh is used to provide the customized shell prompt.

The objective is to keep Oh My Posh completely portable and avoid
requiring a system-wide installation.

The Oh My Posh executable is kept inside the WinDev/ConEmu environment.

## 15.1 Oh My Posh Directory

The relevant directory structure is:

``` text
%ConEmuBaseDir%
│
├── _OhMyPosh
│   ├── _themes
│   │   └── peru.omp.json
│   │
│   └── _Fonts
│
└── clink
    └── oh-my-posh.lua
```

The current theme is:

``` text
peru.omp.json
```

located at:

``` text
%ConEmuBaseDir%\_OhMyPosh\_themes\peru.omp.json
```

# Clink + Oh My Posh Integration

Clink loads Oh My Posh through a Lua startup script.

The file is:

``` text
%ConEmuBaseDir%\clink\oh-my-posh.lua
```

Current contents:

``` lua
local conemu = os.getenv("ConEmuBaseDir")

load(io.popen(
    'oh-my-posh init cmd --config "' ..
    conemu ..
    '/_OhMyPosh/_themes/peru.omp.json"'
):read("*a"))()
```

The script obtains the ConEmu installation directory dynamically using:

``` lua
os.getenv("ConEmuBaseDir")
```

Therefore there is no hard-coded user-specific path inside the Lua
configuration.

The theme is resolved relative to the active ConEmu installation.

# Portable Oh My Posh Font

The PERU theme uses special prompt glyphs which require a compatible
font.

Instead of installing a Nerd Font system-wide, the font files are kept
inside WinDev:

``` text
%ConEmuBaseDir%\_OhMyPosh\_Fonts
```

ConEmu is launched with:

``` text
-FontDir
```

pointing to the portable font directory.

This allows the font to remain part of the portable environment.

No permanent Windows font installation is required.

------------------------------------------------------------------------
