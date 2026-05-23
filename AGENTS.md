# Dependencies Agent Notes

## Start Here

- Read this file first, then `RULES.md`, then the top policy section and latest entries in `MEMORY.md`.
- Keep `README.md` as the human-facing project overview; do not turn it into an agent log.
- Use `MEMORY.md` for durable collaboration notes, decisions, deployment facts, and gotchas.
- Do not write secrets, tokens, private keys, or `.env` values into repository files or logs.

## Repository

- Owner: `DylanChiang-Dev`
- Repository: `rustdesk`
- Origin: `https://github.com/DylanChiang-Dev/rustdesk.git`
- Local path: `/Users/dc/Documents/rustdesk`
- Main branch: `master`
- Private: `False`

## Tech Stack

- Rust/Cargo project

## Common Commands

- Check: `cargo check`

## Work Rules

- Before editing, inspect the relevant source files and existing style.
- Keep changes small and reviewable; avoid unrelated refactors.
- Run the fastest relevant check after changes.
- Commit completed work with a clear Chinese commit message unless the user asks otherwise.
- Push only after the local verification for the change has passed.

## Documentation Map

- `README.md`: project overview for humans.
- `AGENTS.md`: current agent entrypoint and operating notes.
- `RULES.md`: stable repository-specific rules.
- `MEMORY.md`: progressive collaboration memory and historical notes.

## RustDesk Project Layout

### Directory Structure
- `src/` Rust app
- `src/server/` audio / clipboard / input / video / network
- `src/platform/` platform-specific code
- `src/ui/` legacy Sciter UI (deprecated)
- `flutter/` current UI
- `libs/hbb_common/` config / proto / shared utils
- `libs/scrap/` screen capture
- `libs/enigo/` input control
- `libs/clipboard/` clipboard
- `libs/hbb_common/src/config.rs` all options

### Key Components
- **Remote Desktop Protocol**: Custom protocol implemented in `src/rendezvous_mediator.rs` for communicating with rustdesk-server
- **Screen Capture**: Platform-specific screen capture in `libs/scrap/`
- **Input Handling**: Cross-platform input simulation in `libs/enigo/`
- **Audio/Video Services**: Real-time audio/video streaming in `src/server/`
- **File Transfer**: Secure file transfer implementation in `libs/hbb_common/`

### UI Architecture
- **Legacy UI**: Sciter-based (deprecated) - files in `src/ui/`
- **Modern UI**: Flutter-based - files in `flutter/`
  - Desktop: `flutter/lib/desktop/`
  - Mobile: `flutter/lib/mobile/`
  - Shared: `flutter/lib/common/` and `flutter/lib/models/`

## Rust Rules

- Avoid `unwrap()` / `expect()` in production code.
- Exceptions: tests; lock acquisition where failure means poisoning, not normal control flow.
- Otherwise prefer `Result` + `?` or explicit handling.
- Do not ignore errors silently.
- Avoid unnecessary `.clone()`.
- Prefer borrowing when practical.
- Do not add dependencies unless needed.
- Keep code simple and idiomatic.

## Tokio Rules

- Assume a Tokio runtime already exists.
- Never create nested runtimes.
- Never call `Runtime::block_on()` inside Tokio / async code.
- Do not hide runtime creation inside helpers or libraries.
- Do not hold locks across `.await`.
- Prefer `.await`, `tokio::spawn`, channels.
- Use `spawn_blocking` or dedicated threads for blocking work.
- Do not use `std::thread::sleep()` in async code.

## Editing Hygiene

- Change only what is required.
- Prefer the smallest valid diff.
- Do not refactor unrelated code.
- Do not make formatting-only changes.
- Keep naming/style consistent with nearby code.

## Migrated From `CLAUDE.md`

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build Commands
- `cargo run` - Build and run the desktop application (requires libsciter library)
- `python3 build.py --flutter` - Build Flutter version (desktop)
- `python3 build.py --flutter --release` - Build Flutter version in release mode
- `python3 build.py --hwcodec` - Build with hardware codec support
- `python3 build.py --vram` - Build with VRAM feature (Windows only)
- `cargo build --release` - Build Rust binary in release mode
- `cargo build --features hwcodec` - Build with specific features

### Flutter Mobile Commands
- `cd flutter && flutter build android` - Build Android APK
- `cd flutter && flutter build ios` - Build iOS app
- `cd flutter && flutter run` - Run Flutter app in development mode
- `cd flutter && flutter test` - Run Flutter tests

### Testing
- `cargo test` - Run Rust tests
- `cd flutter && flutter test` - Run Flutter tests

### Platform-Specific Build Scripts
- `flutter/build_android.sh` - Android build script
- `flutter/build_ios.sh` - iOS build script
- `flutter/build_fdroid.sh` - F-Droid build script

### Config
All configurations or options are under `libs/hbb_common/src/config.rs` file, 4 types:
- Settings
- Local
- Display
- Built-in
```

## Migrated From `Agent.md`

```markdown
# RustDesk 專案介紹

RustDesk 是一個使用 Rust 語言編寫的開源遠端桌面控制軟體。它主打開箱即用，無需繁雜配置，並且強調使用者擁有對自身資料的完全控制權，確保高度的安全性。

## 核心特色
- **開箱即用**：不需要特別配置即可進行遠端連線。
- **資料自主與安全**：可以選擇使用官方提供的中繼伺服器（Rendezvous/Relay Server），或是自行架設伺服器以實現完全的私人網路控制。
- **跨平台支援**：提供 Windows、macOS、Linux 甚至是行動裝置版本的支援。
- **現代化介面**：目前主要採用 Flutter 進行桌面端與行動端的圖形介面（GUI）開發（舊版的 Sciter 介面已廢棄）。

## 專案目錄結構解析
RustDesk 的架構包含了底層 Rust 實作的影像處理、網路通訊與系統控制，以及上層的圖形介面：

- **`libs/hbb_common`**：包含影像編碼、設定配置、TCP/UDP 封裝、Protobuf 定義、檔案系統操作（用於檔案傳輸）及其他核心工具函數。
- **`libs/scrap`**：負責螢幕畫面擷取（Screen capture）。
- **`libs/enigo`**：針對不同平台實作的硬體級鍵盤與滑鼠控制。
- **`libs/clipboard`**：跨平台（Windows, Linux, macOS）的剪貼簿同步實作，支援文字與檔案的檔案複製貼上。
- **`src/server`**：提供音訊、剪貼簿、輸入控制與影像等服務，負責處理被控端的網路與事件。
- **`src/client.rs`**：負責啟動與遠端的點對點連接（Peer connection）。
- **`src/rendezvous_mediator.rs`**：負責與官方或自建的 Rendezvous 伺服器通訊，處理 NAT 穿透（TCP Hole punching）或退回使用 Relay 中繼網路。
- **`flutter/`**：包含目前主要的 Flutter 應用程式碼，涵蓋桌面端與行動端的 UI 實作。

總結來說，RustDesk 是一個強大且安全的遠端桌面替代方案，結合了 Rust 在底層的高效能與安全性，以及 Flutter 的跨平台靈活優勢。
```
