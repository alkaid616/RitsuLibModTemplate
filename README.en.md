# RitsuLibModTemplate

Languages: [中文](README.md) | English

A copyable, buildable RitsuLib mod template providing a general Godot/C# project layout, sample content, and static placeholder assets.

**What the template includes:**

- A `[ModInitializer]` entry point plus a minimal custom character (with character card pool, relic pool, and potion pool).
- Four starter strikes, four starter defends, and one starter relic as samples.
- Minimal static Godot placeholder scenes for the combat character, energy counter, character select background, merchant, and rest site.
- Placeholder PNG files copied from vanilla resources and renamed for the template. Replace them after copying.
- Basic English and Simplified Chinese localization files.
- A complete Godot project, export preset, mod manifest, and MSBuild scripts.

## Learning Resources

- [STS2-RitsuLib](https://github.com/BAKAOLC/STS2-RitsuLib): the shared framework library for Slay the Spire 2 mods. This template uses it for content registration, character scaffolding, and Godot resource integration.
- [RitsuLib Documentation](https://github.com/GlitchedReme/SlayTheSpire2ModdingTutorials/tree/master/RitsuLib): tutorials and examples by file.
- [Slay the Spire 2 Modding Tutorials site](https://glitchedreme.github.io/SlayTheSpire2ModdingTutorials/index.html): the full tutorial site.
- Template Wiki (Rider-first): [Chinese Home](https://github.com/alkaid616/RitsuLibModTemplate/wiki/Home) | [English Home](https://github.com/alkaid616/RitsuLibModTemplate/wiki/Home-EN).

## Install the Template

### Command line

```powershell
# Install the template
dotnet new install STS2.RitsuLib.ModTemplate

# Create a new mod
dotnet new ritsulibmod -n MyMod

# Uninstall the template
dotnet new uninstall STS2.RitsuLib.ModTemplate
```

`dotnet new ritsulibmod -n MyMod` generates a project called `MyMod` and renames `RitsuLibModTemplate`, sample class names, sample resource file names, resource folders, manifest names, namespaces, and localization IDs to match the new name.

### Creating a project in Rider

After running `dotnet new install STS2.RitsuLib.ModTemplate` once, Rider will pick up the template automatically:

1. `File` → `New Solution` (or `New Solution` from the welcome screen).
2. Scroll down the template list on the left and pick `RitsuLib Mod Template` under `Custom Templates` (or `More Templates`). Short name: `ritsulibmod`.
3. Enter `Solution name` (the new mod name, e.g. `MyMod`) and a location, then click `Create`.

Rider runs `dotnet new ritsulibmod -n <Solution name>` under the hood, matching the CLI behavior. If the template does not appear, confirm `dotnet new install` succeeded and then refresh the `New Solution` dialog or restart Rider.

## Local Path Configuration

```powershell
Copy-Item .\local.props.template .\local.props
```

Set these values in `local.props` (the file is in `.gitignore`; do not commit it):

| Field | Description |
|---|---|
| `Sts2Dir` | Slay the Spire 2 install directory |
| `Sts2DataDir` | Game DLL directory, usually `$(Sts2Dir)/data_sts2_windows_x86_64` |
| `GodotExe` | MegaDot/Godot executable used to export the PCK |
| `RitsuLibDeployDir` | Local RitsuLib deployment directory, defaulting to `$(Sts2Dir)/mods/STS2-RitsuLib`. Used by RitsuLib package/build logic to copy RitsuLib into the game's mods directory — **not** this mod's output directory |

## RitsuLib Version Compatibility

> ⚠️ **Important: align the manifest's RitsuLib version with the csproj before release**
>
> `dependencies[STS2-RitsuLib].version` in `RitsuLibModTemplate.json` **must exactly match** the `STS2.RitsuLib` version your `.csproj` actually compiles against. The template build auto-syncs this dependency version; `min_game_version` and intentional lower runtime-floor declarations still need manual review. See [Pre-release checklist: version alignment](#pre-release-checklist-version-alignment) below for the step-by-step procedure.

### Current version snapshot (as of 2026-05-22)

| Item | Value |
|---|---|
| Current STS2 game version | `0.106.0` |
| Current RitsuLib version | `0.3.0` |
| Template manifest status | `min_game_version` and `dependencies[STS2-RitsuLib].version` are aligned |

### Version mapping

The table summarizes the mainline STS2 target for each boundary RitsuLib release, sourced from the [STS2-RitsuLib Releases](https://github.com/BAKAOLC/STS2-RitsuLib/releases) page. Patch versions not listed follow the range they sit in; check the relevant release notes for boundary versions.

| RitsuLib version | Mainline STS2 target | Compat packages |
|---|---|---|
| `v0.3.0+` (since 2026-05-22) | `0.106.0` | `0.103.2`; `0.104.0` compat removed |
| `v0.2.29` ~ `v0.2.40` | `0.105.1` | `0.104.0`, `0.103.2` |
| `v0.2.27` ~ `v0.2.28` | `0.105.0` | `0.104.0`, `0.103.2` |
| `v0.2.0` ~ `v0.2.26` | `0.104.0` | `0.103.2` (experimental from `v0.2.6`); `0.99.1` compat removed in this range |
| `v0.0.x` / `v0.1.x` | `0.99.1` and earlier | — |

### Package selection: mainline and compat

The template references mainline `STS2.RitsuLib` by default, tracking the latest NuGet version:

```xml
<PackageReference Include="STS2.RitsuLib" Version="*" GeneratePathProperty="true" />
```

**Only enable one RitsuLib package at a time.** If your code still targets an older branch, comment out the mainline and enable the matching compat package:

```xml
<!-- STS2 0.104.0 compatibility branch (no longer maintained since v0.3.0) -->
<PackageReference Include="STS2.RitsuLib.Compat.0.104.0" Version="*" />

<!-- STS2 0.103.2 compatibility branch -->
<PackageReference Include="STS2.RitsuLib.Compat.0.103.2" Version="*" />
```

Compatibility packages only select the matching game branch; they do not restore every old API. Some old mods still need code changes and recompilation.

The template also references `Nothing.STS2RitsuLib.ModAnalyzers` — an AI-written helper analyzer that reports common manifest and resource configuration issues during development.

### Pre-release checklist: version alignment

> **`PackageReference` in `.csproj` only controls compile-time resolution; `dependencies` in `RitsuLibModTemplate.json` is what the game loader checks at runtime. The template syncs the `STS2-RitsuLib` dependency version to the resolved NuGet version during build, but `min_game_version` still needs manual review.**

If the manifest is not synced to the newer RitsuLib version you compile against, players with an old RitsuLib will pass the manifest check and crash at runtime due to missing APIs or signature drift. Conversely, an over-tight manifest will reject players who could otherwise run the mod.

Before every release:

1. After build, confirm `dependencies[STS2-RitsuLib].version` in `RitsuLibModTemplate.json` has been synced to the resolved `STS2.RitsuLib` version.
2. When you switch to a compatibility package (`Compat.0.104.0` / `Compat.0.103.2`), also adjust `min_game_version` to the matching branch. Keep `dependencies[].id` as `STS2-RitsuLib` (compatibility packages expose the same mod id to the loader).
3. If you intentionally want the manifest version to act as a **runtime floor** (e.g. declaring "`0.3.0+` works"), document this in your release notes and verify the mod runs against the declared floor.

### Upgrade notes

#### Upgrading to RitsuLib `v0.3.0` / STS2 `0.106.0`

Major changes (from the [v0.3.0 release notes](https://github.com/BAKAOLC/STS2-RitsuLib/releases/tag/v0.3.0)):

- **Breaking**: `RunSidecar` removed, fully replaced by `RunSavedData`.
- New `TargetType` registration capability for custom `TargetType`s.
- Loader target detection strengthened: branch version files now use hash verification, and mismatched versions are discarded.
- `0.104.0` compatibility removed.

#### Upgrading to RitsuLib `v0.2.27` / STS2 `0.105.0` (historical)

When migrating from an earlier branch (`v0.2.0` ~ `v0.2.26` / STS2 `0.104.0`), check the following:

- Version conditional compilation switched to cumulative interval macros `STS2_AT_LEAST_<ver>`; legacy `STS2_V_<ver>` macros are no longer recommended.
- AnyPlayer / AnyAny targeting logic changed; legacy card targets, base constructor signatures, and registration flows should be checked against the new API.
- Cards support extra icon count labels in the lower-right corner with vanilla UI conflict handling; verify display order and placement for custom UI / icon patches.
- Retain / flush hooks and events have replacements, removals, or `[Obsolete]` markers; migrate legacy uses of `CardRetainedEvent`, `CardsFlushedEvent`, or legacy `Hook.*` entry points.
- `Badge`, `BadgeRuntimeTemplate`, `BadgePool.CreateAll`, and `ModBadgeTemplate` constructor signatures changed; legacy code may need updates to avoid `MissingMethodException`.

## Build

| Command | Behavior |
|---|---|
| `dotnet build .\RitsuLibModTemplate.csproj` | Full build: compile + `CopyMod` + `ExportPCK` |
| `... /p:RunPckExport=false` | Skip PCK export (no `GodotExe` needed) |
| `... /p:CopyModOnBuild=false` | Skip copying to the game's mods directory (output stays in `bin/`) |
| `... /p:RunPckExport=false /p:CopyModOnBuild=false` | C# compile validation only |

A full build runs two MSBuild targets after `Build`:

- **`CopyMod`**: copies the DLL and manifest to the game's `mods/RitsuLibModTemplate` directory.
- **`ExportPCK`**: calls `GodotExe` and exports the PCK to the same mod directory.

> `RitsuLibDeployDir` only controls where the RitsuLib framework itself is deployed locally. This mod's DLL, manifest, and PCK are controlled by `ModOutputDir` (default `$(Sts2Dir)/mods/$(MSBuildProjectName)`).

## Directory Layout

```text
RitsuLibModTemplate/
├── RitsuLibModTemplateCode/   # C# source
├── RitsuLibModTemplate/       # Godot resources, localization, and placeholder scenes
├── RitsuLibModTemplate.csproj
├── RitsuLibModTemplate.json   # Mod manifest
├── project.godot
└── local.props.template
```

`res://RitsuLibModTemplate/...` is the Godot/PCK resource path, mapping to the repository resource directory `RitsuLibModTemplate/`, **not** to the C# namespace. When you create a project from the NuGet template, these directory names, file names, and namespaces are renamed consistently to match the new mod name.

## Template Contents

### Sample character

| Property | Value |
|---|---|
| Type | `RitsuLibModTemplateCharacter` |
| Expected ID | `RITSU_LIB_MOD_TEMPLATE_CHARACTER_RITSU_LIB_MOD_TEMPLATE_CHARACTER` |
| Starter deck | 4 × `RitsuLibModTemplateStrike`, 4 × `RitsuLibModTemplateDefend`, 1 × `RitsuLibModTemplateRelic` |
| Assets | Configured via `CharacterAssetProfile`. The template only specifies static placeholder assets; unspecified audio, trail, transition, etc. fall back through `PlaceholderCharacterId` |

### Sample cards and relic

| Type | Pool | Expected ID |
|---|---|---|
| `RitsuLibModTemplateStrike` (attack) | character card pool | `RITSU_LIB_MOD_TEMPLATE_CARD_RITSU_LIB_MOD_TEMPLATE_STRIKE` |
| `RitsuLibModTemplateDefend` (skill) | character card pool | `RITSU_LIB_MOD_TEMPLATE_CARD_RITSU_LIB_MOD_TEMPLATE_DEFEND` |
| `RitsuLibModTemplateRelic` | `RitsuLibModTemplateRelicPool` | `RITSU_LIB_MOD_TEMPLATE_RELIC_RITSU_LIB_MOD_TEMPLATE_RELIC` |

### Static placeholder assets

**Images** (`res://RitsuLibModTemplate/images/...`):

- `cards/RitsuLibModTemplateStrike.png`, `cards/RitsuLibModTemplateDefend.png`: sample card art.
- `relics/RitsuLibModTemplateRelic.png`: sample relic icon.
- `characters/RitsuLibModTemplate_character_*.png`: character icons, select art, map marker, and energy icons.

**Scenes** (`res://RitsuLibModTemplate/scenes/characters/...`):

| Scene | Purpose | Placeholder structure |
|---|---|---|
| `RitsuLibModTemplate_character.tscn` | Combat character | `%Visuals`, `%Bounds`, `%IntentPos`, `%CenterPos`, `%TalkPos` |
| `RitsuLibModTemplate_energy_counter.tscn` | Energy counter | `%EnergyVfxBack`, `%Layers`, `%RotationLayers`, `%EnergyVfxFront`, `Label` |
| `RitsuLibModTemplate_merchant.tscn` | Merchant | — |
| `RitsuLibModTemplate_rest_site.tscn` | Rest site | `%ControlRoot`, `%SelectionReticle`, `%Hitbox`, `%ThoughtBubbleRight`, `%ThoughtBubbleLeft` |
| `RitsuLibModTemplate_character_select_bg.tscn` | Character select background | — |

These resources only exist to make the template visible and replaceable; they do not try to reproduce vanilla animation quality. After copying the template, replace them with your own assets. If you change paths, update the corresponding `AssetProfile` fields.

## Manifest Format

`RitsuLibModTemplate.json` is the mod manifest. The game loader reads it at startup to identify the mod, check dependencies, and decide whether to load. Full example:

```json
{
  "id": "RitsuLibModTemplate",
  "name": "RitsuLibModTemplate",
  "pck_name": "RitsuLibModTemplate",
  "author": "Author",
  "description": "A starter Slay the Spire 2 mod template built on RitsuLib.",
  "version": "0.0.0",
  "has_pck": true,
  "has_dll": true,
  "affects_gameplay": true,
  "min_game_version": "0.106.0",
  "dependencies": [
    { "id": "STS2-RitsuLib", "version": "0.3.0" }
  ]
}
```

### Field reference

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique mod identifier. **Must match `Entry.ModId` exactly**, and should also match the `mods/<id>` directory name. In-game dependency lookups, localization key prefixes, and resource paths all depend on this value |
| `name` | string | Display name shown in the mod list. May contain spaces and non-ASCII characters |
| `pck_name` | string | `.pck` file name (without extension). **Must match the actual PCK file produced by `.csproj`**, or resources will not load even with `has_pck=true` |
| `author` | string | Display-only author name |
| `description` | string | Short description shown in the mod list |
| `version` | string | Version of this mod itself. SemVer (`MAJOR.MINOR.PATCH`) is recommended. Bump on every release |
| `has_pck` | bool | Whether the mod ships a `.pck`. Code-only mods can set `false` and skip `ExportPCK` |
| `has_dll` | bool | Whether the mod ships a `.dll`. Resource-only mods can set `false` |
| `affects_gameplay` | bool | Whether the mod affects gameplay. When enabled, the game flags saves/achievements/etc.; only purely cosmetic / localization mods should set this to `false` |
| `min_game_version` | string | Minimum compatible STS2 version. Older games refuse to load. **Must align with the game branch targeted by the RitsuLib package selected in `.csproj`** (see [RitsuLib Version Compatibility](#ritsulib-version-compatibility) above) |
| `dependencies` | array | Dependency list. Each entry uses `id` + `version`. **The legacy single-object `min_version` form is no longer supported** |
| `dependencies[].id` | string | The depended-on mod's `id`. RitsuLib itself uses `STS2-RitsuLib` |
| `dependencies[].version` | string | Minimum runtime version of the dependency. **The `STS2-RitsuLib` value must exactly match the NuGet version your `.csproj` actually compiles against** — see [Pre-release checklist: version alignment](#pre-release-checklist-version-alignment) above |

## Development Tips

- Prefer `AssetProfile` for new content; only override legacy `Custom...Path` fields for individual compatibility cases.
- If a character resource field is not specified, RitsuLib fills it from the vanilla character config referenced by `PlaceholderCharacterId`.
- Resource paths must start with `res://`; verify the directory name and casing inside the PCK are correct.
- For `.tscn` files, make sure the scene is packed into the mod resources. If it needs a script, prefer a local wrapper class and call `EnsureGodotScriptsRegistered(...)` from `Entry.Initialize()`.
