# GameJam2026

Game jam project in Unreal Engine 5.8. There is a C++ module (`Source/GameJam2026`, depends on Enhanced Input), but it is still the empty template: all gameplay so far lives in Blueprints under `Content/`.

## Environment
- Engine: UE 5.8.4 launcher build at `E:\Epic Games\UE_5.8`. The path has spaces, so always quote it.
- Toolchain: Visual Studio Community 2026 with MSVC (required components are listed in `.vsconfig`).
- `.uasset` and `.umap` files are stored in Git LFS (`.gitattributes`).
- Generated, never edit or commit: `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`, `*.sln`, `*.slnx`.

## Building C++
First check whether the editor is open: `Get-Process UnrealEditor -ErrorAction SilentlyContinue`.

- **Editor closed:** build the editor target.
  ```powershell
  & "E:\Epic Games\UE_5.8\Engine\Build\BatchFiles\Build.bat" GameJam2026Editor Win64 Development -Project="F:\Working_Games\JamGame\GameJam2026\GameJam2026.uproject" -WaitMutex
  ```
- **Editor open:** Unreal Build Tool refuses to build while Live Coding is active. Compile through Live Coding instead: use the LiveCodingToolset over MCP, or ask the user to press Ctrl+Alt+F11 in the editor.
- Live Coding is unreliable for reflection changes: new or changed `UCLASS`/`USTRUCT`/`UENUM`/`UPROPERTY`/`UFUNCTION`, new classes, constructor defaults. For those, ask the user to save and close the editor, then do a full build.
- After adding or removing source files, refresh the Visual Studio solution (builds pick up new files without this):
  ```powershell
  & "E:\Epic Games\UE_5.8\Engine\Build\BatchFiles\Build.bat" -projectfiles -project="F:\Working_Games\JamGame\GameJam2026\GameJam2026.uproject" -game -rocket -progress
  ```
- Editor log (compile errors, Live Coding results, MCP server messages): the newest `Saved/Logs/GameJam2026*.log`. A restart that overlaps the old editor process writes `GameJam2026_2.log`.

## Working in the editor (Unreal MCP)
- `.mcp.json` registers the `unreal-mcp` server at `http://127.0.0.1:8000/mcp`. It is served by Epic's experimental plugins, enabled in `GameJam2026.uproject` for the Editor target only: `ModelContextProtocol` (the server), `AllToolsets` (the tools), `LiveCodingToolset` (C++ compiles).
- The server exists only while the editor is open and the server is started (Editor Preferences > Model Context Protocol > Auto Start Server, or the console command `ModelContextProtocol.StartServer`). If its tools are unavailable, ask the user to open the editor or start the server. Never guess at what an asset contains.
- Blueprints, levels, materials and input assets are binary. Inspect and change them only through the MCP tools, never by editing files on disk.
- Tool search is on: start with `list_toolsets`, then `describe_toolset`, then run tools through `call_tool`.
- MCP changes live editor state and can run Python inside the editor. Before large changes, check that the user has saved and committed. Save the assets you change.
- All Toolsets turns on the Game Features plugin in the editor only. The editor-only `GameFeatureData` entry in `Config/DefaultGame.ini` silences its startup error. Don't replace it with the editor's "Add entry to PrimaryAssetTypesToScan?" fix: that version isn't editor-only, so packaged Development builds would hit an engine `ensure`.

## Project map
- Levels: `/Game/LVL_Main` is the warehouse level and the startup map. `/Game/LVL_Soundstage` is the haunted movie-set soundstage, with the `CLUE_` actors. `Config/DefaultEngine.ini` sets the startup map and the default game mode, `GM_Main`.
- `Content/GM_Main` (GameMode) spawns `Content/FirstPerson/Blueprints/BP_FirstPersonCharacter` with `BP_FirstPersonPlayerController`. `Content/GS_Main` is the GameState.
- Input: Enhanced Input under `Content/Input`, with mapping contexts `IMC_Default` and `IMC_MouseLook`. The actions are in `Content/Input/Actions`: `IA_Move`, `IA_Look`, `IA_MouseLook`, `IA_Jump`, `IA_Interact`, `IA_Flashlight`.
- Interaction: interface `Content/Interfaces/BPI_Interactables`, base actor `Content/BluePrints/Interactables/_BP_Interactables_Master` (for example `BP_Diary`), trace channel `Interactables` (`ECC_GameTraceChannel1`) and collision profile `Interactables`.
- UI: `Content/UI/WBP_Player`. Project-made props live in `Content/Props`. Fab packs live in `Content/Fab` and at the `Content` root.
- Name new assets with the existing prefixes (`BP_`, `BPI_`, `PC_`, `GM_`, `GS_`, `IA_`, `IMC_`, `MAT_`, `LVL_`). C++ classes use the usual Unreal prefixes (`A`, `U`, `F`, `E`, `I`).
