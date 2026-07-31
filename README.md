# K2 SE Browser Slicer

A local-first browser slicer UI for the Creality K2 SE. The app does STL upload first, 3MF preview when browser parsing succeeds, interactive plate editing, K2 SE build-boundary checks, PLA settings, real backend slicing through PrusaSlicer or SuperSlicer, layer-preview parsing, estimates, and downloadable G-code.

This project does not fake slicing. The `Slice` button requires a real PrusaSlicer-compatible CLI binary on the machine or server that runs the Node backend.

## K2 SE Profile

- Build volume: `220 x 215 x 245 mm`
- Filament: `1.75 mm`
- Nozzle: `0.4 mm`
- Nozzle max: `300 C`
- Bed max: `100 C`
- G-code flavor: generic Klipper-style single-filament profile
- CFS: not used; generated config is single extruder / single filament

Specs are based on Creality's K2 SE product page. The generated start/end G-code is intentionally generic; for production use, compare the first file against a known-good Creality Print or PrusaSlicer K2 SE profile before long prints.

## Requirements

- Node.js 22.13 or newer
- PrusaSlicer or SuperSlicer installed locally

macOS example:

```bash
brew install --cask prusaslicer
export PRUSASLICER_BIN="/Applications/PrusaSlicer.app/Contents/MacOS/PrusaSlicer"
```

Linux example:

```bash
sudo apt-get update
sudo apt-get install -y prusa-slicer
export PRUSASLICER_BIN="/usr/bin/prusa-slicer"
```

## Run Locally

```bash
corepack enable
pnpm install
pnpm dev
```

Open the Vite URL, normally `http://127.0.0.1:5173`.

The API runs on `http://127.0.0.1:8787`. The browser app proxies `/api/*` to it during development.

## Build And Run Production Locally

```bash
pnpm build
PRUSASLICER_BIN="/path/to/prusa-slicer" pnpm start
```

Then open `http://127.0.0.1:8787`.

## Deploy Online

Use a container host or VM that can install native packages. Static hosts and edge workers are not enough because slicing needs the PrusaSlicer process.

Docker:

```bash
docker build -t k2-se-browser-slicer .
docker run --rm -p 8787:8787 k2-se-browser-slicer
```

Render/Fly/Railway:

- Build command: `corepack enable && pnpm install --frozen-lockfile && pnpm build`
- Start command: `pnpm start`
- Environment: `PRUSASLICER_BIN=/usr/bin/prusa-slicer`
- Use the Dockerfile when the host does not let you install system packages directly.

## What Works

- STL upload and browser preview
- 3MF upload and preview when `three` can parse the file's geometry
- Orbit, pan, zoom
- Move, rotate, scale
- Center, lay-flat, reset, duplicate, delete, auto-arrange
- Model dimensions and K2 SE plate/height validation
- PLA layer height, walls, top/bottom layers, infill, supports, brim/skirt, temperatures, speeds, nozzle, filament diameter, flow
- Backend slicing through a real PrusaSlicer-compatible CLI
- Parsed layer preview from generated G-code
- Time and filament estimates from slicer comments when present, with motion-based fallback
- G-code download only after slicing succeeds

## Notes

- The frontend exports the arranged plate as STL in positive K2 SE bed coordinates before sending it to the backend.
- The backend passes `--dont-arrange` to preserve the browser plate layout.
- If you need an exact Creality start macro, edit `server/prusaProfile.ts` or load a vendor profile in the slicer and mirror its start/end G-code.
