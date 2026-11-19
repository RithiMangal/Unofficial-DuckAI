# DuckAI — Unofficial Duck.ai Desktop Client

> **Unofficial project.** DuckAI is an independent, community-built desktop
> wrapper around [Duck.ai](https://duck.ai). It is **not** affiliated with,
> endorsed by, or sponsored by DuckDuckGo, Inc. "Duck.ai", "DuckDuckGo", and
> related marks are trademarks of their respective owners. This project exists
> solely to provide a lightweight, native desktop experience for users who
> prefer Duck.ai as a standalone application.

[![License: MIT](https://img.shields.io/badge/License-MIT-cyan.svg)](#license)
[![Platform: Linux](https://img.shields.io/badge/Platform-Linux-blue.svg)](#compatibility)
[![Format: AppImage](https://img.shields.io/badge/Format-AppImage-green.svg)](#installation)
[![cargo audit: clean](https://img.shields.io/badge/cargo%20audit-clean-brightgreen.svg)](#security)

---

• Strategic Recommendation: Frames Lubuntu as the primary native development environment for guaranteed stability.
• Technical Breakdown: Bullet points highlighting the streamlined configuration, low resource footprint, and its ability to revive older hardware with zero lag.
• Veteran Insight: Concludes with an engaging nod to the fun of managing multiple Linux environments.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots](#screenshots)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Installation](#installation)
- [Build from Source](#build-from-source)
- [Usage](#usage)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Security & Privacy](#security--privacy)
- [Compatibility](#compatibility)
- [Cargo Audit](#cargo-audit)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## Overview

DuckAI is a **very lightweight**, native Linux desktop application that embeds
[Duck.ai](https://duck.ai) directly inside its own window using an embedded
WebKitGTK webview. No external browser is required to use Duck.ai — the chat
interface lives inside the app, while the application itself stays compact,
fast to launch, and gentle on system resources.

The window is fully custom-drawn (no OS window decorations): a dark, neon
luminous title bar with metallic 3D glowing controls, a centered glowing
title and subtitle, a one-click **Download All Images** action, and a
**Contact** shortcut. The app launches straight into **fullscreen** for an
immersive experience.

DuckAI ships as a single, portable **AppImage** that runs on virtually every
Linux distribution, regardless of desktop environment, boot mode (UEFI/EFI),
or kernel variant.

---

## Features

- **Embedded Duck.ai** — Duck.ai loads inside the app via WebKitGTK; no
  external browser needed for chat/image generation.
- **Custom borderless window** — OS decorations removed; a hand-crafted
  title bar replaces them.
- **3D metallic glowing controls (top-left)**:
  - **Minimize** — `-` symbol, **yellow** neon glow
  - **Maximize/Restore** — `□` box symbol, **green** neon glow
  - **Close** — `●` circle symbol, **red** neon glow
- **High-rich luminous title** — "DuckAI" in glowing red, subtitle
  "Your AI Companion" in glowing green, with multi-layer text-shadow.
- **Download All Images** — one click collects every image on the page and
  saves them to `~/Downloads` as `*.jpeg` with randomized names.
- **Contact** — opens `https://github.com/RithiMangal?tab=repositories` in
  the system default browser (URL exists only in this action).
- **Fullscreen on launch** — immersive by default; `Esc` exits, `F11`
  toggles.
- **Privacy-first cleanup** — all cookies, cache and temporary files created
  during the session are isolated in a temp directory and wiped from the HDD
  on exit, quickly, without affecting system resources.
- **Lightweight** — minimal dependencies, release-optimized (LTO, strip,
  `opt-level = "z"`, `panic = "abort"`).

---

## Tech Stack

| Layer        | Technology                                    |
|--------------|-----------------------------------------------|
| Language     | Rust (Edition 2021)                           |
| UI Toolkit   | GTK 3 (via `gtk` 0.19, `gdk` 0.19)           |
| Web Engine   | WebKitGTK (via `webkit2gtk` 3.0, `v2_22`)    |
| JS Bridge    | `javascriptcore-rs` 2.0                       |
| Packaging    | AppImage (`appimagetool`)                     |
| Security     | `cargo audit` (RustSec advisory database)     |
| Styling      | GTK CSS (neon luminous 3D gradients/shadows)  |

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  ApplicationWindow (decorated = false)              │
│  ┌───────────────────────────────────────────────┐  │
│  │  Titlebar (GTK Box, horizontal)               │  │
│  │  [−][□][●]   <spacer>  DuckAI / subtitle  <spacer>  [Download All][Contact] │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │                                               │  │
│  │   WebKitGTK WebView  →  https://duck.ai       │  │
│  │   (embedded, fullscreen-capable)              │  │
│  │                                               │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

- **Window controls** drive the GTK window (`iconify`, `maximize`/`unmaximize`,
  `close`).
- **Download All** injects a JavaScript snippet into the webview that
  enumerates every `<img>` and posts each source (URL or base64 `data:` URL)
  to a registered `downloadImage` script-message handler. Rust then persists
  each image to `~/Downloads` with a randomized `*.jpeg` name.
- **Contact** spawns `xdg-open` with the GitHub URL.
- **Cleanup** — `XDG_CACHE_HOME` / `XDG_DATA_HOME` are pointed at an isolated
  temp directory at startup; on exit the directory is recursively removed,
  wiping cookies, cache and temporary files from the HDD.

---

## Installation

### Run the AppImage (no installation required)

```bash
chmod +x DuckAI-x86_64.AppImage
./DuckAI-x86_64.AppImage
```

The AppImage is fully self-contained for the application binary; it requires
the system WebKitGTK/GTK3 runtime libraries (present on virtually all
mainstream Linux distributions).

---

## Build from Source

### Prerequisites

- Rust toolchain (stable, 1.70+): `rustup` / `cargo`
- System libraries:
  - GTK 3: `libgtk-3-dev` (Debian/Ubuntu) / `gtk3-devel` (Fedora) / `gtk3` (Arch)
  - GDK 3: `libgdk-3-dev`
  - WebKitGTK 4.1: `libwebkit2gtk-4.1-dev` (or `libwebkit2gtk-4.0-dev`)
  - JavaScriptCoreGTK: `libjavascriptcoregtk-4.1-dev`
  - `pkg-config`
- `appimagetool` (for packaging)

### Clone & Build

```bash
git clone https://github.com/RithiMangal/DuckAI.git
cd DuckAI

# Optional: webkit2gtk-4.0 -> 4.1 pkg-config alias (if only 4.1 is installed)
mkdir -p pkgconfig
ln -sf /usr/lib/x86_64-linux-gnu/pkgconfig/webkit2gtk-4.1.pc pkgconfig/webkit2gtk-4.0.pc

# Build (debug)
PKG_CONFIG_PATH="$PWD/pkgconfig" cargo build

# Security audit (required before release packaging)
cargo audit

# Build (release, optimized)
PKG_CONFIG_PATH="$PWD/pkgconfig" cargo build --release
```

### Package as AppImage

```bash
# Assemble AppDir
rm -rf AppDir
mkdir -p AppDir/usr/bin AppDir/usr/share/icons/hicolor/256x256/apps AppDir/usr/share/applications
cp target/release/duckai AppDir/usr/bin/duckai
cp duckai.png AppDir/duckai.png
cp duckai.png AppDir/usr/share/icons/hicolor/256x256/apps/duckai.png
cp duckai.desktop AppDir/duckai.desktop
cp duckai.desktop AppDir/usr/share/applications/duckai.desktop
cat > AppDir/AppRun <<'EOF'
#!/bin/sh
cd "$(dirname "$0")"
exec "./usr/bin/duckai" "$@"
EOF
chmod +x AppDir/AppRun AppDir/usr/bin/duckai

# Build AppImage
ARCH=x86_64 appimagetool AppDir
```

---

## Usage

1. Launch `DuckAI-x86_64.AppImage` (or the binary directly).
2. The app opens in **fullscreen** with Duck.ai loaded inside.
3. Use the embedded Duck.ai interface for chat and image generation.
4. Click **Download All** to save every image currently on the page to
   `~/Downloads` as `*.jpeg` with randomized names.
5. Click **Contact** to open the maintainer's GitHub repositories page in
   your default browser.
6. Use **Min / Max / Close** (top-left) to manage the window.
7. Press **Esc** to exit fullscreen, **F11** to toggle fullscreen.
8. On close, all session cookies/cache/temp files are wiped from the HDD.

---

## Keyboard Shortcuts

| Key   | Action                |
|-------|-----------------------|
| `Esc` | Exit fullscreen       |
| `F11` | Toggle fullscreen     |

---

## Security & Privacy

- **No telemetry, no analytics, no tracking.** DuckAI does not phone home.
- **Session isolation** — all WebKit data (cookies, cache, temp files) is
  stored in an isolated temp directory and deleted on exit.
- **`cargo audit`** is run before every release build; the current release
  is **clean** (no vulnerabilities or errors).
- The only external network action triggered by the app is loading
  `https://duck.ai` inside the webview and, on explicit user action, opening
  the GitHub URL in the default browser.
- The GitHub URL exists **only** in the Contact action and nowhere else
  (not in the window, not duplicated in the script).

---

## Compatibility

- **All Linux distributions** — AppImage is distro-agnostic.
- **Latest Plasma (KDE)** and any other desktop environment (GNOME, XFCE,
  Cinnamon, MATE, etc.).
- **UEFI or EFI** boot mode — irrelevant; the app runs in userspace.
- **All kernel types** — x86_64 userspace binary.
- Requires the WebKitGTK / GTK3 runtime (available on all mainstream
  distributions).

---

## Cargo Audit

```
cargo audit
# Scanning Cargo.lock for vulnerabilities (71 crate dependencies)
# (no vulnerabilities or errors — clean)
```

The release pipeline enforces `cargo audit` before packaging. Any
vulnerability or error reported by the audit must be resolved before an
AppImage is produced.

---

## Project Structure

```
DuckAI/
├── Cargo.toml              # Manifest + release profile
├── src/
│   └── main.rs             # GTK3 + WebKitGTK application
├── duckai.png              # Application icon (multi-size)
├── duckai.desktop          # Desktop entry
├── pkgconfig/              # webkit2gtk-4.0 -> 4.1 alias
├── AppDir/                 # AppImage staging directory
├── target/release/duckai   # Compiled binary
├── DuckAI-x86_64.AppImage  # Final portable AppImage
├── README.md               # This file
└── RELEASE_NOTES.md        # Release notes
```

---

## Contributing

Contributions, bug reports and feature requests are welcome. Please open an
issue or pull request on GitHub.

When submitting changes that touch dependencies, run `cargo audit` and ensure
it reports no vulnerabilities or errors before opening a PR.

---

## Disclaimer

DuckAI is an **unofficial** third-party desktop client. It is not affiliated
with, endorsed by, or sponsored by DuckDuckGo, Inc. Duck.ai and DuckDuckGo are
trademarks of DuckDuckGo, Inc. All trademarks belong to their respective
owners. This software is provided "as is", without warranty of any kind.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Maintainer: [RithiMangal](https://github.com/RithiMangal)*
*GitHub: https://github.com/RithiMangal?tab=repositories*
