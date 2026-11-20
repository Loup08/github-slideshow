---
name: roblox-studio-workflow
description: Professional Roblox development workflow — Rojo for file-based projects and Git, Wally packages, Selene/StyLua/luau-lsp linting and typing, Rokit/Aftman toolchain, Team Create vs external editors, testing with TestEZ/Jest-lua and Lune, places and universes for staging, publishing and versioning, and Open Cloud CI deploys. Use when setting up or improving a Roblox project's tooling.
---

# Roblox Studio Workflow

## Two modes

- **Studio-only + Team Create**: simplest; scripts live in the place file; use Studio's Script Drafts/collaborative editing and version history. Good for small teams and builders.
- **File-based (Rojo)**: code in Git, edited in VS Code, synced live into Studio. Recommended for programmers, code review, CI.

## Toolchain (Rokit)

```toml
# rokit.toml
[tools]
rojo = "rojo-rbx/rojo@7.x.x"
wally = "UpliftGames/wally@0.3.x"
selene = "Kampfkarren/selene@0.2x.x"
stylua = "JohnnyMorganz/StyLua@2.x.x"
lune = "lune-org/lune@0.x.x"
```

`rokit install` pins versions per project (Aftman/Foreman are older equivalents). Pin exact versions in real projects.

## Rojo project

```json
{
  "name": "my-game",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Shared": { "$path": "src/shared" },
      "Packages": { "$path": "Packages" }
    },
    "ServerScriptService": { "Server": { "$path": "src/server" } },
    "StarterPlayer": {
      "StarterPlayerScripts": { "Client": { "$path": "src/client" } }
    }
  }
}
```

File naming: `init.server.luau` / `*.server.luau` → Script, `*.client.luau` → LocalScript, `*.luau` → ModuleScript, `init.luau` makes a folder a module. `*.model.json`/`.rbxm` for instances. `rojo serve` + Rojo Studio plugin → *Connect*. `rojo build -o game.rbxl` for CI.

Keep maps/builds in the place file (or partially managed Rojo trees) — only code and config in Git.

## Packages (Wally)

```toml
# wally.toml
[package]
name = "studio/my-game"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
Promise = "evaera/promise@4.0.0"
Trove = "sleitnick/trove@1.x.x"
React = "jsdotlua/react@17.x.x"
```

`wally install` → `Packages/`; run `wally-package-types` for luau-lsp typing of re-exports.

## Quality

- **luau-lsp** (VS Code extension) with Rojo sourcemap (`rojo sourcemap --watch -o sourcemap.json`) for autocompletion and type checking; `--!strict` everywhere.
- **Selene** (`selene.toml` with `std = "roblox"`) for linting; **StyLua** for formatting (`.stylua.toml`, tabs, 120 cols).
- CI: `selene src`, `stylua --check src`, `luau-lsp analyze`.

## Testing

- Unit-test pure modules with **Jest-lua** (Roblox's port) or TestEZ inside Studio / via `run-in-roblox`.
- **Lune** runs Luau scripts standalone (build scripts, data processing, some tests without Studio).
- Open Cloud **Luau Execution** API can run test scripts in a real server for CI.

## Places & environments

- Use separate **experiences** (or at least places) for Dev/Staging/Production with separate DataStores.
- Never test DataStore migrations against production data.
- Publish via File → Publish to Roblox, or CI with Open Cloud Place Publishing API (`rbxcloud`/`rojo upload` with an API key scoped to the universe).

## Studio tips

- Command bar for quick edits; `Ctrl+Shift+P` (Find All), Script Analysis window.
- Test → *Clients and Servers* (multiple players locally) to test replication.
- Device Emulator for mobile UI; Network simulation (incoming replication lag) in Studio settings.
- MicroProfiler and Developer Console (F9) for live-server debugging (server logs visible to owners).
- Packages (Roblox Packages, not Wally) to share models across places with auto-update.

## Release hygiene

- Changelog + version string in ReplicatedStorage (`game.PlaceVersion` gives the published version number).
- Feature flags via attributes or a config module to disable features without republishing (with MessagingService/MemoryStore for live toggles).
- Shut down old servers after critical updates (*Migrate to Latest Update* in Creator Hub).
