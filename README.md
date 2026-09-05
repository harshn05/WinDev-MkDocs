# WinDev Documentation

Professional MkDocs documentation for WinDev using the built-in ReadTheDocs theme.

## Build

```bat
mkdocs build --clean
```

The generated static website is placed in `site/`.

For local testing of the generated static output:

```bat
python -m http.server 8000 -d site
```

`use_directory_urls: false` is enabled so generated pages use explicit `.html` filenames, which is convenient for portable/offline distribution.
