# 📚 Book-Mine

A simple desktop book-reading application built with **[Wails](https://wails.io/)**, combining a Go backend with a vanilla HTML/CSS/JavaScript frontend. Book-Mine lets you view a book excerpt in a styled reader UI and upload/render PDF files directly in the app using PDF.js.

> **Status:** This project is in an early, template-derived stage. The Go/JS bridge and window scaffolding are functional, but most reader features (navigation, book library, ratings, user profiles) are still UI placeholders. See [Current State & Roadmap](#current-state--roadmap) below for specifics.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Running in Development](#running-in-development)
- [Building for Production](#building-for-production)
- [Usage](#usage)
- [Go ↔ JavaScript Bindings](#go--javascript-bindings)
- [Current State & Roadmap](#current-state--roadmap)
- [Acknowledgments](#acknowledgments)
- [License](#license)

---

## Overview

Book-Mine is a cross-platform desktop app (Windows & macOS) built on top of Wails v2, which packages a Go application together with a native OS webview. The frontend is plain HTML/CSS/JS (no framework), styled to resemble a digital book reader, and includes an in-browser PDF viewer powered by [PDF.js](https://mozilla.github.io/pdf.js/).

The actual application code lives inside the [`sample/`](./sample) directory of this repository.

## Tech Stack

| Layer | Technology |
|---|---|
| Desktop runtime | [Wails v2](https://wails.io/) (`v2.2.0`) |
| Backend | Go `1.18` |
| Frontend | HTML5, CSS3, vanilla JavaScript |
| DOM/AJAX helper | jQuery `2.2.4` (via CDN) |
| PDF rendering | PDF.js (`pdf.js` / `pdf.worker.js`, bundled locally) |
| Packaging | Wails CLI build pipeline (Windows `.exe` / installer, macOS `.app`, Intel/ARM/Universal) |

## Project Structure

```
book-mine/
├── .idea/                     # JetBrains IDE project settings (not required to build)
└── sample/                    # The actual Wails application
    ├── app.go                 # Go "App" struct — lifecycle hooks + methods exposed to the frontend
    ├── main.go                # Wails app entry point / window configuration
    ├── go.mod / go.sum         # Go module dependencies
    ├── wails.json              # Wails project configuration
    ├── README.md               # Original Wails template README
    ├── scripts/                 # Convenience shell scripts for installing & building
    │   ├── install-wails-cli.sh
    │   ├── build.sh
    │   ├── build-windows.sh
    │   ├── build-macos.sh
    │   ├── build-macos-arm.sh
    │   └── build-macos-intel.sh
    ├── build/                   # Platform build assets (icons, Info.plist, Windows manifest/installer)
    │   ├── darwin/
    │   ├── windows/
    │   └── bin/                 # Compiled output (e.g. sample.exe)
    └── frontend/
        ├── wailsjs/              # Auto-generated Go↔JS bindings (go/ and runtime/)
        └── src/
            ├── index.html        # Main UI (nav bar, reader section, embedded PDF viewer script)
            ├── main.css          # Styling for the reader UI
            ├── main.js           # Frontend logic that calls the bound Go methods
            └── assets/
                ├── fonts/
                ├── images/       # Book cover, icons, logo, overlay art
                └── js/           # pdf.js and pdf.worker.js
```

## Prerequisites

Before you begin, make sure you have:

- **[Go](https://go.dev/dl/)** `1.18` or later
- **[Node.js](https://nodejs.org/)** (for any future frontend tooling; not currently required since the frontend uses no build step)
- **[Wails CLI v2](https://wails.io/docs/gettingstarted/installation)**
- Platform build tools:
  - **Windows:** WebView2 runtime, a C compiler (e.g. via MSYS2/TDM-GCC) if building from Windows
  - **macOS:** Xcode Command Line Tools

You can install the Wails CLI using the provided script:

```bash
cd sample/scripts
./install-wails-cli.sh
```

This runs `go install github.com/wailsapp/wails/v2/cmd/wails@latest` under the hood.

Verify your environment is set up correctly:

```bash
wails doctor
```

## Getting Started

1. Clone the repository:

   ```bash
   git clone https://github.com/joshuaomonemu/book-mine.git
   cd book-mine/sample
   ```

2. Install Go module dependencies:

   ```bash
   go mod tidy
   ```

## Running in Development

From the `sample/` directory, start the app in live-development mode:

```bash
wails dev
```

This compiles the Go backend, opens the application window, and watches for changes. Because the frontend has no separate dev server/build step configured (`frontend:install` and `frontend:build` are empty in `wails.json`), edits to files under `frontend/src` are picked up directly by Wails' embedded asset watcher.

## Building for Production

A generic build can be produced with:

```bash
cd sample
wails build --clean
```

Convenience scripts are also provided under `sample/scripts/` (run them **from inside** the `scripts/` directory, since each `cd ..`s into `sample/` before invoking `wails build`):

| Script | Target |
|---|---|
| `build.sh` | Default platform build |
| `build-windows.sh` | `windows/amd64` |
| `build-macos.sh` | `darwin/universal` |
| `build-macos-intel.sh` | `darwin` (Intel) |
| `build-macos-arm.sh` | `darwin/arm64` |

Example:

```bash
cd sample/scripts
./build-windows.sh
```

For a smaller binary, you can additionally compress with [UPX](https://upx.github.io/):

```bash
wails build -upx -upxflags="--best --lzma"
```

Build artifacts are placed in `sample/build/bin/`. Platform packaging assets (app icons, Windows installer config, macOS `Info.plist`) live under `sample/build/windows/` and `sample/build/darwin/`.

## Usage

When the app launches, you'll see:

- A top navigation bar (`HOME`, `BOOKS`, `SERVICES`, profile name)
- A reader section displaying a book cover image and a sample text excerpt
- An embedded **PDF upload & viewer**: selecting a `.pdf` file renders it page-by-page onto an HTML canvas using PDF.js, with an option to download the currently rendered page as a PNG image

> Note: several UI elements (navigation links, "rate this book", "who's reading it", "your books" sections, page prev/next controls) are present in the markup/CSS but are not yet wired up to any interactivity or Go backend logic — see the roadmap below.

## Go ↔ JavaScript Bindings

Wails automatically generates JS bindings (under `frontend/wailsjs/go`) for any Go methods exposed on the `App` struct in `app.go`. Currently exposed methods:

| Go method | Description |
|---|---|
| `Greet(name string) string` | Returns a greeting string for the given name — a Wails template example. |
| `Todo() string` | Placeholder method returning a static string; intended as a stub for future book-related functionality. |

To expose new backend functionality to the frontend, add a method to the `App` struct in `sample/app.go` and call it from JavaScript via `window.go.main.App.<MethodName>(...)`, following the pattern already used in `frontend/src/main.js`.

## Current State & Roadmap

This repository currently reflects an early, template-based snapshot of the project (derived from the [`wails-pure-js-template`](https://github.com/KiddoV/wails-pure-js-template)) rather than a fully-featured reading app. Based on the current code:

**Implemented**
- Wails application shell (Windows & macOS window configuration, app icon, title `Book-Mine`)
- Static reader UI layout and styling
- Client-side PDF loading, page rendering, and page-image download via PDF.js
- Cross-platform build scripts

**Not yet implemented**
- Book library / multiple book management
- Persistent storage (e.g. reading progress, uploaded books, ratings)
- Working navigation between `HOME` / `BOOKS` / `SERVICES`
- User profile/account functionality
- Any Go-backed business logic beyond the `Greet`/`Todo` template stubs

Contributions that flesh out these areas are welcome — see [Contributing](#contributing).

## Contributing

Contributions, issues, and feature requests are welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes
4. Push to your fork and open a Pull Request

## Acknowledgments

- Built with [Wails](https://wails.io/)
- Frontend scaffold based on [wails-pure-js-template](https://github.com/KiddoV/wails-pure-js-template) by [KiddoV](https://github.com/KiddoV)
- PDF rendering powered by [PDF.js](https://mozilla.github.io/pdf.js/)

## License

No license file is currently included in this repository. If you intend for others to use, modify, or distribute this project, consider adding a `LICENSE` file (e.g. [MIT](https://choosealicense.com/licenses/mit/)) to clarify usage terms.
