<div align="center">

<img src="icon.svg" alt="Strife Logo" width="128" height="128" />

# Strife

**An experimental, customizable Discord client built with Tauri, Rust, and SolidJS.**

[![Rust](https://img.shields.io/badge/Rust-1.70+-orange.svg?logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Tauri](https://img.shields.io/badge/Tauri-1.5-blue.svg?logo=tauri&logoColor=white)](https://tauri.app/)
[![SolidJS](https://img.shields.io/badge/SolidJS-1.8-2c4f7c.svg?logo=solid&logoColor=white)](https://www.solidjs.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.2-blue.svg?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-4.5-646CFF.svg?logo=vite&logoColor=white)](https://vitejs.dev/)

*An attempt at building an alternative Discord client with fine-grained UI reactivity, custom live WYSIWYG markdown input, deep runtime theming, and multi-tab workflows.*

Originally developed at https://github.com/ErisTeam/strife.

[Post-Mortem](#-project-status--post-mortem) • [Features](#-features) • [Architecture](#-architecture) • [Getting Started](#-getting-started) • [Project Structure](#-project-structure) • [Disclaimer](#-disclaimer) • [Authors](#-authors)

</div>

---

## 🪦 Project Status & Post-Mortem

**This project is archived and no longer maintained.**

We set out to see if rewriting Discord from scratch with **Tauri (Rust)** and **SolidJS** would noticeably beat the official Electron client on speed and resource usage. 

Ultimately, it didn't. In practice, the performance gains were not satisfactory—nowhere near enough to justify the endless cat-and-mouse game of reverse-engineering private Discord APIs, gateway nuances, and breaking protobuf schemas. 

If nothing else, it was a solid crash course in architecture, scoping, and learning firsthand what *not* to do when taking on a project this massive.

---

## ✨ Features We Built

### ✍️ Custom Live WYSIWYG Markdown Editor
To learn how rich text editing works under the hood, we built a custom live markdown input from scratch in `MessageEditor.tsx` and `Messages.tsx`:
- **Real-Time In-Place Formatting**: As you type, markdown syntax (`**bold**`, `*italic*`, `~~strike~~`, `__underline__`, `# headers`, `` `code` ``, `> quotes`, `||spoilers||`) renders visually right inside the input.
- **Delimiter Preservation**: Delimiter tokens (`**`, `*`, `~~`) remain preserved in subtle indicator spans (`mdSuggestion`) so you always have raw markdown control without visual guesswork.
- **Custom Caret Management**: Implements custom cursor tracking (`getCursorPosition`, `setCursorPosition`) to prevent the cursor from jumping to the start or end on DOM re-renders while typing or pasting.
- **Member Mention Autocomplete**: Inline autocomplete popup triggered when typing `@`, filtering against server recipients in real time.
- **Paste Handling**: Native clipboard paste handling supporting both plain text formatting and direct file/blob uploads.
- **Compatibility Testing**: Markdown rendering was checked against the official Discord client using a dedicated test channel covering nesting and edge cases.

### 💬 Chat & Messaging
- **Virtualized Chat Feed**: Uses `@tanstack/solid-virtual` to keep DOM element counts manageable in long chat logs.
- **Full Message Lifecycle**: Send, edit, delete, and reply to messages.
- **Discord Embeds**: Renders author, description, thumbnail, image, video and field data, with special styling for tweet embeds so they read like a post.
- **Media Attachments**: In-app previews for image attachments and integrated file picker dialogs.
- **Chat Details**: Real-time typing indicators (*"X is typing..."*), message timestamps, context menus, and user mentions.

### 📑 Browser-Style Multi-Tab Navigation
- **Simultaneous Tabs**: Open different channels, DMs, or settings panes across separate tabs at the same time.
- **Drag-and-Drop Reordering**: Smooth tab sorting via `@thisbeyond/solid-dnd`.
- **State & Scroll Caching**: Inactive tabs remain alive in the background (`display: none`), preserving conversation scroll positions and typed input when switching back and forth.
- **Session Persistence**: Saves your active tabs and ordering to disk between launches.

### 🎨 Live Theme & Customization Engine
- **Runtime CSS Overrides**: Built-in settings screen (`Settings.tsx`, `CssPropertyInput.tsx`) to modify CSS variables live without rebuilding or restarting.
- **10-Tier Depth System**: Structured depth layers (`--depth0` to `--depth9`) for flexible interface palettes.
- **Visual Tweaks**: Real-time control over border roundness, channel gradients, hover effects, sidebar widths, and chat backgrounds.
- **Frameless Window**: Custom titlebar controls (minimize, maximize, close) with native Windows DWM rounded borders (`DwmExtendFrameIntoClientArea`).

### 🔐 Authentication
- **Remote QR Code Login**: Implements Discord’s remote-auth QR login handshake over WebSocket and renders the QR code for the mobile app.
- **Standard Auth**: Email/password authentication with MFA (TOTP) and hCaptcha integration (`solid-hcaptcha`).

### ⚡ Rust & Gateway Backend
- **Custom Gateway WebSocket**: Built with `tokio` and `tokio-tungstenite`, handling zlib-stream decompression via `flate2`.
- **Protobuf Support**: Uses `prost` to decode Discord's binary protobuf packets (`PreloadedUserSettings.proto`).
- **Voice Gateway Skeleton**: Scaffolding and types for connecting to Discord voice gateways.
- **Native Notifications**: Desktop notifications wired through Windows WinRT and Tauri APIs.

### 🌍 Localization
- Internationalization powered by `@solid-primitives/i18n` with support for English (`en_US`) and Polish (`pl_PL`).

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       SolidJS Frontend                      │
│  - Live WYSIWYG Markdown Editor (custom cursor & parsing)   │
│  - Reactive UI & Stores (@solidjs/store)                   │
│  - Multi-tab System & Drag-and-Drop (@thisbeyond/solid-dnd)│
│  - Virtualized Message List (@tanstack/solid-virtual)       │
│  - Live CSS Variable Theming Engine                         │
└──────────────────────────────▲──────────────────────────────┘
                               │  Tauri IPC (Commands & Events)
┌──────────────────────────────▼──────────────────────────────┐
│                      Tauri / Rust Core                      │
│  - Gateway WebSocket (tokio-tungstenite + flate2)           │
│  - Remote Auth / QR Crypto (rsa, sha2, base64)             │
│  - Protobuf Decoding (prost)                                │
│  - Windows Win32 DWM Frame & WinRT Notifications            │
└─────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technologies |
|---|---|
| **Core Desktop Framework** | [Tauri](https://tauri.app/) (v1.5) |
| **Backend / Systems** | [Rust](https://www.rust-lang.org/) (2021 Edition, Tokio async runtime) |
| **Frontend Framework** | [SolidJS](https://www.solidjs.com/) (v1.8), [Vite](https://vitejs.dev/) |
| **Language** | [TypeScript](https://www.typescriptlang.org/) (v5.2) |
| **Chat & UI Helpers** | Custom WYSIWYG Parser, `@tanstack/solid-virtual`, `@thisbeyond/solid-dnd` |
| **Networking & Crypto** | `tokio-tungstenite`, `prost`, `rsa`, `sha2`, `reqwest` |
| **Icons & Styling** | `lucide-solid`, CSS Modules, PostCSS Nested |

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** (v18 or newer)
- **pnpm** (`npm install -g pnpm`)
- **Rust toolchain** (1.70+): Install via [rustup.rs](https://rustup.rs/)
- **C++ Build Tools**: Visual Studio C++ Build Tools (Windows)
- **Protobuf Compiler**: Included under `protoc-23.2/` for Windows, or installed in your `PATH`

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Gami13/strife.git
   cd strife
   ```

2. **Install frontend dependencies:**
   ```bash
   pnpm install
   ```

3. **Run in development mode:**
   ```bash
   # Starts Vite and launches the Tauri desktop shell
   pnpm tdev
   ```

   *Alternatively, run just the web view in your browser:*
   ```bash
   pnpm dev
   ```

4. **Build:**
   ```bash
   pnpm build
   pnpm tauri build
   ```

---

## 📁 Project Structure

```
strife/
├── src/                          # SolidJS Frontend
│   ├── API/                      # API modules (Messages, Guilds, Channels, Settings, Style, Tabs)
│   ├── Components/
│   │   ├── Chat/                 # Chat UI, custom WYSIWYG MessageEditor, embeds, attachments
│   │   ├── Tabs/                 # Browser-style multi-tab container & tab sorting
│   │   ├── Settings/             # Live CSS property customization UI
│   │   ├── Guild/                # Guild sidebar, guild icons, server folders
│   │   ├── ChannelList/          # Category trees & channel items
│   │   ├── Friends/              # Friends dashboard & DM relationships
│   │   ├── Login/                # QR code auth, MFA, login forms
│   │   ├── ControlPanel/         # Current user status & audio panel
│   │   └── WindowDecoration/     # Custom frameless titlebar
│   ├── Locales/                  # Translation files (en_US, pl_PL)
│   ├── Routes/                   # Page routes (/app, /login, /dev)
│   ├── types/                    # Shared TypeScript interfaces
│   ├── AppState.tsx              # Reactive state context
│   └── index.tsx                 # Frontend root entry point
├── src-tauri/                    # Rust backend
│   ├── src/
│   │   ├── commands/             # Tauri IPC command handlers
│   │   ├── discord/              # Discord packet definitions & gateway protocols
│   │   ├── events/               # Event dispatchers to frontend webview
│   │   ├── modules/              # Gateway client, remote QR auth, voice gateway
│   │   ├── proto/                # Protocol Buffer schemas (.proto)
│   │   └── main.rs               # Tauri app setup & DWM window border setup
│   ├── Cargo.toml                # Rust dependencies & metadata
│   └── tauri.conf.json           # Tauri window and build config
├── protoc-23.2/                  # Bundled protoc binary for Windows
└── package.json                  # Frontend dependencies and scripts
```

---

## ⚠️ Disclaimer

Strife was developed purely as an educational project and prototype. Using third-party clients to connect to Discord's official APIs is against the [Discord Terms of Service](https://discord.com/terms). Use at your own discretion.

---

## 👥 Authors

- **Pietruszka123**
- **Frytak**
- **Gami** ([@Gami13](https://github.com/Gami13))
