# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TMProject is a decompiled C++ client for the MMORPG "With Your Destiny" (WYD). It is a faithful decompilation — code follows the original game's conventions, not modern best practices. The client is fully functional and playable.

## Build

Windows-only. Requires Visual Studio 2019+ with **Desktop C++ development** and **C++ ATL** workloads.

```
msbuild /m /p:Platform=x86 /p:Configuration=Release TM.sln
```

Output: `Release/TMProject.exe` (or `Debug/TMProject.exe`).

x64 builds are possible but require x64 DirectX libs and fixing arch-specific compilation issues.

CI runs on PRs via `.github/workflows/msbuild.yml` (Debug + Release x86).

No test framework exists.

## Architecture

All source lives in `Projects/TMProject/`. Entry point: `WinMain` in `NewApp.cpp`.

### Core Systems

- **NewApp** (`NewApp.cpp/h`) — Application singleton (`g_pApp`). Owns rendering, sound, networking, input, and the object manager.
- **ObjectManager** (`ObjectManager.cpp/h`) — Game state machine with states: Login → CreateID → SelectServer → SelectChar → CreateChar → Field (gameplay). Manages all game objects.
- **EventTranslator** (`EventTranslator.cpp/h`) — Input handling: keyboard (`m_bKey` array), mouse (DirectInput 8), IME text input.
- **RenderDevice** (`RenderDevice.cpp/h`) — DirectX 9 rendering abstraction.
- **CPSock** (`CPSock.cpp/h`) — TCP socket layer with 131KB send/recv buffers, custom packet protocol (header: size, keyword, checksum, type, ID, tick), AES encryption.

### Subsystem Map

| Subsystem | Key files | Notes |
|-----------|-----------|-------|
| Scenes | `TMScene`, `TMFieldScene`, `TMLoginScene`, `TMSelCharScene`, etc. | Base class + per-state scenes |
| Game objects | `TMObject`, `TMHuman`, `TMItem`, `TMShip`, `TreeNode` | Scene graph with parent-child hierarchy |
| Effects | 14+ effect classes (`TMEffectBillBoard`, `TMEffectParticle`, `TMEffectMesh`, etc.) | Visual FX system |
| Skills | 25+ skill files (`TMSkillBash`, `TMSkillHeal`, `TMSkillMeteorStorm`, etc.) | Individual skill implementations |
| UI | `SControl` base, `SPanel`, `SButton`, `SGrid`, `SText`, `SMessageBox`, `SProgressBar` | Custom widget system |
| Rendering | `D3DDevice`, `TextureManager`, `MeshManager`, font classes | DirectX 9 pipeline |
| Data | `BaseDef.h` (constants), `Quest`, `Mission`, `MrItemMix` | Game constants and data systems |

### Key Dependencies

All bundled in `Dependencies/Directx/` — no package manager:
- DirectX 9 (D3D9, D3DX9)
- DirectInput 8, DirectSound, DirectShow
- Windows API, Winsock, WinInet

### Globals

Singleton pattern via globals: `g_pApp` (app), `g_hInstance`, `g_szServerName`. Constants in `BaseDef.h` (`MAX_CARGO=128`, `TM_CONNECTION_PORT=8281`, etc.).

## Code Conventions

This is decompiled code — follow existing patterns, not modern C++ style:

- **Hungarian notation**: `pPointer`, `nInt`, `bBoolean`, `szString`, `dwDword`
- **PascalCase** for class names (prefixed `TM`, `S`, `CP`)
- **Precompiled header**: `pch.h` (included via `framework.h`)
- **C++17** standard (`/std:c++17`)
- When in doubt about convention, find an existing example in the codebase
