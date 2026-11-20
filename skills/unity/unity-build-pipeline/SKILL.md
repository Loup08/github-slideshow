---
name: unity-build-pipeline
description: Build Unity 6 projects reliably — Build Profiles, scripting backends (Mono vs IL2CPP), scripting define symbols, command-line batchmode builds with BuildPipeline, CI (GameCI), versioning, Addressables content builds, and platform settings. Use when producing builds locally or in CI.
---

# Unity Build Pipeline

## Build Profiles (Unity 6)

File → Build Profiles replaces Build Settings. Create profiles per target/configuration (e.g. *Windows Release*, *Windows Demo*, *Android*), each with its own scene list, scripting defines and player-setting overrides. Profiles are assets — commit them.

## Key player settings

- **Scripting Backend**: IL2CPP for release (performance, required for iOS/consoles, Android 64-bit); Mono for fast iteration.
- **Managed Stripping Level**: Low/Medium; preserve reflection-only types with `link.xml` or `[Preserve]`.
- **API Compatibility**: .NET Standard 2.1.
- Company/Product name + Bundle Identifier (`com.studio.game`) — changing later breaks saves path / store listings.
- Version (`PlayerSettings.bundleVersion`) and Android `bundleVersionCode` must increase each store upload.

## Command-line builds

```csharp
// Assets/_Project/Scripts/Editor/BuildScript.cs
using UnityEditor;
using UnityEditor.Build.Reporting;
using UnityEngine;

public static class BuildScript
{
    public static void BuildWindows()
    {
        var options = new BuildPlayerOptions
        {
            scenes = EditorBuildSettingsScenesEnabled(),
            locationPathName = "Builds/Windows/Game.exe",
            target = BuildTarget.StandaloneWindows64,
            options = BuildOptions.None,
        };
        BuildReport report = BuildPipeline.BuildPlayer(options);
        if (report.summary.result != BuildResult.Succeeded)
            EditorApplication.Exit(1);
    }

    private static string[] EditorBuildSettingsScenesEnabled() =>
        System.Array.ConvertAll(System.Array.FindAll(EditorBuildSettings.scenes, s => s.enabled), s => s.path);
}
```

```bash
Unity -batchmode -nographics -quit -projectPath . \
  -executeMethod BuildScript.BuildWindows -logFile build.log
# Unity 6 can also build a profile directly:
Unity -batchmode -quit -projectPath . -activeBuildProfile "Assets/Settings/Build Profiles/Windows Release.asset" -build Builds/Windows/Game.exe
```

Run tests: `Unity -batchmode -projectPath . -runTests -testPlatform EditMode -testResults results.xml` (no `-quit` with `-runTests`).

## CI with GameCI (GitHub Actions)

```yaml
- uses: actions/cache@v4
  with: { path: Library, key: Library-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }} }
- uses: game-ci/unity-test-runner@v4
  env: { UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}, UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}, UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }} }
- uses: game-ci/unity-builder@v4
  with: { targetPlatform: StandaloneWindows64, buildMethod: BuildScript.BuildWindows }
```

Cache `Library/` — it's the biggest time saver.

## Defines

Per-profile scripting defines (`DEMO`, `STEAM`, `ENABLE_CHEATS`) + `#if DEMO`. `Debug.Log` calls still cost in release — wrap noisy logs with `[Conditional("ENABLE_LOGS")]`.

## Addressables

If using Addressables, build content before the player (`AddressableAssetSettings.BuildPlayerContent()` in the build script, or enable *Build Addressables on Player Build*). Remote catalogs for live content updates.

## Platform checklist

- **Windows/Mac/Linux**: architecture x64/ARM64 (Apple Silicon universal). macOS notarization outside the App Store.
- **Android**: target API per Google Play requirements, AAB, keystore (back it up!), IL2CPP + ARM64.
- **iOS**: generates Xcode project; signing in Xcode.
- **WebGL**: compression Brotli/Gzip needs correct server headers (or *Decompression Fallback*); memory size; no threads by default.

## Pitfalls

- Editor-only code (`using UnityEditor`) in runtime assemblies breaks builds — guard with `#if UNITY_EDITOR` or Editor asmdefs.
- Scenes missing from the profile's scene list → `SceneManager.LoadScene` fails only in builds.
- Always test a development build with *Autoconnect Profiler* before release.
