---
name: steam-publish
description: Publish a game on Steam — Steamworks onboarding and fee, app/depot setup, store page assets and copy, Coming Soon timing and wishlists, Steam Next Fest, builds with SteamPipe (steamcmd) and branches, Steamworks SDK integration (achievements, cloud saves, overlay, Steam Deck verification), pricing and regional pricing, release checklist and review process, and post-launch updates. Use when preparing, uploading, or launching a game on Steam.
---

# Steam Publishing

Requirements and fees change — verify current details in the Steamworks documentation (partner.steamgames.com/doc).

## Timeline (typical indie)

| When | Step |
|---|---|
| ASAP (6–12+ months before release) | Steam Direct fee (US$100 per app, recoupable after revenue threshold), tax/bank info, create app |
| Early | **Store page "Coming Soon"** live to collect wishlists (requires review ~3–5 business days) |
| Months before | Demo + **Steam Next Fest** (register in advance) |
| ≥ 2 weeks before launch | Store page must have been public as Coming Soon for at least 2 weeks before release; build review |
| Launch | Release button; launch discount (optional) |

Wishlists drive launch visibility — the store page is a marketing asset, launch it early.

## Store page assets

- Capsules: header, small, main, vertical, library assets (hero, logo, capsule) at the exact sizes requested in Steamworks.
- **Capsule art** readable at thumbnail size, with the game's name.
- Trailer: gameplay in the first 5 seconds; 30–90 s.
- 5+ screenshots of actual gameplay (no concept art), 16:9.
- Short description (≤ 300 chars) that says genre + hook; long description with GIFs and feature bullets.
- Tags: choose accurate genre/subgenre tags (they determine who sees the game).
- AI content disclosure if applicable; mature content survey; system requirements.

## SteamPipe uploads

```
app_build_1234560.vdf
"AppBuild"
{
  "AppID" "1234560"
  "Desc" "v1.0.3"
  "ContentRoot" "../builds/"
  "BuildOutput" "../output/"
  "SetLive" "beta"
  "Depots"
  {
    "1234561" { "FileMapping" { "LocalPath" "windows/*" "DepotPath" "." "recursive" "1" } }
    "1234562" { "FileMapping" { "LocalPath" "linux/*"   "DepotPath" "." "recursive" "1" } }
  }
}
```

```bash
steamcmd +login <builder_account> +run_app_build ../scripts/app_build_1234560.vdf +quit
```

- Use a dedicated builder account with limited permissions; Steam Guard via steamcmd once, then cached.
- One depot per OS; set launch options per OS in Steamworks (Installation → General).
- Branches: `default` (public), `beta` (password or open), for testing. Promote a build to default in Builds page.
- CI: GitHub Actions with `game-ci/steam-deploy` or a self-hosted runner running steamcmd (store config/ssfn secrets securely).

## SDK integration

Engines: Godot (GodotSteam), Unity (Steamworks.NET / Facepunch.Steamworks), Unreal (OnlineSubsystemSteam plugin), others via C API/wrappers.

- `steam_appid.txt` with the AppID next to the executable **for local testing only** — don't ship it in depots; Steam provides the AppID when the game is launched from the client.
- Initialize Steam API at startup; if Steam isn't running, `SteamAPI_RestartAppIfNecessary` relaunches through Steam (DRM-free option: allow running without).
- Achievements & stats: define in Steamworks, publish, call `SetAchievement` + `StoreStats`.
- **Steam Cloud**: Auto-Cloud (configure save paths, no code) or the Remote Storage API.
- Rich presence, overlay (needs rendering hooks; test fullscreen/borderless), Steam Input (configure default controller layouts — strongly recommended for Deck).

## Steam Deck

Aim for **Verified**: default controller config with full gamepad support, on-screen keyboard for text input, legible text at 1280×800, no launcher, works under Proton (or native Linux). Request review in Steamworks.

## Pricing

Set base USD price; accept Valve's regional pricing suggestions (or adjust). Launch discount 10–20% common. Price changes have cooldown rules relative to discounts.

## Release checklist

- [ ] Store page and build both approved
- [ ] Build tested from the Steam client on a clean machine/account (not the dev folder)
- [ ] Achievements fire; cloud saves sync between two machines
- [ ] Controller support flags set accurately
- [ ] Default branch set to the release build
- [ ] Press kit + key distribution (Steamworks keys, Curator Connect)
- [ ] Day-1 patch plan; crash reporting live
- [ ] Release time chosen (avoid huge launches the same day)

## Post-launch

Patch notes as Steam events/announcements (drive re-engagement), respond to reviews constructively, participate in seasonal sales and themed fests, consider updates/DLC to re-surface the game.
