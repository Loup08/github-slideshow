---
name: unreal-packaging
description: Package and ship Unreal Engine 5 projects — build configurations (Development, Shipping), Project Launcher and RunUAT BuildCookRun, cooking and maps to cook, pak/IoStore and chunking, Pixel Streaming and platform SDKs, size and startup optimization, and crash reporting. Use when building distributable UE5 games or setting up CI.
---

# Unreal Packaging

## Configurations

- **DebugGame** — debug game code, optimized engine.
- **Development** — default; logs, console, stats.
- **Shipping** — no console/logging/stats; what players get.
- **Test** — Shipping + some profiling.

Project Settings → Packaging: Build Configuration, *Full Rebuild* for release, *Use Pak File* / *Use Io Store* (on), *Share Material Shader Code*, *Exclude editor content*, *List of maps to include* (otherwise only referenced maps + startup map are cooked).

## Editor

Platforms → Windows → Package Project (or *Project Launcher* for custom profiles: cook by the book, iterative cook, DLC/patch).

## Command line (CI)

```bash
"<UE>/Engine/Build/BatchFiles/RunUAT.bat" BuildCookRun ^
  -project="C:/Proj/MyGame/MyGame.uproject" ^
  -platform=Win64 -clientconfig=Shipping ^
  -build -cook -stage -pak -iostore -package -archive ^
  -archivedirectory="C:/Builds/MyGame" ^
  -nop4 -utf8output -unattended -prereqs
```

Linux/macOS: `RunUAT.sh`. Add `-compressed` for smaller paks, `-distribution` for store builds on some platforms, `-map=Map1+Map2` to restrict maps, `-CrashReporter` to include crash reporter. Code projects need the matching Visual Studio/Xcode toolchain; Blueprint-only projects use the installed engine binaries.

Horde / BuildGraph for larger teams; Jenkins/TeamCity/GitHub self-hosted runners commonly run RunUAT.

## Asset management & chunks

- **Primary Asset Labels** / Asset Manager rules define what gets cooked and into which chunk (pakchunkN).
- Chunking supports DLC, platform install-on-demand, Steam depots splitting.
- Use the *Size Map* and *Reference Viewer* on maps to find what drags in large assets; soft references to break hard chains.

## Size & startup

- Texture LOD groups and max sizes per platform (DefaultDeviceProfiles.ini).
- Remove unused plugins (each can add content and modules).
- Startup map light: show a lightweight menu map, async load gameplay levels (`Open Level` → loading screen via *MoviePlayer* or `ULoadingScreen` patterns).
- **PSO caching**: enable PSO precaching (UE 5.2+) / bundled PSO cache to eliminate first-play shader hitches on DX12/Vulkan.

## Platform notes

- Windows: ship `UEPrereqSetup_x64.exe` (`-prereqs`) or rely on Steam redistributables; DX12 default, keep DX11 fallback option.
- Steam: enable OnlineSubsystemSteam plugin, `steam_appid.txt` only for dev; see `steam-publish`.
- Mac: notarize .app; Apple Silicon native builds.
- Android: Android Studio + NDK versions matching the engine's requirements (Turnkey `RunUAT Turnkey -command=VerifySdk`), ASTC textures, AAB.
- Consoles: require licensed platform SDKs and separate engine platform extensions.

## Crash reporting

Include CrashReportClient and route reports to a backend (Sentry, BugSplat, Backtrace) via `DefaultEngine.ini` `[CrashReportClient] DataRouterUrl=...`. Keep **.pdb/debug symbols** for every shipped build (archive them with the build number).

## Pre-ship checklist

- [ ] Shipping build tested on clean machine (no editor installed)
- [ ] All maps included; no missing-asset warnings in cook log
- [ ] Version number set (Project Settings → Description → Project Version)
- [ ] Symbols archived
- [ ] Startup/shader hitch pass done
- [ ] Scalability defaults sane on minimum spec
