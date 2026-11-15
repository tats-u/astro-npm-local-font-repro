# Minimal reproduction of Astro's local npm font loading failure

This project reproduces a `CannotFetchFontFile` build failure when Astro tries to read a `file:` URL returned by `fontProviders.npm({ remote: false })`.

## Environment

- Astro 7.4.12
- unifont 1.0.3 (an Astro dependency pinned in `pnpm-lock.yaml`)
- `@fontsource-variable/inter` 5.3.0
- pnpm 12.21.0
- Reproduced with Node.js 26.13.1 on Windows x64
- No adapters or integrations

Use Node.js 22.23.4 or later.

## Steps to reproduce

Run these commands from the project root.

```sh
pnpm install --frozen-lockfile
pnpm why unifont
node --run build
```

## Actual result

The build fails with exit code 1 even though the installed WOFF2 file exists.

```text
[assets] Copying fonts (7 files)...
[CannotFetchFontFile] An error occurred while fetching the font file from file:///<project>/node_modules/@fontsource-variable/inter/files/inter-cyrillic-ext-wght-normal.woff2.
  Caused by:
  fetch failed
```

`<project>` replaces the absolute project path in the log. The configured family name, `Inter Variable`, matches the name in the package's CSS. No font-family-not-found warning is emitted.

## Expected result

Astro reads the locally installed font files, completes the build successfully, and includes seven WOFF2 files in the output.

## Suspected cause and verified workaround

With `remote: false`, unifont's npm provider returns local font paths as `file:` URLs. Astro 7.4.12's `CachedFontFetcher` reads a file only when `isAbsolute(url)` is true and otherwise passes the URL to `fetch(url)`. This appears to prevent `file:` URLs from being read as local files.

Converting the `file:` URLs in the provider's returned `src` entries to absolute filesystem paths with `fileURLToPath()` allows the same configuration to build successfully.

Reproduction on Linux and macOS, and the first affected version, have not been verified.
