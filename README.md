# Two Worlds 1 — Model Injection Guide & Tools

Complete guide and tools for importing custom 3D models into Two Worlds 1 or editing existing ones. Covers VDF format, Blender workflow, material/shader setup, NTF node structure, and editor integration.

![Python 3.6+](https://img.shields.io/badge/python-3.6+-blue) ![License: MIT](https://img.shields.io/badge/license-MIT-green) ![Platform: Windows](https://img.shields.io/badge/platform-Windows-lightgrey)

## Overview

Two Worlds 1 uses a proprietary model format (`.vdf`) with an embedded node tree (NTF) that defines meshes, shaders, textures, and locators. Getting a custom model into the game requires multiple steps across different file formats and tools. This repository bundles everything you need — from importing and editing an existing VDF in Blender to in-game placement — into one place with step-by-step guides.

### What's Included

- 🔧 **Blender VDF Plugin** — native Blender import/export for VDF files (byte-identical roundtrip)
- 🔧 **NTF Editor** — GUI editor for the VDF/NTF node tree (shaders, textures, locators, shader transplant)
- 🔧 **PAR Editor** — GUI editor for `TwoWorlds.par` to register new objects, edit parameters, and manage entries
- 📖 **Step-by-step guide** — full workflow from modeling to in-game placement (EN/DE)

### What You Also Need

- 📦 **WD Repacker** — pack your files into `.wd` archives for the game → [Download (ModDB)](https://www.moddb.com/games/two-worlds/downloads)
- 📦 **Mod Selector** — enable/disable mods and manage WD load order → [Download (ModDB)](https://www.moddb.com/games/two-worlds/downloads)
- 📦 **Two Worlds SDK** — official editor for placing objects on maps → [Download (ModDB)](https://www.moddb.com/games/two-worlds/downloads/two-worlds-software-development-kit-tools-13-installer)

## Workflow Overview

Adding a new model to Two Worlds 1 follows these steps:

```
1. IMPORT BASE      Blender VDF Plugin → import existing similar VDF as base
                    ↓
2. MODIFY & EXPORT  Edit mesh in Blender → export back to VDF (metadata preserved)
                    ↓
3. SETUP SHADERS    NTF Editor → assign textures, shader type, material properties
                    (or use Transplant to copy shader setup from existing VDF)
                    ↓
4. REGISTER IN PAR  PAR Editor → duplicate existing entry → update mesh path
                    ↓
5. EDITORDEF        Add entry name to EditorDef.txt
                    ↓
6. PACK & TEST      WD Repacker → pack into .wd → place object in editor → test
```

## Tools

| Tool | Description | Formats |
|------|-------------|---------|
| **Blender VDF Plugin** | Native Blender import/export, byte-identical roundtrip | `.vdf` ↔ Blender |
| **NTF Editor v1.0** | GUI editor for VDF node tree — shaders, textures, locators | `.vdf` / `.ntf` |
| **PAR Editor v1.3** | Register objects, edit parameters, compare & merge PAR files | `.par` / `.json` |

For a complete overview of all available modding tools, see the [TW1 Modding Hub](https://github.com/MedievalDev/Two-Worlds-Modding-HUB).

## Installation

### Requirements

- Python 3.6 or higher
- Windows (for editor integration and DPI awareness)
- Blender 2.8+ (for the VDF plugin)
- tkinter (included with Python on Windows)

### Setup

1. Download or clone this repository
2. Place the tool files in your modding workspace:

```
YourModdingFolder/
├── ntf_editor.py              (NTF Editor GUI)
├── tw1_par_editor.py           (PAR Editor GUI)
├── tw1_sdk_labels.json         (1808 SDK field labels for PAR Editor)
├── tw1_sdk_descriptions.json   (1666 tooltip descriptions for PAR Editor)
├── blender_vdf_plugin/         (Blender addon folder)
│   └── __init__.py
└── guides/
    ├── model_injection_en.md
    └── model_injection_de.md
```

3. For the Blender plugin: Edit → Preferences → Add-ons → Install → select the plugin folder

4. Run any tool directly:
```
python ntf_editor.py
python tw1_par_editor.py
```

## File Formats

### VDF (Vertex Data Format)

The game's proprietary 3D model format. Contains vertex data, normals, UVs, and an embedded NTF node tree that defines the material/shader hierarchy. **Important:** New models cannot be created from scratch — you always need to import an existing, similar VDF as a base because the embedded NTF data (shaders, locators, metadata) cannot be reliably generated manually. The Blender plugin preserves this data during roundtrip. See the [Modding Guide](https://github.com/MedievalDev/Two-Worlds-Modding-Guid) for the full binary specification.

### NTF (Node Tree Format)

Embedded within VDF files. Defines a tree of typed nodes:

```
Root / AnimRef / FrameRange
├── Locator "PIVOT36"              (attachment point)
├── Model "$Merged$"               (mesh data)
│   └── Shader "Wood"              (material definition)
│       ├── ShaderName: "lightmap.efx"
│       ├── TexS0: "Catapult_wood.dds"       (diffuse texture)
│       ├── TexS1: "Catapult_wood_bump.dds"  (bump map)
│       ├── TexS2: "ROADSIGN_02.dds"         (lightmap)
│       ├── SpecColor: [0.7, 0.7, 0.7, 16.0]
│       └── DestColor: [0.5, 0.5, 0.5, 1.0]
└── Model "$Merged$"
    └── Shader "Metal"
```

### MTR (Material Reference)

A companion file that lists all textures used by a VDF. Must be placed alongside the VDF in the same directory. The engine uses this for texture preloading.

### Key Directories

```
Game Root/
├── Models/
│   └── ADDONS/
│       └── ROADSIGNS/
│           ├── ROADSIGN_L_14.vdf    (3D model)
│           └── ROADSIGN.mtr         (material reference)
├── Parameters/
│   ├── TwoWorlds.par                (object definitions)
│   └── EditorDef                    (editor object browser list)
└── WDFiles/
    └── YourMod.wd                   (packed mod archive)
```

## Step-by-Step Example: Adding a New Road Sign

This example walks through adding `ROADSIGN_L_14` to the game — a practical real-world scenario showing the complete pipeline.

### 1. Create or Copy the Model

Start by importing an existing, similar road sign VDF into Blender via the VDF Plugin. The VDF contains essential metadata (node tree, shader references, locators) that can't be created from scratch — you always need an existing VDF as a base. Modify the mesh in Blender, then export back to VDF. The plugin preserves all original VDF data and only updates the geometry.

```
Import ROADSIGN_L_13.vdf into Blender → modify mesh → export as ROADSIGN_L_14.vdf
```

### 2. Edit Textures with NTF Editor

Open `ROADSIGN_L_14.vdf` in the NTF Editor:
- Navigate to the **Shader** node (e.g. "Wood")
- Update `TexS0` (diffuse), `TexS1` (bump), `TexS2` (lightmap) to your textures
- Or use **Transplant** to copy the complete shader setup from another working VDF
- Save the VDF

### 3. Register in TwoWorlds.par

Open `TwoWorlds.par` in the [PAR Editor](https://github.com/MedievalDev/TwoWorlds_PAR_Editor):
1. Search for `ROADSIGN_L_13`
2. Right-click → **Duplicate** → name it `ROADSIGN_L_14`
3. Update field `[1] mesh` to `ADDONS\ROADSIGNS\ROADSIGN_L_14.vdf`
4. Adjust `meshFlags`, `passiveFlags`, `maxDrawDistanceA` or other parameters as needed
5. Save

### 4. Add to EditorDef.txt

Open `Parameters/EditorDef` in a text editor. Find the ROADSIGN group and add your new entry:

```
ROADSIGN_L_13
ROADSIGN_L_14        ← add this line
ROADSIGN_STICK
```

### 5. Place Files

```
Models/ADDONS/ROADSIGNS/ROADSIGN_L_14.vdf     (your model)
Models/ADDONS/ROADSIGNS/ROADSIGN.mtr           (update if new textures added)
Parameters/TwoWorlds.par                        (with new entry)
Parameters/EditorDef                            (with new name)
```

### 6. Pack and Test

Pack everything into a `.wd` archive using the [WD Repacker](https://www.moddb.com/games/two-worlds/downloads). Place the `.wd` file in the game's `WDFiles/` folder. Enable it with the [Mod Selector](https://www.moddb.com/games/two-worlds/downloads). Launch the Two Worlds Editor — your new road sign should appear in the object browser under its group.

## Mesh Field Syntax

The `mesh` field in PAR entries supports extended syntax for variants and texture overrides:

```
Basic:         ADDONS\ROADSIGNS\ROADSIGN_L_14.vdf
Variants:      ADDONS\ROADSIGNS\ROADSIGN_L_[1-14].vdf        (right-click to cycle in editor)
Texture:       Houses\STABLE.vdf:ALT_TEXTURE.dds               (override default texture)
Multi-Tex:     Houses\STAIRS_0[1-4].vdf:TEX_A.dds|TEX_B.dds   (variants + multiple textures)
```

See the [PAR Editor documentation](https://github.com/MedievalDev/TwoWorlds_PAR_Editor) for full mesh syntax details.

## Related Repositories

| Repository | Description |
|-----------|-------------|
| [TW1 Modding Hub](https://github.com/MedievalDev/Two-Worlds-Modding-HUB) | Central launcher for all 14 modding tools |
| [TW1 Modding Guide](https://github.com/MedievalDev/Two-Worlds-Modding-Guid) | Complete modding documentation & file format specs |
| [TW1 PAR Editor](https://github.com/MedievalDev/TwoWorlds_PAR_Editor) | GUI editor for game parameters (objects, NPCs, weapons, spells) |
| [TW1 VDF Import/Export](https://github.com/MedievalDev/Two-Worlds-VDF-In-Export-Tool) | VDF/OBJ converters and NTF Editor |
| [TW1 LND Viewer](https://github.com/MedievalDev/Twor-Worlds-LND-Viewer) | Map tools — LND viewer, World Viewer, Object Exporter |
| [TW1 Dialog Viewer](https://github.com/MedievalDev/Twor-Worlds-Dialog-Viewer-Editor) | Translation & quest editor (.lan, .idx, .qtx, .shf) |
| [TW Editor CMD Injector](https://github.com/MedievalDev/TwoWorldsEditor_Command_Injector) | Editor automation via command injection |

## Community Resources

- [Two Worlds SDK (ModDB)](https://www.moddb.com/games/two-worlds/downloads/two-worlds-software-development-kit-tools-13-installer) — Official SDK with editor, documentation, and EarthC compiler
- [Two Worlds Downloads (ModDB)](https://www.moddb.com/games/two-worlds/downloads) — Mod Selector, WD Repacker, Control Panel, and community mods
- [Two Worlds Steam Forum](https://steamcommunity.com/app/1930/discussions/) — Community discussions and mod support

## Credits

- **Reality Pump Studios** — Two Worlds game engine, SDK, and editor
- VDF/NTF format reverse-engineered from binary analysis and SDK correlation
- Community contributions from Buglord (original NTF format research, Mod Selector, WD Repacker)
- JadetheReaper (map conversion documentation, test maps)

## License

MIT
