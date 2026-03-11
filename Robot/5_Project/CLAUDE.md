# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"로봇은 실수하지 않아" (Robot Doesn't Make Mistakes) — a top-down casual/stealth/strategy game built with C++ and Win32 API. Genre includes multi-ending and survival mechanics.

- **Language:** C++
- **Platform:** Windows (Win32 API, GDI rendering)
- **IDE:** Visual Studio 2022
- **Solution file:** `WindowEngine.sln`

## Build

Open `WindowEngine.sln` in Visual Studio 2022 and build. Alternatively, use MSBuild from a Developer Command Prompt:

```
msbuild WindowEngine.sln /p:Configuration=Release /p:Platform=x64
```

Supported configurations: Debug/Release for Win32 and x64. Platform toolset is v143 (VS 2022).

There are no automated tests, no linting tools, and no package manager. FMOD is vendored in `WindowEngine/fmod/`.

## Architecture

### Manager-Singleton Pattern

The engine uses a centralized manager architecture. `GameProcess` owns and initializes all managers, which are accessed via getters:

- **SceneManager** — manages scene lifecycle (Main, Option, Tutorial, Stage01, Cartoon, Ending, Credit)
- **ResourceManager** — loads and caches `Texture` objects (bitmaps) by string key
- **PathManager** — resolves filesystem paths to resource directories
- **SoundManager** — wraps FMOD Studio API; organizes audio by `SOUND_TYPE` with channel groups
- **KeyManager** — tracks per-frame input states (TAP, HOLD, AWAY, NONE) for keyboard and mouse
- **CollisionManager** — AABB collision with Enter/Stay/Exit callbacks via `BoxCollider` components
- **TimeManager** — delta time via Windows performance counters
- **CameraManager** — viewport/camera control
- **UIManager** — UI layer management
- **EventManager** — deferred object creation/deletion queue
- **BossManager** — boss entity state management

### Scene and Object System

Each `Scene` holds objects in typed vectors indexed by `OBJECT_TYPE` (DEFAULT, BACKGROUND, FOOD, INTERACTABLE_OBJECT, PLAYER, BOSS, SYSTEM, UI). Objects have components like `BoxCollider` and `Animator`.

### Rendering

Double-buffered GDI rendering. `GameProcess` manages a back-buffer `Texture`, draws all objects, then flips via `BufferFliping()`. Transparency uses `TransBlt` from Msimg32.lib. GDI resources (brushes, pens, fonts) are managed via `SelectGDI`.

### Animation

`Animator` manages per-object animation state. `Animation` handles frame-based sprite atlas playback with per-frame duration and repeat mode.

### Precompiled Header

`pch.h` is the precompiled header. It pulls in Windows headers, STL containers, math utilities (`Vector2`, `Math.h`), FMOD, and `GameProcess.h`. All source files include it.

## External Dependencies

- **FMOD Studio API** — vendored in `WindowEngine/fmod/` (headers in `inc/`, libs in `lib/`). Linked via `#pragma comment(lib)` in `pch.h`. x86 and x64 paths differ.
- **Msimg32.lib** — Windows library for `TransBlt`, linked in `pch.h`.

## Coding Conventions (from ReadMe.h)

1. Enums and `#define` constants: `UPPER_SNAKE_CASE`
2. Local variables/instances: `camelCase`
3. Private member variables: `m_camelCase`
4. Functions and class names: `PascalCase`
5. Function parameters: `_camelCase` (prefixed with underscore)
6. All member variables are `private` with public accessors
7. Managers use the Singleton pattern
8. Virtual functions must use `override`
9. Use `assert` for validation; prefer `const` correctness
10. Code comments are written in Korean

## Key Files

- `WindowEngine.sln` — Visual Studio solution
- `WindowEngine/GameProcess.h/.cpp` — central game loop and manager orchestration
- `WindowEngine/pch.h` — precompiled header with all shared includes
- `WindowEngine/define.h` — enums (`OBJECT_TYPE`, `SCENE_TYPE`, `SOUND_TYPE`, etc.)
- `WindowEngine/ReadMe.h` — coding conventions reference
- `Resource/` — game assets (Font, Sound, Texture subdirectories)
