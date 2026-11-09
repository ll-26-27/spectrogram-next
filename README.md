# Spectrogram (Next.js port)

A hybrid Next.js port of the Chrome Music Lab spectrogram. The WebGL/audio engine
(`src/legacy/3D`, `src/legacy/UI`, `src/legacy/util`, `public/legacy-libs/o3djs-bundle.js`)
is the original code from `../spectrogram`, lightly patched. The page shell
(`src/app`) is React.

## Run

```bash
pnpm install --frozen-lockfile
pnpm dev
```

Then open <http://localhost:3000>.

The pnpm override pins Sass's `immutable` dependency to compatible version 5.1.9 so installation satisfies the Notebook checkout's minimum-release-age policy. Version 5.1.10 was published on November 9, 2026, the day this checkout was added.

## Layout

- `src/app/SpectrogramShell.js` — `'use client'` component. Renders the canvas + UI
  shell as JSX, then in `useEffect` loads jQuery onto `window` and dynamically
  imports `legacy/main.js` to bootstrap the WebGL spectrogram.
- `src/app/layout.js` — Root layout. Loads `o3djs-bundle.js` via `<Script
  strategy="beforeInteractive">` so `window.o3djs` is available before the
  client component mounts.
- `src/app/globals.scss` (+ `_reset.scss`, `_animation.scss`, `_spinner.scss`,
  `_icons.scss`) — Port of `screen.scss`. Icons use CSS `mask-image` against
  the SVGs in `public/icons/` instead of a generated icon font.
- `src/legacy/` — original modules. Patched only as needed:
  - `3D/visualizer.js`: `var AnalyserView = ...`, `o3djs` → `window.o3djs`,
    shader URLs made absolute.
  - `UI/player.js`: `var context = ...`, `empty.mp3` URL made absolute.
- `public/bin/snd`, `public/bin/shaders`, `public/icons`, `public/images` —
  static assets, served from origin root.
- `public/legacy-libs/o3djs-bundle.js` — concatenation of
  `o3djs/{base,math,quaternions,shader}.js`, equivalent to the original
  gulp `bundle-libs` task.

## Toward a full React rewrite

Quarantined seams that should fall next:

1. Replace jQuery click/blur/visibility/dragdrop wiring in `legacy/main.js`
   with React state + event handlers in `SpectrogramShell.js`.
2. Convert `spec3D` (in `legacy/UI/spectrogram.js`) and `Player` (in
   `legacy/UI/player.js`) to a `useSpectrogram()` hook owning the
   `AnalyserView` + `Player` lifecycle.
3. The WebGL engine itself (`3D/visualizer.js`, `3D/matrix4x4.js`,
   `3D/cameracontroller.js`, `o3djs/*`) is framework-agnostic — keep as-is.
4. Once side effects are cleanupable, re-enable `reactStrictMode: true` in
   `next.config.mjs` and remove the `window.__spectrogramBooted` guard.
