# Workspace Agent Instructions & Knowledge

## Workspace Knowledge & Implementation History

### Fix: PostCSS & Tailwind Config Path Resolution in Next.js Turbopack (`border-border` build error)
- **Problem & Root Cause**:
  - When building or starting the app (`next build renderer` or `electron-next dev`), Next.js Turbopack runs PostCSS isolated in the context of the `./renderer` directory.
  - `tailwind.config.js` is located at the workspace root (`/tailwind.config.js`), not inside `/renderer`.
  - When PostCSS executed `tailwindcss: {}` with empty options, Tailwind attempted to discover the configuration file starting from the renderer subfolder. Failing to find a config file there, Tailwind loaded a blank default config without custom colors (`border`, `background`, `ring`, etc.) or plugins (`daisyui`).
  - This caused `@apply border-border` in `renderer/styles/globals.css` to fail with `CssSyntaxError: The 'border-border' class does not exist`.
  - When Next.js runs PostCSS from `./renderer`, Tailwind scans content files relative to the active directory. The default `./renderer/pages/**` globs matched 0 files from `./renderer`, which stripped out all component utility classes (causing dashboard unstyling / mismatch).
- **Solution & Modified Files**:
  - Modified [postcss.config.js](file:///c:/Users/jassi/OneDrive/Desktop/upscayl/postcss.config.js): Explicitly passed the absolute path to `tailwind.config.js` via `path.join(__dirname, "tailwind.config.js")` into the `tailwindcss` plugin config.
  - Modified [tailwind.config.js](file:///c:/Users/jassi/OneDrive/Desktop/upscayl/tailwind.config.js): Added `./pages/**/*.{js,ts,jsx,tsx}` and `./components/**/*.{js,ts,jsx,tsx}` alongside the `./renderer/...` paths in `content` so Tailwind correctly discovers all 65 component and page files from both root and renderer execution contexts.
  - Added [.vscode/settings.json](file:///c:/Users/jassi/OneDrive/Desktop/upscayl/.vscode/settings.json): Suppressed `css.lint.unknownAtRules` in VS Code to avoid false cosmetic warnings on `@apply` and `@tailwind`.
- **Architectural Gotchas / Invariants**:
  - In multi-directory setups where Next.js runs from a subdirectory like `renderer/`, PostCSS plugins must be explicitly supplied with absolute paths to root configurations (`tailwind.config.js`), and Tailwind `content` globs must cover paths evaluated from both root and `./renderer` working directories.


